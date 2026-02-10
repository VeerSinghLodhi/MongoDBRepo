# MongoDB CRUD Operations Guide

This guide demonstrates various MongoDB operations performed on a Books collection, including querying, updating, and manipulating documents.

## Table of Contents
- [Initial Data](#initial-data)
- [Update Operations](#update-operations)
- [Field Management](#field-management)
- [Numeric Operations](#numeric-operations)
- [Array Operations](#array-operations)
- [Common Errors](#common-errors)

## Initial Data

### Sample Document Structure
```javascript
{
  _id: 1,
  title: 'Unlocking Android',
  isbn: '1933988673',
  pageCount: 416,
  publishedDate: ISODate("2009-04-01T07:00:00.000Z"),
  thumbnailUrl: 'https://s3.amazonaws.com/...',
  shortDescription: "...",
  longDescription: "...",
  status: 'PUBLISH',
  authors: ['W. Frank Ableson', 'Charlie Collins', 'Robi Sen'],
  categories: ['Open Source', 'Mobile']
}
```

## Update Operations

### 1. Updating Status Field
```javascript
// Update single document status
db.Books.updateOne(
  {isbn: '1933988673'}, 
  {$set: {status: "NOT PUBLISH"}}
);
```

**Result:**
```javascript
{
  acknowledged: true,
  matchedCount: 1,
  modifiedCount: 1
}
```

### 2. Updating Multiple Documents
```javascript
// Update all books with status 'PUBLISH'
db.Books.updateMany(
  {status: 'PUBLISH'}, 
  {$set: {price: 400}}
);
```

**Result:**
```javascript
{
  acknowledged: true,
  matchedCount: 362,
  modifiedCount: 362
}
```

### 3. Updating First Match Only
```javascript
// Updates only the first document that matches
db.Books.updateOne(
  {status: 'PUBLISH'}, 
  {$set: {pageCount: 1000}}
);
```

## Field Management

### Adding New Fields
```javascript
// Add a new field to a document
db.Books.updateOne(
  {isbn: '1933988673'}, 
  {$set: {puhlisher: 'Veer'}}
);
```

### Setting Fields to Null
```javascript
// Set a field to null
db.Books.updateMany(
  {status: 'NOT PUBLISH'}, 
  {$set: {city: null}}
);
```

### Removing Fields
```javascript
// Remove a field from a document
db.Books.updateOne(
  {isbn: '1933988673'}, 
  {$unset: {pageCount: ""}}
);
```

**Note:** The value in `$unset` doesn't matter; the field will be removed regardless.

## Replace Operations

### Replace Entire Document
```javascript
// Replace entire document (keeps _id)
db.Books.replaceOne(
  {isbn: '1933988673'}, 
  {
    title: 'Java',
    isbn: '1933988673',
    pageCount: 500
  }
);
```

**Result:** All fields except `_id` are replaced with the new document.

**Important:** `replaceMany()` does not exist in MongoDB. Use `updateMany()` instead.

## Numeric Operations

### Increment Field Value
```javascript
// Increase pageCount by 50
db.Books.updateOne(
  {isbn: '1933988673'}, 
  {$inc: {pageCount: 50}}
);
```

### Decrement Field Value
```javascript
// Decrease pageCount by 50
db.Books.updateOne(
  {isbn: '1933988673'}, 
  {$inc: {pageCount: -50}}
);
```

**Note:** `$inc` only works with numeric fields. Using it on a string field will result in an error.

## Array Operations

### Adding Elements to Arrays

#### Add Single Element
```javascript
// Add one author to the authors array
db.Books.updateOne(
  {_id: 2}, 
  {$push: {authors: "Veer"}}
);
```

**Before:**
```javascript
authors: ['W. Frank Ableson', 'Robi Sen']
```

**After:**
```javascript
authors: ['W. Frank Ableson', 'Robi Sen', 'Veer']
```

### Removing Elements from Arrays
```javascript
// Remove an author from the authors array
db.Books.updateOne(
  {_id: 2}, 
  {$pull: {authors: "Veer"}}
);
```

**Result:** Removes all occurrences of "Veer" from the authors array.

## Common Errors

### 1. Syntax Error - Extra Closing Brace
```javascript
// ❌ Incorrect
db.Books.find({status: 'PUBLISH'}}); // Extra }

// ✅ Correct
db.Books.find({status: 'PUBLISH'});
```

### 2. Using Update Operators in Find
```javascript
// ❌ Incorrect - Cannot use $set in find()
db.Books.find(
  {isbn: '1933988673'}, 
  {$set: {email: 'example@gmail.com'}}
);

// ✅ Correct - Use updateOne() instead
db.Books.updateOne(
  {isbn: '1933988673'}, 
  {$set: {email: 'example@gmail.com'}}
);
```

### 3. Missing Parentheses
```javascript
// ❌ Incorrect
db.Books.updateMany{status: 'PUBLISH'}, {$set: {price: 400}})

// ✅ Correct
db.Books.updateMany({status: 'PUBLISH'}, {$set: {price: 400}})
```

### 4. Update Without Atomic Operators
```javascript
// ❌ Incorrect
db.Books.updateOne({isbn: '1933988673'}, {_id: 101})

// ✅ Correct
db.Books.updateOne({isbn: '1933988673'}, {$set: {_id: 101}})
```

**Error:** "Update document requires atomic operators"

### 5. Using $inc on Non-Numeric Fields
```javascript
// ❌ Incorrect
db.Books.updateOne({isbn: '1933988673'}, {$inc: {title: 1}})
```

**Error:** "Cannot apply $inc to a value of non-numeric type"

### 6. Non-Existent Methods
```javascript
// ❌ Does not exist
db.Books.replaceMany(...)

// ✅ Use updateMany() instead
db.Books.updateMany(...)
```

## Query Operations

### Find One Document
```javascript
db.Books.findOne();
```

### Find by Criteria
```javascript
// Find by ISBN
db.Books.find({isbn: '1933988673'});

// Find by status
db.Books.find({status: 'PUBLISH'});

// Find by _id
db.Books.find({_id: 2});
```

## Key Takeaways

1. **updateOne()** - Updates the first matching document
2. **updateMany()** - Updates all matching documents
3. **replaceOne()** - Replaces entire document (no replaceMany)
4. **$set** - Sets field values
5. **$unset** - Removes fields
6. **$inc** - Increments/decrements numeric values
7. **$push** - Adds elements to arrays
8. **$pull** - Removes elements from arrays

## Best Practices

- Always use atomic operators (`$set`, `$unset`, etc.) in update operations
- Test updates with `updateOne()` before using `updateMany()`
- Use projection in `find()` to limit returned fields
- Verify changes with `find()` after updates
- Handle arrays with `$push` and `$pull` instead of replacing entire arrays

---

**Note:** All operations shown use the MongoDB shell syntax. Adapt as needed for your programming language driver.
