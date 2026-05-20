# Backend Notes: MongoDB, Mongoose Models, and Schemas

These notes explain MongoDB connection, Mongoose schemas, models, and how they fit into an Express backend.

Goal: understand how to connect Express to MongoDB and create clean models/schemas for your backend projects.

---

## Table of Contents

1. [What Is MongoDB?](#1-what-is-mongodb)
2. [What Is Mongoose?](#2-what-is-mongoose)
3. [MongoDB Connection String](#3-mongodb-connection-string)
4. [Connecting Express to MongoDB](#4-connecting-express-to-mongodb)
5. [What Is a Schema?](#5-what-is-a-schema)
6. [What Is a Model?](#6-what-is-a-model)
7. [Creating a User Model](#7-creating-a-user-model)
8. [Creating a Todo Model](#8-creating-a-todo-model)
9. [Common Mongoose Schema Options](#9-common-mongoose-schema-options)
10. [Using Models in Services](#10-using-models-in-services)
11. [Full Express + MongoDB + Mongoose Example](#11-full-express--mongodb--mongoose-example)
12. [Practice Tasks](#12-practice-tasks)

---

# 1. What Is MongoDB?

MongoDB is a NoSQL database.

It stores data as documents.

A document looks similar to JSON.

Example user document:

```json
{
  "_id": "665f1a...",
  "name": "Abhishek",
  "email": "abhishek@example.com",
  "createdAt": "2026-05-20T10:00:00.000Z"
}
```

MongoDB is commonly used in MERN stack apps:

```txt
MongoDB
Express
React
Node.js
```

Good for:

- CRUD apps
- Chat apps
- Social apps
- Content apps
- Fast prototyping
- Apps where data shape may change

---

# 2. What Is Mongoose?

Mongoose is an ODM for MongoDB.

ODM means:

```txt
Object Data Modeling
```

Mongoose helps you:

- Connect to MongoDB
- Define schemas
- Create models
- Validate data
- Run queries
- Use middleware/hooks
- Add timestamps

Install:

```bash
npm install mongoose
```

---

# 3. MongoDB Connection String

## Local MongoDB URL

```env
DATABASE_URL=mongodb://localhost:27017/todo-app
```

Breakdown:

```txt
mongodb://localhost:27017/todo-app
```

- `mongodb://` = protocol
- `localhost` = database running on your machine
- `27017` = default MongoDB port
- `todo-app` = database name

---

## MongoDB Atlas URL

MongoDB Atlas is cloud MongoDB.

Example:

```env
DATABASE_URL=mongodb+srv://username:password@cluster0.mongodb.net/todo-app
```

---

# 4. Connecting Express to MongoDB

Create:

```txt
config/db.js
```

```js
const mongoose = require("mongoose");

async function connectDB() {
  try {
    await mongoose.connect(process.env.DATABASE_URL);

    console.log("MongoDB connected successfully");
  } catch (error) {
    console.error("MongoDB connection failed:", error.message);
    process.exit(1);
  }
}

module.exports = connectDB;
```

Use it in `server.js`:

```js
require("dotenv").config();

const app = require("./app");
const connectDB = require("./config/db");

const PORT = process.env.PORT || 3000;

connectDB();

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

Example `.env`:

```env
PORT=3000
DATABASE_URL=mongodb://localhost:27017/todo-app
```

---

# 5. What Is a Schema?

A schema defines the shape of your data.

For example, a todo should have:

```txt
title: string
description: string
completed: boolean
createdAt: date
updatedAt: date
```

The schema tells Mongoose:

- Which fields exist
- Which fields are required
- What type each field should be
- Default values
- Validation rules

Example:

```js
const todoSchema = new mongoose.Schema({
  title: {
    type: String,
    required: true,
  },
});
```

---

# 6. What Is a Model?

A model is created from a schema.

The model is used to interact with the database.

Example:

```js
const Todo = mongoose.model("Todo", todoSchema);
```

After creating the model, you can do:

```js
Todo.find();
Todo.findById(id);
Todo.create(data);
Todo.findByIdAndUpdate(id, data);
Todo.findByIdAndDelete(id);
```

Mental model:

```txt
Schema = data structure
Model  = database tool based on that structure
```

---

# 7. Creating a User Model

File:

```txt
models/user.model.js
```

```js
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: true,
      trim: true,
    },

    email: {
      type: String,
      required: true,
      unique: true,
      lowercase: true,
      trim: true,
    },

    password: {
      type: String,
      required: true,
      minlength: 6,
      select: false,
    },

    role: {
      type: String,
      enum: ["user", "admin"],
      default: "user",
    },
  },
  {
    timestamps: true,
  },
);

const User = mongoose.model("User", userSchema);

module.exports = User;
```

Important parts:

```js
required: true;
```

Field is required.

```js
unique: true;
```

No two users should have the same email.

```js
lowercase: true;
```

Email is stored in lowercase.

```js
trim: true;
```

Removes extra spaces.

```js
select: false;
```

Password will not be returned by default in queries.

```js
timestamps: true;
```

Adds `createdAt` and `updatedAt`.

---

# 8. Creating a Todo Model

File:

```txt
models/todo.model.js
```

```js
const mongoose = require("mongoose");

const todoSchema = new mongoose.Schema(
  {
    title: {
      type: String,
      required: true,
      trim: true,
    },

    description: {
      type: String,
      required: true,
      trim: true,
    },

    completed: {
      type: Boolean,
      default: false,
    },

    priority: {
      type: String,
      enum: ["low", "medium", "high"],
      default: "medium",
    },

    dueDate: {
      type: Date,
    },
  },
  {
    timestamps: true,
  },
);

const Todo = mongoose.model("Todo", todoSchema);

module.exports = Todo;
```

---

# 9. Common Mongoose Schema Options

## type

Defines field type.

```js
title: {
  type: String,
}
```

Common types:

```txt
String
Number
Boolean
Date
Array
Object
mongoose.Schema.Types.ObjectId
```

---

## required

Makes field mandatory.

```js
email: {
  type: String,
  required: true,
}
```

---

## unique

Makes field unique at database level.

```js
email: {
  type: String,
  unique: true,
}
```

---

## default

Sets default value.

```js
completed: {
  type: Boolean,
  default: false,
}
```

---

## enum

Allows only selected values.

```js
role: {
  type: String,
  enum: ["user", "admin"],
  default: "user",
}
```

---

## trim

Removes extra spaces.

```js
name: {
  type: String,
  trim: true,
}
```

---

## minlength and maxlength

Adds length validation.

```js
password: {
  type: String,
  minlength: 6,
}
```

---

## timestamps

Adds automatic `createdAt` and `updatedAt`.

```js
const schema = new mongoose.Schema(fields, {
  timestamps: true,
});
```

---

# 10. Using Models in Services

A service contains business logic and talks to the database through models.

File:

```txt
services/todo.service.js
```

```js
const Todo = require("../models/todo.model");
const ApiError = require("../utils/ApiError");

async function getAllTodos() {
  return await Todo.find().sort({ createdAt: -1 });
}

async function getTodoById(id) {
  const todo = await Todo.findById(id);

  if (!todo) {
    throw new ApiError(404, "Todo not found");
  }

  return todo;
}

async function createTodo(data) {
  return await Todo.create(data);
}

async function updateTodo(id, data) {
  const todo = await Todo.findByIdAndUpdate(id, data, {
    new: true,
    runValidators: true,
  });

  if (!todo) {
    throw new ApiError(404, "Todo not found");
  }

  return todo;
}

async function deleteTodo(id) {
  const todo = await Todo.findByIdAndDelete(id);

  if (!todo) {
    throw new ApiError(404, "Todo not found");
  }

  return todo;
}

module.exports = {
  getAllTodos,
  getTodoById,
  createTodo,
  updateTodo,
  deleteTodo,
};
```

Important:

```js
new: true
```

Returns updated document.

```js
runValidators: true;
```

Runs schema validation during update.

---

# 11. Full Express + MongoDB + Mongoose Example

## Folder Structure

```txt
backend/
  server.js
  app.js
  .env
  config/
    db.js
  models/
    todo.model.js
  routes/
    todo.routes.js
  controllers/
    todo.controller.js
  services/
    todo.service.js
  middlewares/
    error.middleware.js
  utils/
    ApiError.js
    asyncHandler.js
```

---

## Install

```bash
npm init -y
npm install express mongoose dotenv
```

---

## .env

```env
PORT=3000
DATABASE_URL=mongodb://localhost:27017/todo-app
```

---

## server.js

```js
require("dotenv").config();

const app = require("./app");
const connectDB = require("./config/db");

const PORT = process.env.PORT || 3000;

connectDB();

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

---

## app.js

```js
const express = require("express");
const todoRoutes = require("./routes/todo.routes");
const errorMiddleware = require("./middlewares/error.middleware");

const app = express();

app.use(express.json());

app.get("/", (req, res) => {
  res.json({
    success: true,
    message: "API is running",
  });
});

app.use("/todos", todoRoutes);

app.use(errorMiddleware);

module.exports = app;
```

---

## config/db.js

```js
const mongoose = require("mongoose");

async function connectDB() {
  try {
    await mongoose.connect(process.env.DATABASE_URL);

    console.log("MongoDB connected successfully");
  } catch (error) {
    console.error("MongoDB connection failed:", error.message);
    process.exit(1);
  }
}

module.exports = connectDB;
```

---

## models/todo.model.js

```js
const mongoose = require("mongoose");

const todoSchema = new mongoose.Schema(
  {
    title: {
      type: String,
      required: true,
      trim: true,
    },

    description: {
      type: String,
      required: true,
      trim: true,
    },

    completed: {
      type: Boolean,
      default: false,
    },
  },
  {
    timestamps: true,
  },
);

const Todo = mongoose.model("Todo", todoSchema);

module.exports = Todo;
```

---

## services/todo.service.js

```js
const Todo = require("../models/todo.model");
const ApiError = require("../utils/ApiError");

async function getAllTodos() {
  return await Todo.find().sort({ createdAt: -1 });
}

async function getTodoById(id) {
  const todo = await Todo.findById(id);

  if (!todo) {
    throw new ApiError(404, "Todo not found");
  }

  return todo;
}

async function createTodo(data) {
  return await Todo.create(data);
}

async function updateTodo(id, data) {
  const todo = await Todo.findByIdAndUpdate(id, data, {
    new: true,
    runValidators: true,
  });

  if (!todo) {
    throw new ApiError(404, "Todo not found");
  }

  return todo;
}

async function deleteTodo(id) {
  const todo = await Todo.findByIdAndDelete(id);

  if (!todo) {
    throw new ApiError(404, "Todo not found");
  }

  return todo;
}

module.exports = {
  getAllTodos,
  getTodoById,
  createTodo,
  updateTodo,
  deleteTodo,
};
```

---

## controllers/todo.controller.js

```js
const todoService = require("../services/todo.service");
const asyncHandler = require("../utils/asyncHandler");

const getTodos = asyncHandler(async (req, res) => {
  const todos = await todoService.getAllTodos();

  res.status(200).json({
    success: true,
    data: todos,
  });
});

const getTodoById = asyncHandler(async (req, res) => {
  const todo = await todoService.getTodoById(req.params.id);

  res.status(200).json({
    success: true,
    data: todo,
  });
});

const createTodo = asyncHandler(async (req, res) => {
  const todo = await todoService.createTodo(req.body);

  res.status(201).json({
    success: true,
    data: todo,
  });
});

const updateTodo = asyncHandler(async (req, res) => {
  const todo = await todoService.updateTodo(req.params.id, req.body);

  res.status(200).json({
    success: true,
    data: todo,
  });
});

const deleteTodo = asyncHandler(async (req, res) => {
  const todo = await todoService.deleteTodo(req.params.id);

  res.status(200).json({
    success: true,
    data: todo,
  });
});

module.exports = {
  getTodos,
  getTodoById,
  createTodo,
  updateTodo,
  deleteTodo,
};
```

---

## routes/todo.routes.js

```js
const express = require("express");

const {
  getTodos,
  getTodoById,
  createTodo,
  updateTodo,
  deleteTodo,
} = require("../controllers/todo.controller");

const router = express.Router();

router.get("/", getTodos);
router.get("/:id", getTodoById);
router.post("/", createTodo);
router.put("/:id", updateTodo);
router.delete("/:id", deleteTodo);

module.exports = router;
```

---

## utils/ApiError.js

```js
class ApiError extends Error {
  constructor(statusCode, message) {
    super(message);

    this.statusCode = statusCode;
  }
}

module.exports = ApiError;
```

---

## utils/asyncHandler.js

```js
function asyncHandler(fn) {
  return function (req, res, next) {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}

module.exports = asyncHandler;
```

---

## middlewares/error.middleware.js

```js
function errorMiddleware(err, req, res, next) {
  const statusCode = err.statusCode || 500;

  res.status(statusCode).json({
    success: false,
    message: err.message || "Internal Server Error",
  });
}

module.exports = errorMiddleware;
```

---

# 12. Practice Tasks

## Task 1

Connect Express backend to MongoDB using Mongoose.

---

## Task 2

Create a `User` model with:

```txt
name
email
password
role
timestamps
```

---

## Task 3

Create a `Todo` model with:

```txt
title
description
completed
priority
dueDate
timestamps
```

---

## Task 4

Create services for:

```txt
getAllTodos
getTodoById
createTodo
updateTodo
deleteTodo
```

---

# Quick Revision

MongoDB stores documents.

Mongoose helps define schemas and models.

Schema:

```txt
Shape of data
```

Model:

```txt
Database tool created from schema
```

Example:

```js
const Todo = mongoose.model("Todo", todoSchema);
```

Final flow:

```txt
Express route
  |
  v
Controller
  |
  v
Service
  |
  v
Mongoose Model
  |
  v
MongoDB
```
