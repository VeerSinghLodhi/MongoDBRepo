# MongoDB Aggregation Pipeline — Complete Notes

## Table of Contents
1. [What is the Aggregation Pipeline?](#1-what-is-the-aggregation-pipeline)
2. [$match Stage](#2-match-stage)
3. [$project Stage](#3-project-stage)
4. [$group Stage](#4-group-stage)
5. [$count Stage](#5-count-stage)
6. [$sort Stage](#6-sort-stage)
7. [$limit and $skip Stages](#7-limit-and-skip-stages)
8. [Combining Multiple Stages](#8-combining-multiple-stages)
9. [Common Errors & Fixes](#9-common-errors--fixes)
10. [Quick Reference Table](#10-quick-reference-table)

---

## 1. What is the Aggregation Pipeline?

The aggregation pipeline is a framework in MongoDB that processes documents through a sequence of **stages**. Each stage transforms the documents and passes the result to the next stage.

```
db.collection.aggregate([
  { stage1 },
  { stage2 },
  { stage3 }
])
```

- Stages are applied **in order**, top to bottom.
- Each stage receives the output of the previous stage.
- You can chain as many stages as needed.

---

## 2. $match Stage

Filters documents — like a `find()` query inside the pipeline.

### Basic Syntax
```js
db.Books.aggregate([
  { $match: { field: value } }
])
```

### Examples

```js
// Match by _id
db.Books.aggregate([
  { $match: { _id: 1 } }
])

// Match by a string field
db.Books.aggregate([
  { $match: { title: "Java" } }
])

// Match by number
db.Books.aggregate([
  { $match: { pageCount: 400 } }
])

// Match with comparison operator
db.EmployeesData.aggregate([
  { $match: { salary: { $gt: 400000 } } }
])

// Match by department
db.EmployeesData.aggregate([
  { $match: { department: "Management" } }
])
```

### Key Rules
- Accepts the **same query operators** as `find()` (`$gt`, `$lt`, `$eq`, `$in`, etc.)
- Use `$match` **early in the pipeline** to reduce the number of documents passed to later stages (better performance).
- `$match` takes a plain filter object — **NOT** a nested `$match` key.

### Common Errors

| Wrong | Error | Fix |
|-------|-------|-----|
| `countDocuments($match: {...})` | SyntaxError | `countDocuments()` does not accept aggregation stages |
| `countDocuments({$match: {pageCount:400}})` | MongoServerError: unknown top level operator | `countDocuments` takes a plain filter: `countDocuments({pageCount:400})` |
| `{$match: {$department: "Mgmt"}}` | unknown top level operator: $department | Field names don't get `$` prefix in match: `{department: "Management"}` |

---

## 3. $project Stage

Controls which fields appear in the output. Can include/exclude fields and **rename** or **compute** new fields.

### Basic Syntax
```js
db.collection.aggregate([
  { $project: {
      fieldName: 1,   // include
      otherField: 0   // exclude
  }}
])
```

### Include / Exclude Fields

```js
// Include only isbn (excludes everything except _id by default)
db.Books.aggregate([
  { $project: { isbn: 1 } }
])

// Include title and isbn, hide _id
db.Books.aggregate([
  { $project: { _id: 0, title: 1, isbn: 1 } }
])
```

> **Note:** `_id` is always included unless explicitly set to `0`.

### Rename Fields
Use `"$fieldName"` as the value to reference a field under a new key:

```js
db.Student.aggregate([
  { $project: {
      studentName: "$name",       // rename "name" to "studentName"
      addressOfStudent: "$address" // rename "address"
  }}
])
```

- If a document doesn't have the referenced field, the key is **omitted** from that document's output (not null).
- Typos in field references (e.g., `"$adddress"`) silently return nothing — always double-check spelling.

### Access Nested Fields
Use dot notation with `$`:

```js
db.Student.aggregate([
  { $project: {
      studentName: "$name",
      district: "$address.district"   // nested field
  }}
])
```

### Filter + Project Together
Combine `$match` and `$project` stages:

```js
db.Books.aggregate([
  { $match: { title: "Java" } },
  { $project: { _id: 0, title: 1, isbn: 1 } }
])
```

### Projecting a Non-Existent Field
If you project a field that doesn't exist in a document or in any document, you get back only `_id` (or nothing if `_id: 0`):

```js
db.Books.aggregate([
  { $project: { address: 1 } }
])
// Returns: { _id: 1 }, { _id: 2 }, ... (address doesn't exist in Books)
```

---

## 4. $group Stage

Groups documents by a field and applies **accumulator functions** (sum, avg, min, max, count, etc.).

### Basic Syntax
```js
db.collection.aggregate([
  { $group: {
      _id: "$groupByField",      // grouping key (required)
      resultField: { $accumulator: "$field" }
  }}
])
```

- `_id` is **required** and defines the grouping key.
- Setting `_id` to a **string literal** (not prefixed with `$`) groups ALL documents together.
- Setting `_id` to `"$fieldName"` groups by that field's value.

### Accumulator Functions

| Accumulator | Purpose |
|-------------|---------|
| `$sum` | Sum of values (use `1` to count) |
| `$avg` | Average |
| `$min` | Minimum value |
| `$max` | Maximum value |
| `$count: {}` | Count of documents (takes **no argument**) |
| `$push` | Collect values into an array |
| `$first` | First value in group |
| `$last` | Last value in group |

### Examples

```js
// Total salary by department
db.EmployeesData.aggregate([
  { $group: {
      _id: "$department",
      totalSalary: { $sum: "$salary" }
  }}
])

// Average salary by department
db.EmployeesData.aggregate([
  { $group: {
      _id: "$department",
      avgSalary: { $avg: "$salary" }
  }}
])

// Min salary by department
db.EmployeesData.aggregate([
  { $group: {
      _id: "$department",
      minSalary: { $min: "$salary" }
  }}
])

// Max salary by department
db.EmployeesData.aggregate([
  { $group: {
      _id: "$department",
      maxSalary: { $max: "$salary" }
  }}
])

// Grand total (all documents in one group)
db.EmployeesData.aggregate([
  { $group: {
      _id: "total",         // literal string = one group for all docs
      totalSalary: { $sum: "$salary" }
  }}
])
// Output: { _id: 'total', totalSalary: 215350000 }

// Using $id with a non-existent field → groups all into null
db.EmployeesData.aggregate([
  { $group: {
      _id: "$veer",         // field doesn't exist → _id: null
      totalSalary: { $sum: "$salary" }
  }}
])
// Output: { _id: null, totalSalary: 215350000 }
```

### Counting Documents per Group
`$count` inside `$group` takes **no argument** (just empty object `{}`):

```js
// WRONG - throws TypeMismatch error
{ numberOfEmployees: { $count: "$department" } }
{ numberOfEmployees: { $count: "department" } }

// CORRECT
{ numberOfEmployees: { $count: {} } }
```

### Common Errors

| Wrong | Error | Fix |
|-------|-------|-----|
| `{ salary: { avg: "$salary" } }` | Location40234: must be an accumulator object | Use `$avg` not `avg` |
| `{ $count: "$department" }` | TypeMismatch: $count takes no arguments | Use `{ $count: {} }` |

---

## 5. $count Stage

Counts the **total number of documents** that pass through the pipeline at that point.

### Basic Syntax
```js
db.collection.aggregate([
  { $count: "fieldName" }
])
```

- `fieldName` is a string — the name of the output field that will hold the count.
- Returns a **single document** with that field.

### Examples

```js
// Count all employees
db.EmployeesData.aggregate([
  { $count: "total" }
])
// Output: { total: 100 }

// Count employees in Management
db.EmployeesData.aggregate([
  { $match: { department: "Management" } },
  { $count: "total" }
])
// Output: { total: 12 }

// Count employees with salary > 400000
db.EmployeesData.aggregate([
  { $match: { salary: { $gt: 400000 } } },
  { $count: "higherSalaryEmployees" }
])
// Output: { higherSalaryEmployees: 96 }
```

### $count vs countDocuments()
- `countDocuments({filter})` — simple count with a filter, **not** part of aggregation pipeline.
- `$count` — pipeline stage, used **inside** `aggregate()`, can be chained after `$match`, `$group`, etc.

### Common Errors

| Wrong | Error | Fix |
|-------|-------|-----|
| `{{ $count: "Total" }}` (double braces) | SyntaxError | Use single braces: `{ $count: "Total" }` |
| `skip: 10` (missing $) | MongoServerError: Unrecognized pipeline stage name: 'skip' | Add `$`: `{ $skip: 10 }` |

---

## 6. $sort Stage

Sorts documents in the pipeline.

### Basic Syntax
```js
db.collection.aggregate([
  { $sort: { field: 1 } }   // 1 = ascending, -1 = descending
])
```

### Examples

```js
// Sort by salary ascending (lowest first)
db.EmployeesData.aggregate([
  { $sort: { salary: 1 } }
])

// Sort by salary descending (highest first)
db.EmployeesData.aggregate([
  { $sort: { salary: -1 } }
])

// Sort after grouping
db.EmployeesData.aggregate([
  { $group: { _id: "$department", totalSalary: { $sum: "$salary" } } },
  { $sort: { totalSalary: -1 } }
])
```

### Sort Values

| Value | Order |
|-------|-------|
| `1` | Ascending (A→Z, 0→9) |
| `-1` | Descending (Z→A, 9→0) |

---

## 7. $limit and $skip Stages

### $limit
Returns only the first N documents from the pipeline.

```js
db.EmployeesData.aggregate([
  { $limit: 10 }    // returns first 10 documents
])
```

### $skip
Skips the first N documents and returns the rest.

```js
db.EmployeesData.aggregate([
  { $skip: 10 }    // skips first 10, returns from document 11 onwards
])
```

> **Note:** `$skip: 150` on a 100-document collection returns nothing (no error, just empty).

### Pagination Pattern
Combine `$skip` and `$limit` for pagination:

```js
// Page 2 (10 results per page)
db.EmployeesData.aggregate([
  { $skip: 10 },
  { $limit: 10 }
])
```

### Common Error

| Wrong | Error | Fix |
|-------|-------|-----|
| `{ skip: 10 }` | Unrecognized pipeline stage name: 'skip' | Use `{ $skip: 10 }` |

---

## 8. Combining Multiple Stages

Stages are chained together. The output of one stage feeds directly into the next.

### Example 1: Filter → Transform
```js
// Add computed field for Management employees only
db.EmployeesData.aggregate([
  { $match: { department: "Management" } },
  { $addFields: { annualSalary: { $multiply: ["$salary", 2] } } }
])
```

### Example 2: Filter → Count
```js
// Count Management employees
db.EmployeesData.aggregate([
  { $match: { department: "Management" } },
  { $count: "managementCount" }
])
```

### Example 3: Filter → Select Fields
```js
// Show only title and isbn for Java books
db.Books.aggregate([
  { $match: { title: "Java" } },
  { $project: { _id: 0, title: 1, isbn: 1 } }
])
```

### Example 4: Group → Sort → Limit (Top N pattern)
```js
// Top 5 departments by total salary
db.EmployeesData.aggregate([
  { $group: { _id: "$department", totalSalary: { $sum: "$salary" } } },
  { $sort: { totalSalary: -1 } },
  { $limit: 5 }
])
```

### Example 5: Filter → Sort → Skip → Limit (Pagination)
```js
// Second page of Engineering employees sorted by salary
db.EmployeesData.aggregate([
  { $match: { department: "Engineering" } },
  { $sort: { salary: -1 } },
  { $skip: 5 },
  { $limit: 5 }
])
```

---

## 9. Common Errors & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `Unrecognized pipeline stage name: 'skip'` | Missing `$` before stage | `$skip` not `skip` |
| `unknown top level operator: $match` | Using `$match` in `countDocuments()` | `countDocuments` takes a plain filter, not aggregation stages |
| `unknown top level operator: $department` | Using `$` on a field name in `$match` | Field names in `$match` don't use `$` prefix |
| `TypeMismatch: $count takes no arguments` | Passing argument to `$count` in `$group` | Use `{ $count: {} }` not `{ $count: "fieldName" }` |
| `Location40234: must be an accumulator object` | Missing `$` on accumulator | `$avg` not `avg`, `$sum` not `sum` |
| `ReferenceError: $count is not defined` | Writing `{ $count }` as shorthand | Write it in full: `{ $count: {} }` |
| SyntaxError on double braces | `{{ $count: "Total" }}` | Use single braces `{ $count: "Total" }` |
| Field silently missing in output | Typo in field name like `"$adddress"` | Check spelling carefully |
| `_id: null` instead of grouping | `_id: "$nonExistentField"` | Check field name exists in documents |

---

## 10. Quick Reference Table

| Stage | Purpose | Key Syntax |
|-------|---------|------------|
| `$match` | Filter documents | `{ $match: { field: value } }` |
| `$project` | Include/exclude/rename fields | `{ $project: { field: 1, other: 0 } }` |
| `$group` | Group and aggregate | `{ $group: { _id: "$field", result: { $sum: "$val" } } }` |
| `$count` | Count documents in pipeline | `{ $count: "labelName" }` |
| `$sort` | Sort documents | `{ $sort: { field: 1 } }` (1=asc, -1=desc) |
| `$limit` | Limit output to N docs | `{ $limit: 10 }` |
| `$skip` | Skip first N docs | `{ $skip: 5 }` |
| `$addFields` | Add computed fields | `{ $addFields: { newField: expression } }` |

### Accumulator Summary ($group)

| Accumulator | Usage |
|-------------|-------|
| `$sum` | `{ $sum: "$salary" }` or `{ $sum: 1 }` to count |
| `$avg` | `{ $avg: "$salary" }` |
| `$min` | `{ $min: "$salary" }` |
| `$max` | `{ $max: "$salary" }` |
| `$count` | `{ $count: {} }` (no argument!) |

### $project Field Reference Syntax

| Goal | Syntax |
|------|--------|
| Include field | `fieldName: 1` |
| Exclude field | `fieldName: 0` |
| Hide `_id` | `_id: 0` |
| Rename field | `newName: "$oldName"` |
| Nested field | `newName: "$parent.child"` |
