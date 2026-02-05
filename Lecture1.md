# MongoDB Practice Commands – db1 Database

## 📌 Database Selection
```js
use db1
```

---

## 📌 Create Collection
```js
db.createCollection("Student")
```

---

## 📌 Insert Single Documents

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

## 📌 Find Data

```js
db.Student.find()
db.Student.findOne()
```

---

## 📌 Insert Many Documents

```js
db.Student.insertMany([{}])

db.Student.insertMany([
  { name: "Veer", age: 20 },
  { name: "Krishna", age: 21, phone: "9770225248" }
])
```

---

## 📌 Custom _id Example

```js
db.xyz.insertOne({ _id: 101, name: "Veer" })
db.xyz.insertOne({ _id: 102, name: "Krishan" })
db.xyz.insertOne({ _id: "102", name: "Krishan" })
```

⚠ Duplicate `_id` is not allowed.

---

## 📌 Array Data Insert

```js
db.student.insertOne({
  course: "Java",
  modes: ["Online", "Offline"]
})

db.student.insertOne({
  course: "Installments",
  amounts: [2000, 1280, 1500, 2600, 4600, 2200]
})
```

---

## 📌 Nested Array Documents

```js
db.courses.insertOne({
  courseName: "Java",
  students: [
    { name: "Veer" },
    { name: "Krishna", fee: 4600 }
  ]
})
```

```js
db.courses.insertOne({
  courseName: "Python",
  students: [
    { name: "Veer" },
    { name: "Ayush", fee: 4600 },
    { name: "Rajendra", address: "Sagar" }
  ]
})
```

```js
db.courses.insertOne({
  courseName: "Spring Boot",
  students: [
    { name: "Veer" },
    { name: "Ayush", fee: 4600 },
    { name: "Rajendra", address: "Sagar" },
    { name: "Krishna", fee: 2500 },
    { name: "Muskan", fee: 4600, email: "muskan@gmail.com" }
  ]
})
```

---

## 📌 Nested Object Insert

```js
db.Student.insertOne({
  name: "Veer",
  address: {
    district: "Sagar",
    street: "Gopalganj shree ram colony",
    pin: 470001
  }
})
```

```js
db.Student.insertOne({
  name: "Krishna",
  address: {
    district: "Sagar",
    street: "BMC",
    pin: 470002,
    phone: "7896541230",
    email: "krishna@gmail.com"
  }
})
```

---

## 📌 Count Documents

```js
db.Student.countDocuments()
```

---

## 📌 Drop Collection

```js
db.Demo.drop()
```

---

## 📌 Query With Condition

```js
db.Student.find({ name: "Veer" })

db.Student.findOne({ name: "Veer" })
```

⚠ Case Sensitive
```js
db.Student.findOne({ name: "veer" }) // null
```

---

## 📌 Query Nested Fields

```js
db.Student.findOne({ "address.pin": 470001 })
```

---

## 📌 Projection (Select Specific Fields)

```js
db.Student.find({}, { name: 1, age: 1 })

db.Student.find({ age: 20 }, { name: 1 })

db.Student.find({ name: "Veer" }, { age: 1, _id: 0 })
```

⚠ Cannot mix include and exclude (except `_id`)

---

## 📌 Common Errors

❌ Wrong  
```js
db.Student.find(name: "Veer")
```

✅ Correct  
```js
db.Student.find({ name: "Veer" })
```

---

## 📌 Notes

- MongoDB is **schema-less**
- `_id` must be unique
- Field names are **case-sensitive**
- Nested fields use **dot notation**

---

## ✅ Total Documents Example
```js
db.Student.countDocuments() // Example Output: 10
```

---

# 👨‍💻 Author Practice Notes
Database Used: **db1**  
Collections Used:
- Student
- xyz
- student
- courses
- Demo
