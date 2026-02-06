# MongoDB Employees Operations

## Commands and Queries

```js
db.createCollection("Employees");

db.Employees.insertOne({
  empId : 101,
  name : "Veer Singh Lodhi",
  department : "IT",
  designation : "Software Engineer",
  salary : 70000,
  experience : 2,
  city : "Sagar"
});

db.Employees.insertOne({
  empId : 102,
  name : "Muskan Shroti",
  department : "IT",
  designation : "Software Engineer",
  salary : 60000,
  experience : 1,
  city : "Sagar"
});

db.Employees.insertOne({
  empId : 103,
  name : "Krishna Kurmi",
  department : "HR",
  designation : "HR Executive",
  salary : 50000,
  experience : 3,
  city : "Bhopal"
});

db.Employees.insertOne({
  empId : 104,
  name : "Rajendra Kurmi",
  department : "Finance",
  designation : "Accountant",
  salary : 40000,
  experience : 2,
  city : "Ahmedabad"
});

db.Employees.insertOne({
  empId : 104,
  name : "Akash Pateriya",
  department : "Marketing",
  designation : "Marketing Manager",
  salary : 75000,
  experience : 6,
  city : "Delhi"
});

db.Employees.insertOne({
  empId : 106,
  name : "Rohit Kumar",
  department : "IT",
  designation : "Senior Developer",
  salary : 90000,
  experience : 7,
  city : "Bengaluru"
});

db.Employees.find();

db.Employees.find({department : "HR"});

db.Employees.find({salary : {$gt:50000}});

db.Employees.find({experience : {$lt:5}});

db.Employees.find({
  $and : [
    {salary : {$gt:40000} },
    {salary : {$lt:70000}}
  ]
});

db.Employees.find({
  $and : [
    { department : "IT"},
    { experience : {$gt:2}}
  ]
});

db.Employees.find({
  $or : [
    {department : "Finance"},
    {salary : {$gt:80000}}
  ]
});

db.Employees.find({
  $and : [
    {experience : {$gt:3}},
    {salary : {$lt:60000}}
  ]
});

db.Employees.find({
  $and : [
    {experience : {$gt:3}},
    {salary : {$lt:6000}}
  ]
});

db.Employees.find({
  $and : [
    {experience : {$gt:3}},
    {salary : {$gt:60000}}
  ]
});

db.Employees.find({
  $or : [
    {city : "Bhopal"},
    {designation : "Marketing Manager"}
  ]
});
