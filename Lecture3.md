# MongoDB Regex Query Practice – Student Collection

This document contains MongoDB regex query commands tested on the `Student` collection.

---

## 🔍 Basic Name Regex Searches

```js
db.Student.find({name : /Veer/})
db.Student.find({name : /Vee/})
db.Student.find({name : /V/})
db.Student.find({name : /vr/})
db.Student.find({name : /veer/})
db.Student.find({name : /veer/i})
db.Student.find({name : /vr/i})
```

---

## 🔍 District Regex Searches

```js
db.Student.find({'address.district' : /sagar/})
db.Student.find({'address.district' : /sagar/i})
db.Student.find({'address.district' : /^s/i})
db.Student.find({'address.district' : /^a/i})
```

---

## 🔍 Starts With Regex

```js
db.Student.find({ name : /^ss/i})
db.Student.find({ name : /^s/i})
db.Student.find({ name : /^k/i})
db.Student.find({ name : /^ve/i})
db.Student.find({ name : /^e/i})
```

---

## 🔍 Ends With Regex

```js
db.Student.find({ name : /r$/i})
db.Student.find({ name : /a$/i})
```

---

## 🔍 Multiple Name Matching (OR using Regex)

```js
db.Student.find({ name : /Veer|Krishna|Rajendra/i})
db.Student.find({ name : /V|K|R/i})
db.Student.find({ name : /v|k|r/i})
db.Student.find({ name : /v|k|r/})
```

---

## 🔍 Word Boundary Regex

```js
db.Student.find({name : /\bVeer\b/})
db.Student.find({name : /\bSingh\b/})
db.Student.find({name : /\bLodhi\b/})
db.Student.find({name : /\bVeerSinghLodhi\b/})
```

---

## 🔍 Character Set Regex

```js
db.Student.find({'address.district' : /[sdb]/})
db.Student.find({'address.district' : /[SBA]/})
db.Student.find({'address.district' : /[S]/})
db.Student.find({'address.district' : /[SB]/})
```

---

## 🔍 Address Field Regex (String Address)

```js
db.Student.find({address : /[SB]/})
```

---

## 🔍 OR Condition With Regex

```js
db.Student.find({
  $or : [
    {adddress : /[SB]/},
    {'address.district' : /[SB]/}
  ]
})

db.Student.find({
  $or : [
    {adddress : /[RB]/},
    {'address.district' : /[SB]/}
  ]
})
```

---

## 🔍 Length Based Regex

```js
db.Student.find({name : /^.{4}$/})
db.Student.find({name : /^.{5}$/})
db.Student.find({name : /^.{6}$/})
db.Student.find({name : /^.{7}$/})
```

---

## 📌 Notes

- `/text/` → Case sensitive search  
- `/text/i` → Case insensitive search  
- `^` → Starts with  
- `$` → Ends with  
- `[]` → Character set  
- `|` → OR condition  
- `\b` → Word boundary  
- `.{n}` → Exact length  

---

## 🗄 Collection Used

```
Student
```

---

## ✅ Purpose

Practice MongoDB regex queries for:
- Name searching  
- Nested field searching  
- Pattern matching  
- Multiple condition matching  
- Length validation  

---

