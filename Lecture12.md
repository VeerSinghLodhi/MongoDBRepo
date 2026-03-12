# MongoDB Aggregation Pipeline — Notes

## Collections Used

### `Products`
| pid | name | price |
|-----|-----------|-------|
| 1 | Pen | 20 |
| 2 | NoteBook | 80 |
| 3 | Keyboard | 450 |
| 4 | Headphones | 700 |
| 5 | Monitor | 950 |
| 6 | Laptop | 50000 |

### `Sales`
| item | price |
|------|-------|
| A | 30 |
| B | 80 |
| B | 150 |
| C | 450 |
| D | 700 |
| E | 1500 |

---

## `$bucketAuto` — Auto Bucketing

Groups documents into a specified number of buckets automatically based on a field's value distribution.

### Syntax
```js
db.collection.aggregate([
  {
    $bucketAuto: {
      groupBy: "$fieldName",
      buckets: <number>,         // required
      granularity: "<series>"    // optional
    }
  }
])
```

### Basic Example — 4 Buckets
```js
db.Sales.aggregate({
  $bucketAuto: {
    groupBy: "$price",
    buckets: 4
  }
})
```
> **Note:** `aggregate()` accepts a single stage object directly (without array `[]`) only when there's one stage.

**Result:**
```
{ _id: { min: 30,  max: 150  }, count: 2 }
{ _id: { min: 150, max: 700  }, count: 2 }
{ _id: { min: 700, max: 1500 }, count: 2 }
```
> Only 3 buckets returned instead of 4 because MongoDB tries to distribute documents evenly — with 6 docs, 3 buckets of 2 is optimal.

---

### With `granularity` — Preferred Number Series

The `granularity` option snaps bucket boundaries to a standard number series (e.g., R5, R10, E6, POWERSOF2, etc.).

```js
db.Sales.aggregate({
  $bucketAuto: {
    groupBy: "$price",
    buckets: 3,
    granularity: "R5"   // Renard R5 series
  }
})
```
**Result:**
```
{ _id: { min: 25,  max: 100  }, count: 2 }
{ _id: { min: 100, max: 630  }, count: 2 }
{ _id: { min: 630, max: 1600 }, count: 2 }
```

```js
db.Sales.aggregate({
  $bucketAuto: {
    groupBy: "$price",
    buckets: 3,
    granularity: "R10"  // Renard R10 series (finer granularity)
  }
})
```
**Result:**
```
{ _id: { min: 25,  max: 100  }, count: 2 }
{ _id: { min: 100, max: 500  }, count: 2 }
{ _id: { min: 500, max: 1600 }, count: 2 }
```

---

### ❌ Common Mistakes with `$bucketAuto`

| Mistake | Error |
|--------|-------|
| Typo: `backets` instead of `buckets` | `Unrecognized option to $bucketAuto: backets` |
| Missing `buckets` field (only `groupBy` provided) | `$bucketAuto requires 'groupBy' and 'buckets' to be specified` |

---

## `$group` — Grouping Documents

Groups documents by a field and computes aggregated values.

### Syntax
```js
db.collection.aggregate([
  {
    $group: {
      _id: "$fieldName",
      aliasName: { $accumulator: "$field" }
    }
  }
])
```

### Example
```js
db.Orders.aggregate([{
  $group: {
    _id: "$product",
    totalSalary: { $sum: "$amount" }
  }
}])
```
**Sample Result:**
```
{ _id: 'Laptop', totalSalary: 52000 }
{ _id: 'xyz',    totalSalary: 2500  }
{ _id: 'abc',    totalSalary: 2000  }
...
```

---

## `$out` — Writing Results to a Collection

Writes the output of an aggregation pipeline to a collection. Replaces the collection if it already exists.

### Syntax — Same Database
```js
db.collection.aggregate([
  { /* pipeline stage */ },
  { $out: "targetCollectionName" }
])
```

### Syntax — Different Database
```js
db.collection.aggregate([
  { /* pipeline stage */ },
  {
    $out: {
      db: "targetDB",
      coll: "targetCollectionName"
    }
  }
])
```

### Example — Save `$group` result to `sales_summary`
```js
db.Orders.aggregate([
  {
    $group: {
      _id: "$product",
      totalSalary: { $sum: "$amount" }
    }
  },
  {
    $out: "sales_summary"
  }
])

// Verify
db.sales_summary.find();
```

### Example — Save to a Different Database (`reportsDB`)
```js
db.Orders.aggregate([
  {
    $group: {
      _id: "$product",
      totalSalary: { $sum: "$amount" }
    }
  },
  {
    $out: {
      db: "reportsDB",
      coll: "sales_summary"
    }
  }
])

// Switch and verify
use reportsDB
db.sales_summary.find();
```

---

### ❌ Common Mistakes with `$out`

| Mistake | Error |
|--------|-------|
| Typo: `colling` instead of `coll` | `BSON field '$out.colling' is an unknown field` |
| Passing multiple stages as a plain object `{}` instead of array `[]` | `SyntaxError: Unexpected token` |
| Missing closing brace `}` before next stage in array | `SyntaxError: Unexpected token` |

---

## ✅ Correct Multi-Stage Pipeline Syntax

Always wrap multiple pipeline stages in an **array `[]`**, and each stage in its own **object `{}`**.

```js
// ✅ CORRECT
db.Collection.aggregate([
  {
    $stageName: { /* options */ }
  },
  {
    $out: "resultCollection"
  }
])

// ❌ WRONG — stages not wrapped in array
db.Collection.aggregate(
  { $stageName: { ... } },
  { $out: "..." }
)

// ❌ WRONG — stages inside one object instead of separate objects
db.Collection.aggregate([{
  $stageName: { ... },
  { $out: "..." }     // syntax error here
}])
```

---

## Quick Reference

| Stage | Purpose |
|-------|---------|
| `$bucketAuto` | Auto-divide documents into N buckets by a field |
| `$group` | Group docs and compute aggregates (`$sum`, `$avg`, etc.) |
| `$out` | Write pipeline result to a collection (replaces existing) |

| Option | Used With | Notes |
|--------|-----------|-------|
| `groupBy` | `$bucketAuto` | Field to bucket by |
| `buckets` | `$bucketAuto` | Number of buckets — **required** |
| `granularity` | `$bucketAuto` | Snap boundaries to a number series (R5, R10, etc.) |
| `_id` | `$group` | Field to group by |
| `db` / `coll` | `$out` | Target database and collection name |
