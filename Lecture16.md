# MongoDB Indexes — Notes

## What is an Index?

An index in MongoDB is a special data structure that speeds up queries by allowing the database to look up documents efficiently rather than doing a full collection scan. Without indexes, MongoDB reads every document in a collection to find matches.

---

## Types of Indexes Covered

### 1. Default `_id` Index
Every collection automatically has a unique index on the `_id` field. You cannot drop it.

---

### 2. Single Field Index
An index on one field.

```js
db.collection.createIndex({ fieldName: 1 })   // 1 = ascending
db.collection.createIndex({ fieldName: -1 })  // -1 = descending
```

**Example:**
```js
db.Example3.createIndex({ name: -1 })
// Returns: name_-1
```

> The index name is auto-generated as `fieldName_direction` (e.g., `name_-1`).

---

### 3. Unique Index
Prevents duplicate values in a field across documents.

```js
db.collection.createIndex({ email: 1 }, { unique: true })
```

**Key behaviors:**
- Inserting a duplicate value throws: `E11000 duplicate key error`
- MongoDB is **case-sensitive** — `veerlodhi@gmail.com` and `veerLodhi@gmail.com` are treated as different values
- A unique index **cannot be created** on a field that already has duplicate values in existing documents
- A unique index **cannot be created** on a field that is missing (`null`) in multiple documents — `null` counts as a value and would be a duplicate

**Example of the null problem:**
```js
// Example1 has documents without a 'phone' field
db.Example1.createIndex({ phone: 1 }, { unique: true })
// ERROR: E11000 duplicate key error — { phone: null }
// Because multiple documents have phone = null (missing = null)
```

---

### 4. Compound Index
An index on **multiple fields together**. Uniqueness is enforced on the *combination* of fields.

```js
db.collection.createIndex({ name: 1, email: 1 }, { unique: true })
```

**Behavior:**
- `{ name: "Krishna", email: "krishna@gmail.com" }` can only exist once
- `{ name: "Krishna Kurmi", email: "krishna@gmail.com" }` is allowed — different `name`
- `{ name: "Krishna", email: "other@gmail.com" }` is allowed — different `email`

---

### 5. Partial Index
An index that only covers documents matching a filter condition. Useful for indexing a **subset** of a collection (e.g., only adults).

```js
db.collection.createIndex(
  { age: 1 },
  { partialFilterExpression: { age: { $gte: 18 } } }
)
```

**Key points:**
- The filter condition must be a proper query expression (e.g., `{ age: { $gte: 18 } }`)
- Writing just `{ $gte: 18 }` without a field name throws: `unknown top level operator: $gte`
- Documents **not** matching the filter are excluded from the index entirely
- Queries that don't include the filter condition won't use the partial index

**Example:**
```js
// Index only covers documents where age >= 18
db.ExamplePartialIn.createIndex(
  { age: 1 },
  { partialFilterExpression: { age: { $gte: 18 } } }
)

// This query WILL use the index
db.ExamplePartialIn.find({ age: { $gte: 18 } })

// abc (age: 15) is NOT indexed
```

---

## Managing Indexes

### View All Indexes on a Collection
```js
db.collection.getIndexes()
```

**Example output:**
```js
[
  { v: 2, key: { _id: 1 }, name: '_id_' },
  { v: 2, key: { email: 1 }, name: 'email_1', unique: true }
]
```

---

### Drop an Index

You can drop by **index name** (string) or by **key pattern** (object).

```js
// By key pattern (recommended)
db.collection.dropIndex({ fieldName: 1 })
db.collection.dropIndex({ fieldName: -1 })

// By index name
db.collection.dropIndex("fieldName_1")
```

> ⚠️ The direction matters — `{ name: 1 }` and `{ name: -1 }` are different indexes.
> Trying to drop `{ name: 1 }` when only `{ name: -1 }` exists throws `IndexNotFound`.

---

### Drop a Collection (removes all indexes too)
```js
db.collection.drop()
```

> Note: `db.Example2.drop("email_1")` — the argument is **ignored**; `drop()` always drops the entire collection regardless of what you pass in.

---

## Common Errors & Fixes

| Error | Cause | Fix |
|---|---|---|
| `E11000 duplicate key error` | Tried to insert a duplicate value into a unique-indexed field | Remove the duplicate or use a non-unique index |
| `Index build failed: dup key: { field: null }` | Multiple documents are missing the field (null collision) | Add the field to all docs first, or use a partial index |
| `index not found with name [email]` | Wrong index name passed to `dropIndex` | Use `getIndexes()` to find exact name, or drop by key object |
| `Missing required argument` | Called `deleteOne()` with no filter | Always provide a filter: `deleteOne({ field: value })` |
| `unknown top level operator: $gte` | Missing field name in `partialFilterExpression` | Wrap in field: `{ age: { $gte: 18 } }` |
| `SyntaxError: Missing semicolon` | Used `:` instead of `;` at end of command | Replace `:` with `;` |

---

## Quick Reference

```js
// Create indexes
db.col.createIndex({ field: 1 })                                      // single
db.col.createIndex({ field: 1 }, { unique: true })                    // unique
db.col.createIndex({ f1: 1, f2: 1 }, { unique: true })               // compound unique
db.col.createIndex({ field: 1 }, { partialFilterExpression: { field: { $gte: 18 } } }) // partial

// View indexes
db.col.getIndexes()

// Drop index
db.col.dropIndex({ field: 1 })      // by key
db.col.dropIndex("field_1")         // by name

// Drop collection (removes all indexes)
db.col.drop()
```
