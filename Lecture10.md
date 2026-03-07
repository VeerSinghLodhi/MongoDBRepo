# MongoDB Aggregation Pipeline Notes
## `$graphLookup` and `$unionWith`

---

## `$graphLookup`

Performs a **recursive search** on a collection, traversing graph-like relationships (e.g., hierarchies, org charts).

### Syntax

```js
db.collection.aggregate([
  {
    $graphLookup: {
      from: "<collection>",         // Collection to search
      startWith: "<expression>",    // Starting value for recursion
      connectFromField: "<field>",  // Field to follow from current doc
      connectToField: "<field>",    // Field to match in target collection
      as: "<outputField>",          // Name of the result array
      maxDepth: <number>,           // (Optional) Limit recursion depth
      depthField: "<fieldName>",    // (Optional) Adds depth level to each result
      restrictSearchWithMatch: {}   // (Optional) Filter docs during traversal
    }
  }
])
```

---

### Dataset Used

```js
db.EmployeeData.insertMany([
  { employeeId: 1, name: "CEO",        managerId: null },
  { employeeId: 2, name: "Manager 1",  managerId: 1 },
  { employeeId: 3, name: "Manager 2",  managerId: 1 },
  { employeeId: 4, name: "Employee A", managerId: 2 },
  { employeeId: 5, name: "Employee B", managerId: 2 },
  { employeeId: 6, name: "Employee C", managerId: 3 },
  { employeeId: 7, name: "Employee D", managerId: 4 },
  { employeeId: 8, name: "Employee E", managerId: 5 }
])
```

**Hierarchy:**
```
CEO (1)
├── Manager 1 (2)
│   ├── Employee A (4)
│   │   └── Employee D (7)
│   └── Employee B (5)
│       └── Employee E (8)
└── Manager 2 (3)
    └── Employee C (6)
```

---

### Examples

#### Basic — Full Chain (no depth limit)

```js
db.EmployeeData.aggregate({
  $graphLookup: {
    from: "EmployeeData",
    startWith: "$managerId",
    connectFromField: "managerId",
    connectToField: "employeeId",
    as: "employeeRecords"
  }
})
```

- Each document gets an `employeeRecords` array containing **all ancestors** up the chain.
- CEO gets an empty array (no manager).
- Employee D (depth 3) gets: Employee A → Manager 1 → CEO.

---

#### `maxDepth` — Limit Traversal Levels

```js
$graphLookup: {
  ...,
  maxDepth: 0   // Only immediate parent
}
```

| `maxDepth` | Meaning |
|---|---|
| `0` | Only direct parent (1 hop) |
| `1` | Parent + grandparent (2 hops) |
| `2` | 3 levels up |
| _(omitted)_ | Full chain, no limit |

> ⚠️ **Common mistake:** Missing a comma before `maxDepth` causes a `SyntaxError`.
>
> ```js
> // ❌ Wrong
> as: "employeeRecords"
> maxDepth: 1
>
> // ✅ Correct
> as: "employeeRecords",
> maxDepth: 1
> ```

---

#### `depthField` — Track Depth of Each Result

```js
$graphLookup: {
  ...,
  depthField: "level"   // Can be any custom name
}
```

Adds a numeric field (e.g., `level`) to each doc inside the result array:
- `0` → direct parent
- `1` → grandparent
- `2` → great-grandparent

**Example output for Employee D:**
```js
employeeRecords: [
  { name: "Employee A", level: 0 },
  { name: "Manager 1",  level: 1 },
  { name: "CEO",        level: 2 }
]
```

> The `depthField` name is **customizable** — `"level"`, `"veer"`, or any string works.

---

#### `restrictSearchWithMatch` — Filter During Traversal

```js
$graphLookup: {
  ...,
  restrictSearchWithMatch: { name: "CEO" }
}
```

- Only includes documents matching the filter **in the result array**.
- Traversal also **stops** when a non-matching document is encountered (it won't jump past it).
- Example: With `{ name: "CEO" }`, Employee A's `employeeRecords` is empty — Manager 1 is visited but filtered out, so the chain breaks before reaching CEO.

---

## `$unionWith`

Combines documents from **two collections** into a single result set — similar to SQL's `UNION ALL`.

### Syntax

```js
db.collection.aggregate([
  {
    $unionWith: {
      coll: "<otherCollection>",   // Collection to union with
      pipeline: [ ... ]            // (Optional) Pipeline to apply to that collection
    }
  }
])
```

> **Note:** `$unionWith` does **NOT** deduplicate by default — use `$group` or `$setUnion` if needed.

---

### Dataset Used

```js
db.current_users.insertMany([
  { name: "Amit",   city: "Sagar"  },
  { name: "Krishna",city: "Bhopal" },
  { name: "Amit",   city: "Sagar"  },  // duplicate
  { name: "Ayush",  city: "Sagar"  }
])

db.archived_users.insertMany([
  { name: "Rajendra", city: "Dahom"  },
  { name: "Veer",     city: "Mumbai" }
])
```

---

### Examples

#### Basic — Combine All Docs

```js
db.current_users.aggregate([
  {
    $unionWith: {
      coll: "archived_users"
    }
  }
])
```

Returns all documents from `current_users` first, then all from `archived_users`. Duplicates are **not removed**.

---

#### With `pipeline` — Filter the Second Collection

```js
db.current_users.aggregate([
  {
    $unionWith: {
      coll: "archived_users",
      pipeline: [
        { $match: { city: "Sagar" } }
      ]
    }
  }
])
```

- The `pipeline` applies **only to `archived_users`** (the second collection).
- `current_users` documents are returned as-is (all of them).
- Only `archived_users` docs where `city: "Sagar"` are included.

To filter the **base** collection too, add a `$match` stage before `$unionWith`:

```js
db.current_users.aggregate([
  { $match: { city: "Sagar" } },
  {
    $unionWith: {
      coll: "archived_users",
      pipeline: [
        { $match: { city: "Sagar" } }
      ]
    }
  }
])
```

---

### Common Syntax Error

```js
// ❌ Wrong — missing "pipeline" key name
$unionWith: {
  coll: "archived_users",
  :[
    { $match: { city: "Sagar" } }
  ]
}
// SyntaxError: Unexpected token

// ✅ Correct
$unionWith: {
  coll: "archived_users",
  pipeline: [
    { $match: { city: "Sagar" } }
  ]
}
```

---

## Quick Reference

| Feature | `$graphLookup` | `$unionWith` |
|---|---|---|
| Purpose | Recursive graph traversal | Combine two collections |
| SQL Equivalent | Recursive CTE | `UNION ALL` |
| Deduplication | N/A | Not automatic |
| Key options | `maxDepth`, `depthField`, `restrictSearchWithMatch` | `pipeline` |
| Direction | Follows field references upward/downward | Appends second collection |
