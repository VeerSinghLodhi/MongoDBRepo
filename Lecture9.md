# MongoDB `$addFields` Aggregation – Notes

> These notes cover the `$addFields` stage in MongoDB aggregation pipelines, including arithmetic, string, date, and conditional operators.

---

## 1. What is `$addFields`?

`$addFields` is an aggregation pipeline stage that **adds new fields** to existing documents (or overwrites existing ones) without removing other fields.

```js
db.collection.aggregate([
  {
    $addFields: {
      newField: <expression>
    }
  }
])
```

---

## 2. Arithmetic Operators

### `$add` — Add numeric fields and/or constants

**Syntax:** `{ $add: [expr1, expr2, ...] }`

> ⚠️ Use **array `[]`** syntax, NOT object `{}` syntax — `{$add: {"$salary","$age"}}` causes a SyntaxError.

```js
// ✅ Correct
{ $add: ["$salary", "$age"] }

// ❌ Wrong
{ $add: {"$salary", "$age"} }
```

**Examples:**

```js
// Add salary + age
{ totalSalary: { $add: ["$salary", "$age"] } }

// Add multiple numeric fields + a constant
{ totalSalary: { $add: ["$salary", "$age", "$employeeId", 500] } }

// Subtract using a negative constant
{ totalSalary: { $add: ["$salary", "$age", "$employeeId", -500] } }
```

> ⚠️ `$add` only supports **numeric or date** types — passing a string field (e.g., `"$city"`) throws:
> `MongoServerError: $add only supports numeric or date types, not string`

---

## 3. String Operators

### `$concat` — Concatenate strings

**Syntax:** `{ $concat: [str1, str2, ...] }`

> ⚠️ `$concat` only works with **strings** — passing a numeric field directly throws an error.
> To include numbers, first convert using `$toString`.

```js
// ✅ Works — only string fields
{ totalSalary: { $concat: ["Mr. ", "$firstName", " Ji"] } }

// ❌ Fails — $salary is a number
{ totalSalary: { $concat: ["Mr. ", "$salary"] } }
// Error: $concat only supports strings, not int

// ✅ Works — convert number to string first
{ totalSalary: { $concat: ["Mr. ", "$firstName", " Age is ", { $toString: "$age" }] } }
```

> ⚠️ If any argument in `$concat` resolves to a **non-string (including null)**, the result is `null`.
> e.g., `"$firstName!"` doesn't exist as a field, so it resolves to null → result is null.

---

### `$strLenCP` — Get string length (Unicode code points)

**Syntax:** `{ $strLenCP: "$fieldName" }` *(not an array)*

```js
// ✅ Correct
{ Length: { $strLenCP: "$firstName" } }

// ❌ Wrong — using array syntax
{ Length: { $strLenCP: ["$firstName"] } }

// ❌ Wrong — missing $ prefix (treats as literal string, not field)
{ Length: { $strLenCP: ["firstName"] } }  // Returns 9 (length of "firstName")
```

> Works with Unicode strings too — e.g., Hindi `"रवि"` returns `3`.

---

### `$toUpper` / `$toLower` — Change case

**Syntax:** `{ $toUpper: ["$field"] }` or `{ $toLower: ["$field"] }`

```js
{ NAME: { $toUpper: ["$firstName"] } }   // "Aarav" → "AARAV"
{ NAME: { $toLower: ["$firstName"] } }   // "Aarav" → "aarav"
```

---

## 4. Type Conversion

### `$toString` — Convert any value to string

Used to mix numeric fields inside `$concat`:

```js
{ $concat: ["Age: ", { $toString: "$age" }] }
```

---

## 5. Date Operators

> ⚠️ Date operators require **actual Date type** fields, not date strings like `"2020-05-15T09:00:00Z"`.
> Passing a string field throws: `can't convert from BSON type string to Date`.

### Extracting Date Parts

```js
{ $year:       ["$dob"] }   // e.g., 2026
{ $month:      ["$dob"] }   // e.g., 2
{ $dayOfMonth: ["$dob"] }   // e.g., 26
{ $dayOfWeek:  ["$dob"] }   // 1=Sunday ... 7=Saturday
{ $dayOfYear:  ["$dob"] }   // e.g., 57
{ $week:       ["$dob"] }   // Week number of year, e.g., 8
```

### `$$NOW` — Current date/time system variable

> ⚠️ Use `$$NOW` (uppercase), not `$$now` — lowercase throws an error.

```js
{ $year: ["$$NOW"] }        // Current year
{ $month: ["$$NOW"] }       // Current month
{ $dayOfYear: ["$$NOW"] }   // Current day of year
```

---

### `$dateAdd` — Add time to a date

**Syntax:**
```js
{
  $dateAdd: {
    startDate: <date>,
    unit: <string>,
    amount: <number>
  }
}
```

**Supported units:** `"millisecond"`, `"second"`, `"minute"`, `"hour"`, `"day"`, `"week"`, `"month"`, `"quarter"`, `"year"`

> ⚠️ `"quater"` (typo) throws an error — must be `"quarter"`

```js
// Add 5 days to now
{ newDate: { $dateAdd: { startDate: "$$NOW", unit: "day", amount: 5 } } }

// Add 5 years
{ newDate: { $dateAdd: { startDate: "$$NOW", unit: "year", amount: 5 } } }

// Add 5 quarters
{ newDate: { $dateAdd: { startDate: "$$NOW", unit: "quarter", amount: 5 } } }

// Add 2 hours
{ newDate: { $dateAdd: { startDate: "$$NOW", unit: "hour", amount: 2 } } }

// Add 2 weeks
{ newDate: { $dateAdd: { startDate: "$$NOW", unit: "week", amount: 2 } } }
```

---

### `$dateDiff` — Difference between two dates

**Syntax:**
```js
{
  $dateDiff: {
    startDate: <date>,
    endDate: <date>,
    unit: <string>
  }
}
```

> If `startDate` or `endDate` is `null`/missing, result is `null`.

```js
// Days between dob and a fixed date
{ newDate: { $dateDiff: { startDate: "$dob", endDate: new Date('2030-10-20'), unit: "day" } } }
// → 1697

// Years between dob and a fixed date
{ newDate: { $dateDiff: { startDate: "$dob", endDate: new Date('2030-10-20'), unit: "year" } } }
// → 4

// Hours between dob and fixed date
{ newDate: { $dateDiff: { startDate: "$dob", endDate: new Date('2030-10-20'), unit: "hour" } } }
// → 40724

// Years using a numeric expression as date (not recommended — use proper Date)
{ newDate: { $dateDiff: { startDate: new Date(2005-10-16), endDate: "$$NOW", unit: "year" } } }
// Note: new Date(2005-10-16) evaluates the math expression first → incorrect date
```

---

## 6. Conditional Operators

### `$cond` — If/Then/Else

**Syntax (object form):**
```js
{
  $cond: {
    if: <boolean-expression>,
    then: <value-if-true>,
    else: <value-if-false>
  }
}
```

> ⚠️ The `if` condition must be a **separate object** — don't mix `if:` and `then:` inside one object.

```js
// ✅ Correct
{
  $cond: {
    if: { $gt: ["$salary", 50000] },
    then: "High",
    else: "Low"
  }
}

// ❌ Wrong (syntax error — then/else inside the if-condition object)
{
  $cond: {
    if: { $gt: ["$salary", 50000],
    then: "High",       // ← SyntaxError
    else: "Low"
    }
  }
}
```

---

### `$switch` — Multi-branch conditional (like switch/case)

**Syntax:**
```js
{
  $switch: {
    branches: [
      { case: <expr>, then: <value> },
      { case: <expr>, then: <value> },
      ...
    ],
    default: <value>   // Required if any document might not match
  }
}
```

> ⚠️ `$switch` evaluates branches **in order** — first matching case wins. Later cases are never evaluated for already-matched documents.

> ⚠️ If no branch matches and there's **no `default`**, MongoDB throws:
> `$switch could not find a matching branch for an input, and no default was specified.`

```js
// ✅ Correct with default
{
  $switch: {
    branches: [
      { case: { $gte: ["$salary", 600000] }, then: "Full Stack Developer" },
      { case: { $gte: ["$salary", 500000] }, then: "UI Designer" },
      { case: { $gte: ["$salary", 700000] }, then: "Tester" }  // never reached if 600000 matches first
    ],
    default: "Others"
  }
}

// ❌ No default — throws error if no case matches
{
  $switch: {
    branches: [
      { case: { $gte: ["$salary", 600000] }, then: "Senior" }
    ]
    // Missing default!
  }
}
```

---

## 7. Common Errors & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `SyntaxError: Unexpected token` | Used `{}` instead of `[]` in `$add` | Use array: `[$add: ["$a", "$b"]]` |
| `$add only supports numeric or date types` | Passed a string field to `$add` | Only use numeric fields |
| `$concat only supports strings, not int` | Passed numeric field to `$concat` | Wrap with `{ $toString: "$field" }` |
| `$strLenCP requires a string argument, found: missing` | Missing `$` in field name | Use `"$fieldName"` not `"fieldName"` |
| `can't convert from BSON type string to Date` | Date field is stored as string | Store dates as `new Date()`, not strings |
| `Use of undefined variable: now` | Used `$$now` (lowercase) | Use `$$NOW` (uppercase) |
| `unknown time unit value: quater` | Typo in unit name | Use `"quarter"` (correct spelling) |
| `Unrecognized pipeline stage name: 'addFields'` | Missing `$` before stage name | Use `$addFields` not `addFields` |
| `$switch could not find a matching branch` | No `default` and no branch matched | Add `default: "value"` to `$switch` |
| Result is `null` in `$concat` | One argument resolves to null/missing | Check field names are correct |

---

## 8. Quick Reference Summary

| Operator | Type | Use For |
|----------|------|---------|
| `$add` | Arithmetic | Sum numeric fields + constants |
| `$concat` | String | Join strings (strings only) |
| `$strLenCP` | String | Length of string in Unicode code points |
| `$toUpper` | String | Convert to uppercase |
| `$toLower` | String | Convert to lowercase |
| `$toString` | Conversion | Convert number/date to string |
| `$year`, `$month`, `$dayOfMonth` | Date | Extract date parts |
| `$dayOfWeek`, `$dayOfYear`, `$week` | Date | Extract day/week info |
| `$$NOW` | System Variable | Current date-time |
| `$dateAdd` | Date | Add time interval to a date |
| `$dateDiff` | Date | Difference between two dates |
| `$cond` | Conditional | If/then/else logic |
| `$switch` | Conditional | Multi-branch conditions |

---

*Collection used in examples: `db1.EmployeesData` with fields: `employeeId`, `firstName`, `lastName`, `email`, `gender`, `age`, `department`, `designation`, `salary`, `city`, `skills`, `joiningDate`, `isRemote`*
