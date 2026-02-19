# MongoDB Notes

## 1. Basic Collection Operations

```js
// Create a collection
db.createCollection("Customer");

// Count documents
db.Customer.countDocuments();

// Find all documents
db.Customer.find();
```

---

## 2. Inserting Documents

### Insert One
```js
db.Customer.insertOne({
  name: "Veer",
  email: "veerlodhi54@gmail.com",
  city: "Sagar"
});
```

Returns:
```
{ acknowledged: true, insertedId: ObjectId('...') }
```

### Insert and Capture the Inserted ID
```js
let result = db.Customer.insertOne({
  name: "Rajendra",
  email: "rajendra@gmail.com"
});
let customerId = result.insertedId;
```

> ✅ Use `insertedId` from the result to reference the new document in related collections.

---

## 3. Referencing Documents (Manual References)

MongoDB doesn't enforce foreign keys. You manually store the `_id` of one document in another:

```js
db.Orders.insertOne({
  customerId: customerId,  // ObjectId from Customer
  product: "Laptop",
  amount: 52000,
  status: "Delivered"
});
```

### ⚠️ Common Mistakes

| Mistake | Problem |
|--------|---------|
| `customerId: 101` (number) | No match with ObjectId — `$lookup` returns empty |
| `customerId: result._id` where `result = db.Collection.find(...)` | `find()` returns a cursor, not a document — `._id` is `undefined`/`null` |
| `customerId: result._id` where `result = db.Collection.findOne(...)` | ✅ Correct — `findOne()` returns a single document |

### ✅ Correct Way to Reference
```js
let result = db.Customer.findOne({ name: "Veer" });
db.Orders.insertOne({
  customerId: result._id,  // Valid ObjectId
  product: "xyz",
  amount: 2500
});
```

---

## 4. Deleting Documents

```js
// ❌ Wrong — passing string ID won't match ObjectId
db.Orders.deleteOne({ _id: "69968ff7e06fc9950b61942a" });
// deletedCount: 0

// ✅ Correct — wrap in ObjectId()
db.Orders.deleteOne({ _id: ObjectId("69968ff7e06fc9950b61942a") });
// deletedCount: 1
```

---

## 5. Aggregation — `$lookup` (JOIN)

`$lookup` is used to join two collections (like SQL JOIN).

### Syntax
```js
db.Orders.aggregate([
  {
    $lookup: {
      from: "Customer",        // Collection to join
      localField: "customerId", // Field in Orders
      foreignField: "_id",     // Field in Customer
      as: "OrderDetails"       // Output array field name
    }
  }
]);
```

### Result
Each order document gets an `OrderDetails` array containing the matched customer.

---

### Orders → Customer (Each order shows its customer)
```js
db.Orders.aggregate([
  {
    $lookup: {
      from: "Customer",
      localField: "customerId",
      foreignField: "_id",
      as: "OrderDetails"
    }
  }
]);
```

### Customer → Orders (Each customer shows their orders)
```js
db.Customer.aggregate([
  {
    $lookup: {
      from: "Orders",
      localField: "_id",
      foreignField: "customerId",
      as: "CustomerDetails"
    }
  }
]);
```

> ⚠️ Field names must be correct! A typo like `"cusmoterId"` instead of `"customerId"` returns empty arrays.

---

## 6. `$unwind` — Flatten Array

`$lookup` returns an array. Use `$unwind` to flatten it into individual documents.

```js
db.Orders.aggregate([
  {
    $lookup: {
      from: "Customer",
      localField: "customerId",
      foreignField: "_id",
      as: "OrderDetails"
    }
  },
  {
    $unwind: "$OrderDetails"   // ✅ Must prefix with $
  }
]);
```

> ⚠️ `$unwind: "OrderDetails"` (without `$`) throws an error:  
> `path option to $unwind stage should be prefixed with a '$'`

After `$unwind`, `OrderDetails` becomes an **object** instead of an **array**.

---

## 7. JavaScript in Mongo Shell

The Mongo shell supports JavaScript:

```js
let x = "Veer";
print(x);       // Veer

let a = 10, b = 20;
print(a + b);   // 30
```

> ❌ Java-style `System.out.println()` does NOT work — throws `ReferenceError: System is not defined`

---

## 8. Common Syntax Errors

| Mistake | Error |
|--------|-------|
| `db.Customer.find():` (colon instead of semicolon) | `SyntaxError: Missing semicolon` |
| `db.Orders.find())` (extra closing parenthesis) | `SyntaxError: Missing semicolon` |

---

## 9. Key Concepts Summary

| Concept | Notes |
|---------|-------|
| `insertOne()` | Returns `{ acknowledged, insertedId }` |
| `findOne()` | Returns a single document object |
| `find()` | Returns a cursor (not a document) |
| `$lookup` | Joins two collections using matching fields |
| `$unwind` | Flattens an array field into separate documents |
| `ObjectId("...")` | Required when querying/deleting by `_id` using a string |
| Manual references | Store `_id` of one doc inside another (no automatic FK enforcement) |
