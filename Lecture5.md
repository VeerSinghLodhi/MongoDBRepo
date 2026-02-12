# MongoDB Operations Reference Guide

A comprehensive collection of MongoDB shell commands and operations for reference.

## Table of Contents
- [Collection Overview](#collection-overview)
- [Read Operations](#read-operations)
- [Array Operations](#array-operations)
- [Field Operations](#field-operations)
- [Update Operations](#update-operations)
- [Delete Operations](#delete-operations)
- [Collection Management](#collection-management)

---

## Collection Overview

```javascript
// Count total documents in Books collection
db.Books.countDocuments();
// Output: 431

// Count documents in other collections
db.DummyCollection.countDocuments();  // 200
db.DummyData.countDocuments();        // 5641
```

---

## Read Operations

### Find Single Document

```javascript
// Find one document (returns first match)
db.Books.findOne();

// Find by specific ID
db.Books.findOne({_id: 2});

// Find returns null if not found
db.Books.findOne({_id: 37});
// Output: null
```

### Find Multiple Documents

```javascript
// Find documents with specific IDs
db.Books.find({_id: {$in: [1, 12, 3]}});

db.Books.find({_id: {$in: [42, 43, 45]}});

// Projection - return only specific fields
db.Books.find({}, {_id: 1});
// Returns only _id field for all documents

// Pagination
// Type "it" for more results when output is truncated
```

---

## Array Operations

### Adding to Arrays

#### $addToSet (No Duplicates)

```javascript
// ❌ Incorrect syntax (missing colon)
db.Books.updateOne({_id:2}, {$addtoset{authors : "Robi Lodhi"}});
// SyntaxError

db.Books.updateOne({_id:2}, {$addToSet{authors : "Robi Lodhi"}});
// SyntaxError

// ✅ Correct syntax
db.Books.updateOne({_id:2}, {$addToSet: {authors: "Robi Lodhi"}});
// Result: modifiedCount: 1

// Running again with same value - no modification (prevents duplicates)
db.Books.updateOne({_id:2}, {$addToSet: {authors: "Robi Lodhi"}});
// Result: modifiedCount: 0
```

#### $addToSet with $each (Multiple Values)

```javascript
// Add multiple unique values at once
db.Books.updateOne(
  {_id:2},
  {$addToSet: {authors: {$each: ["Robi Lodhi", "Veer", "Veerendra", "Abhay"]}}}
);
// Adds only new values, skips existing ones

// Note: MongoDB is case-sensitive
db.Books.updateOne(
  {_id:2},
  {$addToSet: {authors: {$each: ["Robi Lodhi", "Veer", "Veerendra", "abhay"]}}}
);
// "abhay" is added because it's different from "Abhay"
```

#### $push (Allows Duplicates)

```javascript
// Push adds value even if it already exists
db.Books.updateOne({_id:2}, {$push: {authors: "Robi Lodhi"}});
// Result: Creates duplicate "Robi Lodhi" in array
```

### Key Differences

| Operator | Duplicates | Use Case |
|----------|------------|----------|
| `$addToSet` | ❌ No | When you need unique values only |
| `$push` | ✅ Yes | When duplicates are allowed |

---

## Field Operations

### Renaming Fields

```javascript
// Rename field: pageCount → totalPages
db.Books.updateOne(
  {_id: 2},
  {$rename: {'pageCount': 'totalPages'}}
);

// ❌ Incorrect - value must be a string
db.Books.updateOne(
  {_id: 2},
  {$rename: {'totalPages': pageCount}}
);
// ReferenceError: pageCount is not defined

// ✅ Correct - rename back: totalPages → pageCount
db.Books.updateOne(
  {_id: 2},
  {$rename: {totalPages: 'pageCount'}}
);
```

---

## Update Operations

### Basic Update

```javascript
// Update existing document
db.Books.updateOne(
  {title: "Android in Action, Second Edition"},
  {$set: {price: 500}}
);
```

### Upsert (Update or Insert)

```javascript
// ❌ Common syntax errors
db.Books.updateOne(
  {title: "Android in Action, Second Edition"},
  {$set: {price: 500}},
  {$upset: true}  // Wrong: $upset
);

db.Books.updateOne(
  {_id: 1000, title: "Demo"},
  {$set: {price: 500}},
  {upset: true}   // Wrong: upset
);

// ✅ Correct syntax with upsert
db.Books.updateOne(
  {_id: 1000, title: "Demo"},
  {$set: {price: 500}},
  {upsert: true}
);
// Result: insertedId: 1000, upsertedCount: 1
// Creates new document if no match found

// Verify the insert
db.Books.findOne({_id: 1000});
// Output: { _id: 1000, title: 'Demo', price: 500 }
```

---

## Delete Operations

### Delete Single Document

```javascript
// Delete one document by ID
db.Books.deleteOne({_id: 1000});
// Result: deletedCount: 1
```

### Delete Multiple Documents

```javascript
// Delete all books with pageCount less than 100
db.Books.deleteMany({pageCount: {$lt: 100}});
// Result: deletedCount: 166

// After deletion
db.Books.countDocuments();
// Output: 265 (was 431)
```

### Delete with Logical Operators

```javascript
// $and - deletes documents matching ALL conditions
db.Books.deleteMany({
  $and: [
    {_id: 42},
    {_id: 44}
  ]
});
// Result: deletedCount: 0 (impossible to match both IDs)

// $or - deletes documents matching ANY condition
db.Books.deleteMany({
  $or: [
    {_id: 42},
    {_id: 44}
  ]
});
// Result: deletedCount: 1

db.Books.deleteMany({
  $or: [
    {_id: 39},
    {_id: 40}
  ]
});
// Result: deletedCount: 2
```

### Find and Delete

```javascript
// Find and delete in one operation (returns deleted document)
db.Books.findOneAndDelete({_id: 37});
// Returns the deleted document details

// Verify deletion
db.Books.findOne({_id: 37});
// Output: null
```

---

## Collection Management

### Check Collection Existence

```javascript
// Check if collection exists
db.Books.exists();
// Returns collection metadata if exists

db.Books.exists({_id: 37});  // Note: Ignores parameter, just checks collection
db.Books.exists({_id: 40});  // Same output regardless of document
db.Books.exists({_id: 30});  // Same output

// Non-existent collection
db.xyz.exists();
// Output: null
```

### Drop Collection

```javascript
// Drop (delete) entire collection
db.xyz.drop();
// Output: true (if successful)

// Verify
db.xyz.exists();
// Output: null
```

---

## Sample Document Structure

```javascript
{
  _id: 2,
  title: 'Android in Action, Second Edition',
  pageCount: 1000,
  publishedDate: 2011-01-14T08:00:00.000Z,
  thumbnailUrl: 'https://s3.amazonaws.com/AKIAJC5RLADLUMVRPFDQ.book-thumb-images/ableson2.jpg',
  shortDescription: '...',
  longDescription: '...',
  status: 'PUBLISH',
  authors: [
    'W. Frank Ableson',
    'Robi Sen',
    'Robi Lodhi',
    'Robi Lodhi',
    'Veer',
    'Veerendra',
    'Abhay',
    'abhay'
  ],
  categories: ['Java'],
  price: 400,
  isbn: '1935182722'
}
```

---

## Common Errors and Fixes

### Syntax Errors

```javascript
// ❌ Missing colon after operator
{$addtoset{authors: "value"}}

// ✅ Correct
{$addToSet: {authors: "value"}}

// ❌ Wrong option name
{upset: true}

// ✅ Correct
{upsert: true}

// ❌ Value not quoted
{$rename: {field: newName}}

// ✅ Correct
{$rename: {field: 'newName'}}
```

---

## Quick Reference

### CRUD Operations
- **Create**: `insertOne()`, `insertMany()`, or `updateOne()` with `upsert: true`
- **Read**: `find()`, `findOne()`
- **Update**: `updateOne()`, `updateMany()`
- **Delete**: `deleteOne()`, `deleteMany()`, `findOneAndDelete()`

### Array Operators
- `$addToSet`: Add unique values
- `$push`: Add values (allows duplicates)
- `$each`: Add multiple values

### Field Operators
- `$set`: Set field value
- `$rename`: Rename field
- `$unset`: Remove field

### Comparison Operators
- `$in`: Match any value in array
- `$lt`: Less than
- `$gt`: Greater than
- `$eq`: Equal to

### Logical Operators
- `$and`: Match all conditions
- `$or`: Match any condition

---

**Note**: This reference is based on MongoDB shell operations and demonstrates common patterns, syntax errors, and their corrections.
