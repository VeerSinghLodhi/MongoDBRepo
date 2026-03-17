# MongoDB Schema Validation — Notes & Lessons Learned

This document covers MongoDB collection validation using `$jsonSchema`, common mistakes encountered, and how `validationLevel` and `validationAction` work.

---

## Table of Contents

1. [What is Schema Validation?](#what-is-schema-validation)
2. [Correct Syntax — `$jsonSchema`](#correct-syntax--jsonschema)
3. [Common Mistakes & Errors](#common-mistakes--errors)
4. [Validation Level: strict vs moderate](#validation-level-strict-vs-moderate)
5. [Validation Action: error vs warn](#validation-action-error-vs-warn)
6. [Modifying Validation on Existing Collections](#modifying-validation-on-existing-collections)
7. [Full Working Examples](#full-working-examples)
8. [Quick Reference Cheatsheet](#quick-reference-cheatsheet)

---

## What is Schema Validation?

MongoDB allows you to enforce rules on documents in a collection using **schema validation**. When a document is inserted or updated, MongoDB checks it against the defined rules. If it doesn't match, the operation either fails or logs a warning (depending on settings).

---

## Correct Syntax — `$jsonSchema`

The validator must be wrapped inside `$jsonSchema` (not `bsonSchema`, not `$bsonSchema`, not `bsonType` at the top level).

```js
db.createCollection('CollectionName', {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ['name', 'salary'],
      properties: {
        name: { bsonType: 'string' },
        salary: {
          bsonType: 'int',
          minimum: 10000
        }
      }
    }
  }
});
```

> **Key rule:** All field rules must be inside `properties: { ... }` — not alongside it at the same level.

---

## Common Mistakes & Errors

### ❌ Mistake 1: `salary` outside `properties`

```js
// WRONG — salary is outside properties block
validator: {
  bsonType: "object",
  required: ['name', 'salary'],
  properties: {
    name: { bsonType: 'string' }
  },
  salary: {           // ← This is ignored / wrong position
    bsonType: 'int',
    minimum: 10000
  }
}
```

**What happened in the session:** Even though inserts with salary values like `15000` or `150000` failed, it was actually because the `validator` key itself was missing `$jsonSchema`, not because of the salary rule.

**Fix:** Move `salary` inside `properties`.

---

### ❌ Mistake 2: Typo in `bsonType` — `"objecct"` instead of `"object"`

```js
bsonType: "objecct"   // ← Typo — MongoDB accepted it but it didn't work correctly
```

MongoDB accepted this at collection creation time but it did not validate properly. Always double-check spelling.

---

### ❌ Mistake 3: Using `bsonSchema` or `$bsonSchema` instead of `$jsonSchema`

```js
// WRONG
validator: { bsonSchema: { ... } }         // → Collection created but validator ignored
validator: { $bsonSchema: { ... } }        // → MongoServerError: unknown top level operator
validator: { '$bsonSchema': { ... } }      // → Same error
```

**Fix:** Use `$jsonSchema` — this is the correct and only supported key.

---

### ❌ Mistake 4: Missing comma between `validator` and `validationLevel`

```js
// WRONG — SyntaxError
db.runCommand({
  collMod: 'Emps',
  validator: { ... }    // ← Missing comma here!
  validationLevel: 'moderate'
})
```

**Fix:** Add a comma after the closing `}` of `validator`.

---

### ❌ Mistake 5: Trying to recreate a collection without dropping it first

```js
// If Persons already exists, this will fail:
db.createCollection('Persons', { ... })
// MongoServerError[NamespaceExists]: ...

// Fix: Drop first, then recreate
db.Persons.drop();
db.createCollection('Persons', { ... });
```

---

### ❌ Mistake 6: `getCollectionInfos` requires an object, not a string

```js
db.getCollectionInfos('Persons')    // ❌ Error — expects object
db.getCollectionInfos({name: 'Persons'})  // ✅ Correct
```

---

## Validation Level: strict vs moderate

`validationLevel` controls **which documents** are checked against the schema.

| Level | New Inserts | Existing Documents (on update) |
|---|---|---|
| `strict` (default) | ✅ Validated | ✅ Validated |
| `moderate` | ✅ Validated | ⚠️ Only validated if they **already passed** validation |

### strict (default)

Every insert AND every update is checked against the schema, no exceptions.

```js
db.runCommand({
  collMod: 'Emps',
  validator: { $jsonSchema: { ... } },
  validationLevel: 'strict'
});
```

Use this when: You want no documents — old or new — to violate the schema.

---

### moderate

New inserts are always validated. But when **updating** a document, MongoDB only enforces the schema if that document **already matched** the validator. Documents that were inserted before the validator was added (and don't match it) can still be updated without hitting a validation error.

```js
db.runCommand({
  collMod: 'Emps',
  validator: { $jsonSchema: { ... } },
  validationLevel: 'moderate'
});
```

**Real example from the session:**

```js
// A document with salary: 5000 was inserted BEFORE the validator was added
// After adding the validator with validationLevel: 'moderate'...

db.Emps.updateOne({name: "Veer"}, {$set: {name: "Krishna"}})
// ✅ Succeeds — document pre-dates the validator, so it is exempt from validation

db.Emps.updateOne({name: "Veer"}, {$set: {name: "Krishna", salary: 2000}})
// This also succeeds silently (matched 0 because name was already updated)
```

Use this when: You have legacy data that doesn't conform to the new schema and you don't want to break existing update operations.

---

## Validation Action: error vs warn

`validationAction` controls **what happens** when a document fails validation.

| Action | Effect |
|---|---|
| `error` (default) | Insert/update is **rejected**. An error is returned to the client. |
| `warn` | Insert/update **succeeds**, but a warning message is written to the MongoDB log. |

### error (default)

```js
db.createCollection('Emps', {
  validator: { $jsonSchema: { ... } },
  validationAction: 'error'
});

db.Emps.insertOne({name: 'Veer', salary: 0});
// MongoServerError: Document failed validation
```

### warn

```js
db.createCollection('validationAction', {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ['name', 'salary'],
      properties: {
        name: { bsonType: 'string' },
        salary: { bsonType: 'int', minimum: 10000 }
      }
    }
  },
  validationAction: 'warn'
});

db.validationAction.insertOne({name: 'veer', salary: 0});
// ✅ Insert SUCCEEDS
// ⚠️  Warning written to MongoDB log — check with:
db.adminCommand({getLog: 'global'});
```

Use `warn` when: You are migrating to a new schema and want to monitor violations without blocking operations.

---

## Modifying Validation on Existing Collections

Use `db.runCommand` with `collMod` to add or update a validator on an existing collection:

```js
db.runCommand({
  collMod: 'Emps',
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ['name', 'salary'],
      properties: {
        name: { bsonType: 'string' },
        salary: {
          bsonType: 'int',
          minimum: 10000
        }
      }
    }
  },
  validationLevel: 'moderate',   // optional
  validationAction: 'error'      // optional
});
```

> **Note:** `collMod` does **not** retroactively reject existing documents. It only affects future inserts and updates.

---

## Full Working Examples

### Example 1 — Emp Collection (from session)

```js
db.createCollection('Emp', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['name', 'address', 'dept', 'salary', 'email'],
      properties: {
        name: { bsonType: 'string', description: 'Name is required' },
        address: { bsonType: 'string', description: 'Address is required' },
        dept: {
          enum: ['IT', 'Support'],
          description: 'Department must be IT or Support'
        },
        salary: {
          bsonType: 'int',
          minimum: 10000,
          maximum: 50000,
          description: 'Salary must be between 10000 and 50000'
        },
        email: {
          bsonType: 'string',
          pattern: '@',
          description: 'Email must contain @'
        }
      }
    }
  }
});
```

### Example 2 — Warn + Moderate together

```js
db.createCollection('SoftValidation', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['name', 'salary'],
      properties: {
        name: { bsonType: 'string' },
        salary: { bsonType: 'int', minimum: 10000 }
      }
    }
  },
  validationLevel: 'moderate',
  validationAction: 'warn'
});
```

---

## Quick Reference Cheatsheet

```
✅ Correct keyword:        $jsonSchema
❌ Wrong keywords:         bsonSchema, $bsonSchema, $jsonType

✅ Field rules go inside:  properties: { fieldName: { ... } }
❌ NOT alongside:          properties: { ... }, fieldName: { ... }

validationLevel:
  strict   → validates ALL inserts + updates (default)
  moderate → validates all inserts; updates only checked if doc already valid

validationAction:
  error    → rejects the operation on failure (default)
  warn     → allows the operation but logs a warning

To update validator on existing collection → use db.runCommand({ collMod: ... })
To view validator on a collection         → db.getCollectionInfos({name: 'collName'})
To view warnings in log                   → db.adminCommand({getLog: 'global'})
```
