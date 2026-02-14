# MongoDB Array Operations - Command Reference

A comprehensive guide to working with arrays in MongoDB, including querying, updating, and manipulating array fields.

## Table of Contents
- [Basic Array Queries](#basic-array-queries)
- [Array Query Operators](#array-query-operators)
- [Array Update Operations](#array-update-operations)
- [Array Projection](#array-projection)
- [Array Element Matching](#array-element-matching)

---

## Basic Array Queries

### Adding Arrays to Documents

```javascript
// Add an array field to an existing document
db.Books.updateOne(
  {_id: 2},
  {$set: {years: [2000, 2005, 2010, 2015, 2020, 2025]}}
);

db.Books.updateOne(
  {_id: 4},
  {$set: {locations: ['Damoh', 'Sagar', 'Bina', 'Bhopal', 'Jabalput']}}
);
```

### Querying Array Elements

```javascript
// Find documents where array contains a specific value
db.Books.find({years: 2000});
db.Books.find({locations: 'Damoh'});

// Note: Array queries are case-sensitive
db.Books.find({locations: 'damoh'});  // Returns nothing
db.Books.find({locations: 'Damoh'});  // Returns matching documents
```

### Exact Array Match

```javascript
// Match exact array (order matters)
db.Books.find({locations: ['Damoh', 'Sagar', 'Bina', 'Bhopal', 'Jabalput']});

// This won't match if order or elements differ
db.Books.find({locations: ['Damoh', 'Sagar', 'Bina']});  // No results
```

---

## Array Query Operators

### $in Operator
Matches documents where array contains **any** of the specified values.

```javascript
// Find documents with ANY of these locations
db.Books.find({locations: {$in: ['Sagar', 'Bhopal', 'Bina']}});

// Multiple IDs
db.Books.find({_id: {$in: [2, 4]}});
```

### $all Operator
Matches documents where array contains **all** of the specified values (order doesn't matter).

```javascript
// Must contain all three cities
db.Books.find({locations: {$all: ['Sagar', 'Bhopal', 'Bina']}});

// Single element
db.Books.find({locations: {$all: ['Sagar']}});

// If any element is missing, no match
db.Books.find({locations: {$all: ['Sagar', 'Veer', 'Bina']}});  // No results
```

**Key Difference:**
- `$in`: OR logic - matches if **any** value exists
- `$all`: AND logic - matches only if **all** values exist

### $size Operator
Matches arrays with a specific number of elements.

```javascript
// Find arrays with exactly 3 elements
db.Countries.find({countries: {$size: 3}});

// Different sizes
db.Countries.find({countries: {$size: 4}});  // No match if size != 4
db.Countries.find({countries: {$size: 2}});  // No match if size != 2
```

**Important Limitations:**
```javascript
// ❌ Negative values not allowed
db.Countries.find({countries: {$size: -3}});  
// Error: Expected a non-negative number

// ❌ Cannot use comparison operators with $size
db.Countries.find({countries: {$size: {$gt: 2}}});  
// Error: Expected a number in $size
```

---

## Array Update Operations

### $pop Operator
Removes the first or last element from an array.

```javascript
// Remove last element (1)
db.Countries.updateOne({}, {$pop: {countries: 1}});

// Remove first element (-1)
db.Countries.updateOne({}, {$pop: {countries: -1}});

// Result: array becomes shorter by one element
```

**Before and After Example:**
```javascript
// Before: countries: ['Germany', 'Italy', 'Spain']
db.Countries.updateOne({}, {$pop: {countries: 1}});
// After: countries: ['Germany', 'Italy']

db.Countries.updateOne({}, {$pop: {countries: -1}});
// After: countries: ['Italy']
```

### Updating Array Elements by Index

```javascript
// Update element at specific index (0-based)
db.Countries.updateOne(
  {},
  {$set: {'countries.0': 'Denmark'}}
);

// If index doesn't exist, creates null elements
db.Countries.updateOne(
  {},
  {$set: {'countries.5': 'Denmark'}}
);
// Result: ['India', null, null, null, null, 'Denmark']
```

---

## Array Projection

### $slice Operator
Limits the number of array elements returned.

```javascript
// Get first N elements
db.Countries.find({}, {countries: {$slice: 1}});   // First element
db.Countries.find({}, {countries: {$slice: 2}});   // First 2 elements
db.Countries.find({}, {countries: {$slice: 3}});   // First 3 elements

// Get last N elements (negative number)
db.Countries.find({}, {countries: {$slice: -1}});  // Last element
db.Countries.find({}, {countries: {$slice: -2}});  // Last 2 elements
db.Countries.find({}, {countries: {$slice: -3}});  // Last 3 elements

// Get 0 elements (empty array)
db.Countries.find({}, {countries: {$slice: 0}});
```

### $slice with Skip and Limit

```javascript
// Syntax: {$slice: [skip, limit]}
// Skip 1, return 2 elements
db.Countries.find({}, {countries: {$slice: [1, 2]}});

// Skip 2, return 3 elements
db.Countries.find({}, {countries: {$slice: [2, 3]}});

// Skip 0, return 3 elements (same as {$slice: 3})
db.Countries.find({}, {countries: {$slice: [0, 3]}});

// Negative skip (from end)
db.Countries.find({}, {countries: {$slice: [-1, 2]}});
```

**Example with Data:**
```javascript
// Array: ['Germany', 'Italy', 'Spain']

{countries: {$slice: [1, 2]}}   // ['Italy', 'Spain']
{countries: {$slice: [2, 3]}}   // ['Spain']
{countries: {$slice: [0, 3]}}   // ['Germany', 'Italy', 'Spain']
{countries: {$slice: [-1, 2]}}  // ['Spain']
```

---

## Array Element Matching

### $elemMatch Operator
Matches documents where at least one array element satisfies all conditions.

```javascript
// Create sample data
db.Employee.insertOne({
  name: "Veerendra",
  marks: [
    {subject: "Maths", score: 33},
    {subject: "Science", score: 50},
    {subject: "English", score: 99}
  ]
});

// Find students with English score = 99
db.Employee.find({
  marks: {
    $elemMatch: {
      subject: "English",
      score: 99
    }
  }
});

// Find students with English score > 90
db.Employee.find({
  marks: {
    $elemMatch: {
      subject: "English",
      score: {$gt: 90}
    }
  }
});

// Find students with English score < 98
db.Employee.find({
  marks: {
    $elemMatch: {
      subject: "English",
      score: {$lt: 98}
    }
  }
});
```

**Why $elemMatch is Important:**
Without `$elemMatch`, conditions might match different array elements:
```javascript
// ❌ Without $elemMatch - can match different elements
db.Employee.find({
  "marks.subject": "English",
  "marks.score": {$gt: 90}
});
// Could match: English:50, Math:95 (different elements!)

// ✅ With $elemMatch - all conditions must match same element
db.Employee.find({
  marks: {
    $elemMatch: {
      subject: "English",
      score: {$gt: 90}
    }
  }
});
// Only matches if the SAME element has both subject="English" AND score>90
```

---

## Field Projection Basics

### Including Fields

```javascript
// Include only specific fields
db.Books.find({_id: 1}, {title: 1});
// Returns: {_id: 1, title: 'Java'}

// Exclude _id field
db.Books.find({_id: 1}, {title: 1, _id: 0});
// Returns: {title: 'Java'}
```

### Projection Rules

```javascript
// ❌ Cannot mix inclusion and exclusion (except for _id)
db.Books.find({_id: 1}, {title: 1, isbn: 0});
// Error: Cannot do exclusion on field isbn in inclusion projection

// ✅ Can exclude _id in inclusion projection
db.Books.find({_id: 1}, {title: 1, _id: 0});  // Valid
```

---

## Common Patterns and Use Cases

### Pattern 1: Finding Documents with Specific Array Values

```javascript
// Find books available in Sagar
db.Books.find({locations: 'Sagar'});

// Find books available in multiple cities
db.Books.find({locations: {$all: ['Sagar', 'Bina']}});
```

### Pattern 2: Array Size Validation

```javascript
// Find documents with exactly 3 countries
db.Countries.find({countries: {$size: 3}});

// Note: For size ranges, use aggregation pipeline
db.Countries.aggregate([
  {$match: {$expr: {$gt: [{$size: "$countries"}, 2]}}}
]);
```

### Pattern 3: Limited Array Results

```javascript
// Get only first 2 elements for all documents
db.Countries.find({}, {countries: {$slice: 2}});

// Skip first element, get next 2
db.Countries.find({}, {countries: {$slice: [1, 2]}});
```

### Pattern 4: Complex Array Element Queries

```javascript
// Students with English score between 90-100
db.Employee.find({
  marks: {
    $elemMatch: {
      subject: "English",
      score: {$gte: 90, $lte: 100}
    }
  }
});
```

---

## Quick Reference Table

| Operator | Purpose | Example |
|----------|---------|---------|
| `$in` | Match any value | `{arr: {$in: [1,2,3]}}` |
| `$all` | Match all values | `{arr: {$all: [1,2,3]}}` |
| `$size` | Exact array size | `{arr: {$size: 3}}` |
| `$slice` | Limit returned elements | `{arr: {$slice: 2}}` |
| `$pop` | Remove first/last | `{$pop: {arr: 1}}` |
| `$elemMatch` | Match array element | `{arr: {$elemMatch: {x:1}}}` |

---

## Common Errors and Solutions

### Error 1: Syntax Error in find()
```javascript
// ❌ Missing curly braces
db.Books.find(_id: {$in: [2,4]})

// ✅ Correct syntax
db.Books.find({_id: {$in: [2,4]}})
```

### Error 2: $size with Comparisons
```javascript
// ❌ Cannot use operators with $size
db.Countries.find({countries: {$size: {$gt: 2}}});

// ✅ Use aggregation for size comparisons
db.Countries.aggregate([
  {$match: {$expr: {$gt: [{$size: "$countries"}, 2]}}}
]);
```

### Error 3: Mixing Projection Types
```javascript
// ❌ Cannot mix inclusion and exclusion
db.Books.find({}, {title: 1, isbn: 0});

// ✅ Use only inclusion or only exclusion
db.Books.find({}, {title: 1, _id: 0});  // Include title, exclude _id
db.Books.find({}, {isbn: 0});           // Exclude only isbn
```

---

## Additional Notes

- Array queries are **case-sensitive**
- Array element order matters for exact matches
- Use `$all` when all values must exist (AND logic)
- Use `$in` when any value must exist (OR logic)
- `$size` only accepts non-negative integers
- `$pop` removes one element at a time
- `$elemMatch` ensures all conditions match the same array element
- Document count: Use `db.collection.countDocuments()` to get total documents

---

## Example Collections Used

```javascript
// Books collection
db.Books.find({_id: 2});
// Contains: years[], locations[], authors[], categories[]

// Countries collection
db.Countries.find();
// Contains: countries[]

// Employee collection
db.Employee.find();
// Contains: marks[] with nested objects
```

---

**Last Updated:** February 2026  
**MongoDB Version:** Compatible with MongoDB 4.0+
