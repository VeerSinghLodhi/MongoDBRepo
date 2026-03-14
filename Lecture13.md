# MongoDB Schema Validation Notes

## Table of Contents
- [What is Schema Validation?](#what-is-schema-validation)
- [Creating a Collection with Validator](#creating-a-collection-with-validator)
- [JSON Schema Keywords](#json-schema-keywords)
- [Practical Examples](#practical-examples)
- [Key Observations & Gotchas](#key-observations--gotchas)
- [Inspecting Collection Validators](#inspecting-collection-validators)
- [Common Errors](#common-errors)

---

## What is Schema Validation?

MongoDB allows you to enforce rules on documents inserted into a collection using **Schema Validation** via `$jsonSchema`. This ensures data consistency without a rigid relational schema.

---

## Creating a Collection with Validator

```js
db.createCollection("CollectionName", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ['field1', 'field2'],
      properties: {
        field1: { bsonType: 'string' },
        field2: { bsonType: 'int' }
      }
    }
  }
})
```

> ⚠️ **Typo Alert:** The key is `validator` (not `validaor`).  
> Wrong spelling causes:  
> `MongoServerError[IDLUnknownField]: BSON field 'create.validaor' is an unknown field.`

---

## JSON Schema Keywords

| Keyword     | Purpose                                      | Example Value              |
|-------------|----------------------------------------------|----------------------------|
| `bsonType`  | Specifies the BSON data type                 | `'string'`, `'int'`        |
| `required`  | Array of fields that must be present         | `['name', 'age']`          |
| `enum`      | Field value must be one of the listed values | `['IT', 'Support']`        |
| `minimum`   | Minimum numeric value (inclusive)            | `10000`                    |
| `maximum`   | Maximum numeric value (inclusive)            | `50000`                    |
| `pattern`   | Regex pattern the string must match          | `'@'`                      |
| `description` | Human-readable description (not enforced)  | `"Name is required"`       |

---

## Practical Examples

### Example 1: Basic Validator (Person)

```js
db.createCollection("Person", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ['name', 'age'],
      properties: {
        name: { bsonType: 'string' },
        age:  { bsonType: 'int' }
      }
    }
  }
})
```

**Behavior:**

| Insert                          | Result  | Reason                        |
|---------------------------------|---------|-------------------------------|
| `{ name: "veer", age: 20 }`    | ✅ Pass  | Valid types, required fields present |
| `{ name: "veer", age: '20' }`  | ❌ Fail  | `age` must be `int`, not `string` |
| `{ name: 10, age: 20 }`        | ❌ Fail  | `name` must be a `string` |
| `{ name: 'veer' }`             | ❌ Fail  | `age` is required but missing |
| `{ name: 'veer', age: 20, address: 'sagar' }` | ✅ Pass | Extra fields are **allowed** |
| `{ name: null, age: 20 }`      | ❌ Fail  | `null` fails `bsonType: 'string'` |

---

### Example 2: Advanced Validator (Emp)

```js
db.createCollection("Emp", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ['name', 'address', 'dept', 'salary', 'email'],
      properties: {
        name: {
          bsonType: 'string',
          description: "Name is required"
        },
        address: {
          bsonType: 'string',
          description: "Address is required"
        },
        dept: {
          enum: ['IT', 'Support'],
          description: "Department must be IT or Support"
        },
        salary: {
          bsonType: 'int',
          minimum: 10000,
          maximum: 50000,
          description: 'Salary must be between 10000 and 50000'
        },
        email: {
          bsonType: "string",
          pattern: '@',
          description: "Email must be a valid string"
        }
      }
    }
  }
})
```

**Behavior:**

| Insert Attempt                                 | Result  | Reason                              |
|------------------------------------------------|---------|-------------------------------------|
| Valid doc with all correct fields              | ✅ Pass  | All rules satisfied                 |
| `email: 'veerlodhi54gmail.com'` (no `@`)       | ❌ Fail  | Pattern `'@'` not found in string   |
| `dept: 'I'`                                    | ❌ Fail  | Not in `enum ['IT', 'Support']`     |
| `dept: 'It'` (wrong case)                      | ❌ Fail  | Enum is case-sensitive              |
| `salary: 1000` (below minimum)                 | ❌ Fail  | Below `minimum: 10000`              |
| `salary: 50001` (above maximum)                | ❌ Fail  | Above `maximum: 50000`              |
| `salary: 10000` or `salary: 50000`             | ✅ Pass  | Boundary values are **inclusive**   |
| `name: 1` (integer instead of string)          | ❌ Fail  | `bsonType: 'string'` required       |
| `email: '@'` (just the `@` symbol)             | ✅ Pass  | Pattern only checks for presence of `@` |
| `name: ''` (empty string)                      | ✅ Pass  | `bsonType` doesn't enforce non-empty |

---

## Key Observations & Gotchas

### 1. Extra Fields Are Allowed
MongoDB schema validation does **not** block extra/unknown fields by default.

```js
// This PASSES even though 'address', 'course', 'marks' are not in the schema
db.Person.insertOne({ name: 'veer', age: 20, address: 'sagar', course: 'Java' })
```

To restrict extra fields, use `additionalProperties: false` in the schema.

---

### 2. `null` Fails `bsonType` Checks
If a field is in the `required` array and has `bsonType: 'string'`, passing `null` will fail.

```js
db.Person.insertOne({ name: null, age: 20 })  // ❌ MongoServerError: Document failed validation
```

---

### 3. `enum` Is Case-Sensitive
```js
dept: 'IT'   // ✅ Valid
dept: 'It'   // ❌ Invalid — enum values must match exactly
dept: 'it'   // ❌ Invalid
```

---

### 4. `pattern` Only Checks for Substring Match
The `pattern` field uses regex. Using just `'@'` only verifies that `@` appears **somewhere** in the string.

```js
email: '@'                    // ✅ Passes (technically valid by the rule)
email: 'veerlodhi54gmail.com' // ❌ Fails (no @ present)
```

Use a stricter pattern like `'^[^@]+@[^@]+\\.[^@]+$'` for real email validation.

---

### 5. `bsonType: 'string'` Does NOT Enforce Non-Empty
An empty string `''` is still a valid string.

```js
db.Emp.insertOne({ name: '', address: '', dept: 'Support', salary: 50000, email: '@' })
// ✅ This PASSES
```

Use `minLength: 1` to enforce non-empty strings.

---

### 6. `validationAction` (Not `validationActive`)
The correct option key is `validationAction`, not `validationActive`.

```js
// ❌ Wrong — causes error
validationActive: 'error'

// ✅ Correct
validationAction: 'error'  // or 'warn'
```

---

### 7. `salary: int` vs JavaScript Numbers
MongoDB's `bsonType: 'int'` expects a 32-bit integer. In the MongoDB shell, `NumberInt()` can be used explicitly. Plain JS numbers like `25000` are typically inferred as `int` in mongosh.

---

## Inspecting Collection Validators

### Method 1: `getCollectionInfos`
```js
db.getCollectionInfos({ name: 'Emp' })
```

### Method 2: `runCommand` with `listCollections`
```js
db.runCommand({
  listCollections: 1,
  filter: { name: 'Emp' }
})
```

> Note: The value passed to `listCollections` (1, 0, 10, etc.) does **not** affect output. It acts as a truthy flag — any value lists collections.

Both methods return the full validator schema, options, UUID, and index info for the collection.

---

## Common Errors

| Error | Cause |
|-------|-------|
| `MongoServerError: Document failed validation` | Inserted document violates the `$jsonSchema` rules |
| `IDLUnknownField: BSON field 'create.validaor'` | Typo in `validator` key |
| `IDLUnknownField: BSON field 'create.validationActive'` | Wrong key — should be `validationAction` |

---

## Quick Reference

```
required   → field must be present (null still fails bsonType check)
bsonType   → enforces data type ('string', 'int', 'double', 'bool', 'array', 'object', etc.)
enum       → value must be one of the listed options (case-sensitive)
minimum    → inclusive lower bound for numbers
maximum    → inclusive upper bound for numbers
pattern    → regex pattern the string must contain/match
```
