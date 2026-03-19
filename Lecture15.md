# MongoDB Indexes — Notes

## What is an Index?

An index in MongoDB is a data structure that improves the speed of read operations (queries) on a collection. Without an index, MongoDB performs a **collection scan** — reading every document. With an index, it can jump directly to the relevant documents.

---

## Default Index

Every MongoDB collection automatically gets a default index on `_id`:

```js
{ v: 2, key: { _id: 1 }, name: '_id_' }
```

- This cannot be dropped.
- `1` means **ascending** order.

---

## Inserting Documents

```js
db.IndexEx.insertOne({ name: 'A' })
db.IndexEx.insertOne({ name: 'B' })
db.IndexEx.insertOne({ name: 'C' })
```

Documents can be inserted **before or after** creating indexes. Indexes apply retroactively to existing documents too.

---

## Viewing Indexes

```js
db.IndexEx.getIndexes()
```

Returns an array of all indexes on the collection.

---

## Creating a Single Field Index

```js
db.IndexEx.createIndex({ name: 1 })
// Returns: name_1
```

- `1` = **ascending**
- `-1` = **descending**
- The default name is auto-generated: `fieldName_direction` → e.g. `name_1`

> **Note:** Running `createIndex()` on an already-existing index does **not** create a duplicate — MongoDB just returns the existing index name. Safe to run multiple times.

---

## Creating an Index with a Custom Name

```js
db.IndexEx.createIndex({ email: 1 }, { name: "email_index" })
// Returns: email_index
```

- Second argument is the **options object**.
- `name` lets you set a human-readable index name instead of the auto-generated one.
- Again, re-running this command on an existing index is idempotent — no duplicate is created.

---

## Dropping an Index

```js
db.IndexEx.dropIndex("email_index")
// Returns: { nIndexesWas: 3, ok: 1 }
```

- Pass the **index name** as a string.
- `nIndexesWas` shows how many indexes existed before the drop.
- After dropping, you can recreate it fresh.

> ⚠️ You **cannot** drop the default `_id_` index.

---

## Compound Index (Multiple Fields)

A compound index covers multiple fields in one index.

```js
// ❌ Syntax error — missing colon
db.IndexEx.createIndex({ phone: 1, age -1 })

// ✅ Correct syntax
db.IndexEx.createIndex({ phone: 1, age: -1 })
// Returns: phone_1_age_-1
```

Auto-generated name format: `field1_dir1_field2_dir2`

### Three-field Compound Index

```js
db.IndexEx.createIndex({ phone: 1, name: 1, age: -1 })
// Returns: phone_1_name_1_age_-1
```

---

## Final State of Indexes

```js
db.IndexEx.getIndexes()
```

```json
[
  { "v": 2, "key": { "_id": 1 },                       "name": "_id_"                },
  { "v": 2, "key": { "name": 1 },                      "name": "name_1"              },
  { "v": 2, "key": { "email": 1 },                     "name": "email_index"         },
  { "v": 2, "key": { "phone": 1, "age": -1 },          "name": "phone_1_age_-1"      },
  { "v": 2, "key": { "phone": 1, "name": 1, "age": -1 }, "name": "phone_1_name_1_age_-1" }
]
```

---

## Quick Reference

| Operation | Command |
|---|---|
| View all indexes | `db.collection.getIndexes()` |
| Create single field index | `db.collection.createIndex({ field: 1 })` |
| Create with custom name | `db.collection.createIndex({ field: 1 }, { name: "myName" })` |
| Create compound index | `db.collection.createIndex({ f1: 1, f2: -1 })` |
| Drop an index | `db.collection.dropIndex("indexName")` |

---

## Key Takeaways

- `1` = ascending, `-1` = descending
- `_id` index is **always present** and cannot be dropped
- `createIndex()` is **idempotent** — safe to call multiple times
- Compound indexes follow **field order** — it matters for query optimization
- Custom names make indexes easier to manage than auto-generated ones
- Indexes speed up **reads** but add overhead to **writes**
