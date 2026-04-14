# MongoDB Shell (mongosh) — Notes

## Connecting to MongoDB

```bash
# Connect without auth (if auth not enforced)
mongosh

# Connect with username and password
mongosh -u <username> -p <password>

# Connect with auth database specified
mongosh -u <username> -p <password> --authenticationDatabase admin

# Connect directly to a specific database
mongosh -u <username> -p <password> --authenticationDatabase admin <dbName>
```

> ⚠️ Always use `--authenticationDatabase admin` when user credentials are stored in the `admin` database.

---

## User Management

### View all users (requires admin access)
```js
use admin
db.getUsers()
```

### Create a user
```js
db.createUser({
  user: "username",
  pwd: "password",
  roles: [{ role: "readWrite", db: "targetDB" }]
})
```

### Update a user's roles
```js
db.updateUser("username", {
  roles: [{ role: "dbAdmin", db: "targetDB" }]
})
```

### Change a user's password
```js
db.changeUserPassword("username", "newPassword")
```

### Drop / Delete a user
```js
db.dropUser("username")
```

---

## Built-in Roles

| Role | Permissions |
|------|-------------|
| `read` | Read-only access to a database |
| `readWrite` | Read and write access |
| `dbAdmin` | Administrative tasks (no user management) |
| `userAdmin` | Manage users and roles (no data access) |
| `root` | Full access to everything |

> ⚠️ `userAdmin` does **not** grant read/write data access. Use `dbAdmin` or `readWrite` for data operations.

---

## Custom Roles

### Create a custom role
```js
use db1
db.createRole({
  role: "role1",
  privileges: [
    {
      resource: { db: "db1", collection: "Employee" },
      actions: ["insert"]
    },
    {
      resource: { db: "db1", collection: "Person" },
      actions: ["update"]
    },
    {
      resource: { db: "db1", collection: "Student" },
      actions: ["find"]
    },
    {
      resource: { db: "db1", collection: "Books" },
      actions: ["remove"]
    }
  ],
  roles: []
})
```

> 💡 Custom roles are **database-scoped** — a role created in `db1` must be assigned as `{ role: "role1", db: "db1" }`.

### Assign custom role to a user
```js
db.createUser({
  user: "accessDemo",
  pwd: "123",
  roles: [{ role: "role1", db: "db1" }]
})
```

---

## Collections

### List collections
```js
db.getCollectionNames()
```

### Create a collection
```js
db.createCollection("collectionName")
```

### Drop a collection
```js
db.collectionName.drop()
```

---

## CRUD Operations

### Insert
```js
// Insert one document into a collection
db.CollectionName.insertOne({ field: "value" })
```

> ⚠️ `db.insertOne()` is **not** valid — you must target a collection: `db.CollectionName.insertOne()`

### Read
```js
db.CollectionName.find()
db.CollectionName.find({ field: "value" })
```

### Delete
```js
// Delete one matching document
db.CollectionName.deleteOne({ field: "value" })

// Delete all matching documents
db.CollectionName.deleteMany({})
```

> ⚠️ `deleteOne()` and `deleteMany()` require a filter argument `{}` — passing no argument throws an error.

---

## Common Errors & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `Authentication failed` | Wrong password or wrong `--authenticationDatabase` | Use `--authenticationDatabase admin` |
| `not authorized on <db>` | User lacks permission for that database/action | Update user role via system/root account |
| `RoleNotFound: role1@demoDB` | Role was created in a different DB | Assign role with the correct DB: `{ role: "role1", db: "db1" }` |
| `db.insertOne is not a function` | Calling insert directly on `db` | Use `db.CollectionName.insertOne()` |
| `db.countCollections is not a function` | Method doesn't exist | Use `db.getCollectionNames()` |
| `Missing required argument` | Calling delete without a filter | Pass `{}` or a valid filter object |
| `SyntaxError: Unexpected token` | Typo in object syntax (missing `collection:` key) | Use `{ db: "x", collection: "y" }` in resource |

---

## Key Concepts

- **`authenticationDatabase`** — The database where the user's credentials are stored. Usually `admin`.
- **Roles are scoped** — A role assigned to `db: "companyDB"` only grants access to that database.
- **Custom roles created in `db1`** must be referenced as `{ role: "roleName", db: "db1" }`.
- **`userAdmin`** lets a user manage users but **cannot read/write data**.
- **`dbAdmin`** allows schema/index management but also **no read/write data access**.
- For full data + user management, combine roles or use `readWrite` + `userAdmin`.
