# MongoDB Notes — User Management & Basic CRUD

> Personal notes from hands-on practice with MongoDB 8.2.4 using `mongosh 2.8.2` on Windows

---

## Table of Contents
1. [Connecting to MongoDB](#1-connecting-to-mongodb)
2. [Creating Users](#2-creating-users)
3. [Logging in as a User](#3-logging-in-as-a-user)
4. [Common Syntax Mistakes](#4-common-syntax-mistakes)
5. [Listing Users](#5-listing-users)
6. [Basic CRUD Operations](#6-basic-crud-operations)
7. [Key Takeaways](#7-key-takeaways)

---

## 1. Connecting to MongoDB

### Connect without authentication (default/local)
```bash
mongosh
```

### Connect with authentication
```bash
mongosh -u <username> -p <password> --authenticationDatabase <db>
```

**Example:**
```bash
mongosh -u system -p manager --authenticationDatabase admin
```

> ⚠️ `mongosh` commands must be run from the **Windows terminal (CMD/PowerShell)**, NOT from inside the `mongosh` shell itself.

---

## 2. Creating Users

### Switch to the `admin` database first
```javascript
use admin
```

### Correct syntax for `createUser`
```javascript
db.createUser({
  user: "username",
  pwd: "password",
  roles: [{ role: "roleName", db: "targetDatabase" }]
})
```

**Real examples:**
```javascript
// Superuser (root access)
db.createUser({ user: "system", pwd: "manager", roles: [{ role: "root", db: "admin" }] })

// Regular read/write user on a specific DB
db.createUser({ user: "user1", pwd: "123", roles: [{ role: "readWrite", db: "companyDB" }] })
```

### Common roles
| Role        | Description                          |
|-------------|--------------------------------------|
| `root`      | Full access to everything            |
| `readWrite` | Read and write on a specific DB      |
| `read`      | Read-only on a specific DB           |
| `dbAdmin`   | Admin tasks on a specific DB         |

---

## 3. Logging in as a User

```bash
mongosh -u user1 -p 123 --authenticationDatabase admin
```

### ✅ Rule: Always use `admin` as `--authenticationDatabase`
Even if the user has roles on `companyDB` or `db1`, the user record is stored in `admin`. So always authenticate against `admin`.

```bash
# ✅ Correct — user is stored in admin
mongosh -u user1 -p 123 --authenticationDatabase admin

# ❌ Wrong — will fail even if user has a role on companyDB
mongosh -u user1 -p 123 --authenticationDatabase companyDB
```

### Verify your logged-in session
```javascript
db.runCommand({ connectionStatus: 1 })
```

---

## 4. Common Syntax Mistakes

### ❌ Missing curly braces `{}` around role object
```javascript
// WRONG
roles: [role: "root", db: "admin"]

// ✅ CORRECT
roles: [{ role: "root", db: "admin" }]
```

### ❌ Mismatched brackets
```javascript
// WRONG
roles: [{ role: "root", db: "admin" ]}   // ] before }

// ✅ CORRECT
roles: [{ role: "root", db: "admin" }]   // } before ]
```

### ❌ Running shell commands inside mongosh
```javascript
// WRONG (typed inside mongosh — will throw SyntaxError)
mongosh -u user1 -p 123 --authenticationDatabase admin

// ✅ CORRECT — run this from CMD/PowerShell, not inside mongosh
```

### ❌ Using `updateOne` without an update operator
```javascript
// WRONG
db.Demo.updateOne({ name: "veersagar" }, { name: "veer" })

// ✅ CORRECT — use $set
db.Demo.updateOne({ name: "veersagar" }, { $set: { name: "veer" } })
```

---

## 5. Listing Users

```javascript
use admin
db.getUsers()
```

This shows all users, their `db`, `roles`, and `mechanisms`.

---

## 6. Basic CRUD Operations

> Make sure you're on the right database first: `use companyDB`

### Insert
```javascript
db.CollectionName.insertOne({ field: "value" })
db.CollectionName.insertMany([{ field: "v1" }, { field: "v2" }])
```

### Read
```javascript
db.CollectionName.find()                        // all documents
db.CollectionName.find({ name: "veer" })        // filtered
```

### Update
```javascript
// Update first match
db.CollectionName.updateOne(
  { name: "veersagar" },         // filter
  { $set: { name: "veer" } }    // update operator required!
)

// Update all matches
db.CollectionName.updateMany(
  { city: "Delhi" },
  { $set: { city: "Mumbai" } }
)
```

### Delete
```javascript
db.CollectionName.deleteOne({ name: "veer" })   // deletes first match
db.CollectionName.deleteMany({ name: "veer" })  // deletes all matches
```

> ⚠️ `deleteOne()` requires a filter argument. Calling it without one throws an error.

---

## 7. Key Takeaways

- Always create users while connected to the **`admin`** database.
- Always authenticate with `--authenticationDatabase admin` even for users with roles on other DBs.
- The `roles` field must be an **array of objects**: `[{ role: "...", db: "..." }]`
- Shell commands like `mongosh` must be run from CMD/PowerShell — **not inside the mongosh shell**.
- `updateOne` / `updateMany` require an **update operator** like `$set`, `$inc`, `$unset`, etc.
- Only users with `root` or `userAdminAnyDatabase` roles can create new users via `db.createUser()`.

---

*MongoDB version: 8.2.4 | Mongosh version: 2.8.2 | OS: Windows 10*
