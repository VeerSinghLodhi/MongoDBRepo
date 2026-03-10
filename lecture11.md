# MongoDB Aggregation — `$bucket` & `$bucketAuto` Notes

## 📦 Sample Collections Used

### `Employees` Collection
| empId | Name | Department | Designation | Salary | Experience | City |
|-------|------|------------|-------------|--------|------------|------|
| 101 | Veer Singh Lodhi | IT | Software Engineer | 70000 | 2 | Sagar |
| 102 | Muskan Shroti | IT | Software Engineer | 60000 | 1 | Sagar |
| 103 | Krishna Kurmi | HR | HR Executive | 50000 | 3 | Bhopal |
| 104 | Rajendra Kurmi | Finance | Accountant | 40000 | 2 | Ahmedabad |
| 105 | Akash Pateriya | Marketing | Marketing Manager | 75000 | 6 | Delhi |
| 106 | Rohit Kumar | IT | Senior Developer | 90000 | 7 | Bengaluru |

### `Products` Collection
| pid | name | price |
|-----|------|-------|
| 1 | Pen | 20 |
| 2 | NoteBook | 80 |
| 3 | Keyboard | 450 |
| 4 | Headphones | 700 |
| 5 | Monitor | 950 |
| 6 | Laptop | 50000 |

---

## 🪣 `$bucket` Stage

Groups documents into **manually defined** price/value ranges (buckets).

### Syntax
```js
db.collection.aggregate([
  {
    $bucket: {
      groupBy: "$fieldName",       // Field to group by
      boundaries: [v1, v2, v3],    // Bucket edges — must be sorted, min 2 values
      default: "label",            // Catch-all for values outside boundaries (optional)
      output: {                    // Custom output fields (optional)
        fieldName: { $accumulator }
      }
    }
  }
])
```

### How Boundaries Work
- `boundaries: [0, 100, 500, 1000]` creates buckets:
  - `[0, 100)` → `_id: 0`
  - `[100, 500)` → `_id: 100`
  - `[500, 1000)` → `_id: 500`
  - anything outside → goes to `default`
- Boundaries must be in **ascending order**
- Each bucket's `_id` is the **lower bound**

---

### Examples

#### Basic Count per Bucket
```js
db.Products.aggregate({
  $bucket: {
    groupBy: "$price",
    boundaries: [0, 100, 500, 1000],
    default: "Other",
    output: { count: { $sum: 1 } }
  }
})
```
**Output:**
```
{ _id: 0,       count: 2 }   // Pen (20), NoteBook (80)
{ _id: 100,     count: 1 }   // Keyboard (450)
{ _id: 500,     count: 2 }   // Headphones (700), Monitor (950)
{ _id: 'Other', count: 1 }   // Laptop (50000) — outside [0,1000)
```

> 💡 To avoid using `default`, extend your boundaries to cover all values:
> `boundaries: [0, 100, 500, 1000, 50001]` — Laptop goes into `_id: 1000` bucket.

---

#### Average Price per Bucket
```js
db.Products.aggregate({
  $bucket: {
    groupBy: "$price",
    boundaries: [0, 100, 500, 1000],
    default: "Other",
    output: {
      count: { $sum: 1 },
      averagePrice: { $avg: "$price" }
    }
  }
})
```
**Output:**
```
{ _id: 0,       count: 2, averagePrice: 50    }
{ _id: 100,     count: 1, averagePrice: 450   }
{ _id: 500,     count: 2, averagePrice: 825   }
{ _id: 'Other', count: 1, averagePrice: 50000 }
```

> ⚠️ Using `$avg: "$salary"` on Products collection returns `null` — Products don't have a `salary` field.

---

#### Push Product Names into an Array per Bucket
```js
db.Products.aggregate({
  $bucket: {
    groupBy: "$price",
    boundaries: [0, 40, 60, 80, 100],
    default: "Others",
    output: {
      count: { $sum: 1 },
      proarray: { $push: "$name" }
    }
  }
})
```
**Output:**
```
{ _id: 0,        count: 1, proarray: ['Pen']                                    }
{ _id: 80,       count: 1, proarray: ['NoteBook']                               }
{ _id: 'Others', count: 4, proarray: ['Keyboard','Headphones','Monitor','Laptop']}
```

> ⚠️ If a value falls outside boundaries and no `default` is set → **MongoServerError** is thrown.

---

#### Grouping by String Field (`$name`)
```js
db.Products.aggregate({
  $bucket: {
    groupBy: "$name",
    boundaries: ['A', 'M', 'T'],
    output: { count: { $sum: 1 } }
  }
})
```
**Output:**
```
{ _id: 'A', count: 3 }   // Headphones, Keyboard, Laptop (H, K, L)
{ _id: 'M', count: 3 }   // Monitor, NoteBook, Pen (M, N, P)
```

> ⚠️ When using string boundaries with `default`, the default value must be **less than the lowest boundary** OR **greater than or equal to the highest boundary**. Otherwise → `MongoServerError[Location40199]`.
>
> Example: `default: "Other"` fails because "O" falls between 'A' and 'T'. Use `default: "Z"` or omit it.

---

### `output` Field Behavior

| `output` value | Result |
|---|---|
| `{ count: { $sum: 1 } }` | Counts documents per bucket |
| `{ count: { $sum: 0 } }` | Always returns `count: 0` |
| `{ count: { $sum: 2 } }` | Returns `count: 2 * (number of docs)` |
| `{ count: { $sum: 2/2 } }` | JS evaluates `2/2=1`, same as `$sum: 1` |
| `{}` (empty object) | Only `_id` is returned, no other fields |
| *(omit output entirely)* | Defaults to `{ count: { $sum: 1 } }` |

---

## 🤖 `$bucketAuto` Stage

Groups documents into **automatically calculated** equal buckets. You specify *how many* buckets you want — MongoDB figures out the boundaries.

### Syntax
```js
db.collection.aggregate([
  {
    $bucketAuto: {
      groupBy: "$fieldName",   // Field to group by
      buckets: N,              // Number of buckets desired
      output: {                // Custom output (optional)
        fieldName: { $accumulator }
      }
    }
  }
])
```

> 💡 No need for `boundaries` or `default` — MongoDB handles everything automatically.

---

### Examples

#### Auto-split Products into 4 buckets
```js
db.Products.aggregate({
  $bucketAuto: {
    groupBy: "$price",
    buckets: 4
  }
})
```
**Output:** *(MongoDB actually created 3 buckets to balance the data)*
```
{ _id: { min: 20,  max: 450   }, count: 2 }
{ _id: { min: 450, max: 950   }, count: 2 }
{ _id: { min: 950, max: 50000 }, count: 2 }
```

> 💡 The `_id` contains `{ min, max }` instead of a single lower-bound value like `$bucket`.
> Actual number of buckets may be **less than requested** if data doesn't divide evenly.

---

#### Auto Buckets with Product Name Array
```js
db.Products.aggregate({
  $bucketAuto: {
    groupBy: "$price",
    buckets: 4,
    output: {
      proArray: { $push: "$name" }
    }
  }
})
```
**Output:**
```
{ _id: { min: 20,  max: 450   }, proArray: ['Pen', 'NoteBook']         }
{ _id: { min: 450, max: 950   }, proArray: ['Keyboard', 'Headphones']  }
{ _id: { min: 950, max: 50000 }, proArray: ['Monitor', 'Laptop']       }
```

---

## ⚖️ `$bucket` vs `$bucketAuto` — Quick Comparison

| Feature | `$bucket` | `$bucketAuto` |
|---|---|---|
| Boundary definition | Manual (`boundaries` array) | Automatic |
| Number of buckets | Depends on boundaries | You specify `buckets: N` |
| `_id` format | Lower bound value | `{ min, max }` object |
| `default` field | Supported (and needed for out-of-range values) | Not needed |
| Use case | When you know the ranges | When you want equal distribution |

---

## ⚠️ Common Errors & Fixes

| Error | Cause | Fix |
|---|---|---|
| `SyntaxError: expected ","` | Missing `:` in key-value pair e.g. `price 450` | Use `price: 450` |
| `SyntaxError: Unexpected token` | Missing `:` in output field e.g. `averagePrice {$avg...}` | Use `averagePrice: {$avg...}` |
| `MongoServerError[Location7158303]` | Value falls outside all boundaries, no `default` set | Add `default: "label"` |
| `MongoServerError[Location40199]` | `default` value falls inside the boundary range (for strings) | Use a `default` value outside the range, or omit it |
| `averagePrice: null` | Accumulator references a field that doesn't exist in the collection | Check field name — e.g. use `"$price"` not `"$salary"` |
