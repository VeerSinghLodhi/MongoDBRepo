# 📘 MongoDB Complete Practice Notes – Student Database

---

# 📌 Database Selection

```js
use db1
```

---

# 📌 Create Collection

```js
db.createCollection("Student")
```

---

# 📌 Insert Single Documents

```js
db.Student.insertOne({ studentName: "Veer", rollno: 101 })

db.Student.insertOne({ studentName: "Krishna", rollno: 102 })

db.Student.insertOne({ studentName: "Rajendra", rollno: 103, address: "Sagar" })

db.Student.insertOne({
  studentName: "Muskan",
  rollno: 104,
  address: "Sagar",
  email: "muskan@gmail.com"
})

db.Student.insertOne({
  studentName: "Abhay",
  rollno: 105,
  address: "Bhopal",
  email: "abhay@gmail.com",
  phone: "9770225248"
})
```

---

# 📌 Insert Many Documents

```js
db.Student.insertMany([
  { name: "Veer", age: 20 },
  { name: "Krishna", age: 21, phone: "9770225248" }
])
```

---

# 📌 Find Data

```js
db.Student.find()
db.Student.findOne()
```

---

# 📌 Custom _id Example

```js
db.xyz.insertOne({ _id: 101, name: "Veer" })
db.xyz.insertOne({ _id: 102, name: "Krishan" })
db.xyz.insertOne({ _id: "102", name: "Krishan" })
```

⚠ `_id` must be unique

---

# 📌 Array Data Insert

```js
db.student.insertOne({
  course: "Java",
  modes: ["Online", "Offline"]
})
```

```js
db.student.insertOne({
  course: "Installments",
  amounts: [2000, 1280, 1500, 2600]
})
```

---

# 📌 Nested Array Documents

```js
db.courses.insertOne({
  courseName: "Java",
  students: [
    { name: "Veer" },
    { name: "Krishna", fee: 4600 }
  ]
})
```

---

# 📌 Nested Object Insert

```js
db.Student.insertOne({
  name: "Veer",
  address: {
    district: "Sagar",
    street: "Gopalganj",
    pin: 470001
  }
})
```

---

# 📌 Count Documents

```js
db.Student.countDocuments()
```

---

# 📌 Drop Collection

```js
db.Demo.drop()
```

---

# 📌 Query With Condition

```js
db.Student.find({ name: "Veer" })
```

⚠ Case Sensitive

```js
db.Student.find({ name: "veer" }) // No result
```

---

# 📌 Nested Field Query

```js
db.Student.find({ "address.pin": 470001 })
```

---

# 📌 Projection

```js
db.Student.find({}, { name: 1, age: 1 })

db.Student.find({ name: "Veer" }, { age: 1, _id: 0 })
```

---

# 📌 Date Handling

## ✅ Correct Date Insert

```js
db.Student.insertOne({
  name: "Veer",
  dob: new Date("2005-10-16")
})
```

---

## ❌ Wrong Date Insert

```js
dob: Date("2005-10-16")
```

---

## 📌 Find Using Date

```js
db.Student.find({
  dob: new Date("2005-10-16")
})
```

---

# 📌 ObjectId Query

```js
db.Student.find({
  _id: ObjectId("6984154c8110b00c86609bad")
})
```

---

# 📌 Comparison Operators

## Equal
```js
{ age: { $eq: 20 } }
```

## Not Equal
```js
{ age: { $ne: 25 } }
```

## Greater Than
```js
{ age: { $gt: 20 } }
```

## Less Than
```js
{ age: { $lt: 25 } }
```

---

# 📌 Logical Operators

## AND Operator
```js
db.Student.find({
  $and: [
    { name: "Veer" },
    { age: { $gt: 18 } }
  ]
})
```

---

## OR Operator
```js
db.Student.find({
  $or: [
    { name: "Veer" },
    { "address.district": "Sagar" }
  ]
})
```

---

# 📌 Deep Nested Query

```js
db.Student.find({
  "address.sagar.street": "Gopalganj"
})
```

---

# 📌 Common MongoDB Mistakes

## ❌ Wrong
```js
db.Student.find(name: "Veer")
```

## ✅ Correct
```js
db.Student.find({ name: "Veer" })
```

---

## ❌ Missing Quotes in Nested Field
```
address.district
```

## ✅ Correct
```
"address.district"
```

---

# 📌 Mixed Data Problem

❌ String Date  
```
Thu Feb 05 2026 ...
```

✅ Proper Date  
```
2005-10-16T00:00:00.000Z
```

---

# 📌 Best Practices

✔ Always use `new Date()`  
✔ Keep same data types  
✔ `_id` must be unique  
✔ MongoDB is Case Sensitive  
✔ Use Dot Notation for nested fields  

---

# 📌 Important Interview Points ⭐

MongoDB is:
- Schema-less
- Document-based
- JSON-like storage
- Flexible but requires type consistency

---

# 🧠 Memory Tricks

👉 Dot Notation → Nested Field  
👉 ObjectId() → _id Search  
👉 new Date() → Date Search  
👉 Quotes → Required in Nested Path  

---

# 👨‍💻 Practice Context

Database: **db1**  
Collections:
- Student
- xyz
- courses
- Demo

Topics Covered:
- CRUD Operations
- Date Handling
- Operators
- Logical Queries
- Nested Documents
- ObjectId Queries
