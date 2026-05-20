# Backend Notes: PostgreSQL, Prisma Models, and Schemas

These notes explain PostgreSQL connection, Prisma schemas/models, and how they fit into an Express backend.

Goal: understand how to use PostgreSQL with Prisma in a clean backend project.

---

## Table of Contents

1. [What Is PostgreSQL?](#1-what-is-postgresql)
2. [What Is Prisma?](#2-what-is-prisma)
3. [PostgreSQL Connection String](#3-postgresql-connection-string)
4. [Setting Up Prisma](#4-setting-up-prisma)
5. [Prisma Schema File](#5-prisma-schema-file)
6. [What Is a Prisma Model?](#6-what-is-a-prisma-model)
7. [Creating a User Model](#7-creating-a-user-model)
8. [Creating a Todo Model](#8-creating-a-todo-model)
9. [Relations in Prisma](#9-relations-in-prisma)
10. [Running Migrations](#10-running-migrations)
11. [Creating Prisma Client](#11-creating-prisma-client)
12. [Using Prisma in Services](#12-using-prisma-in-services)
13. [Full Express + PostgreSQL + Prisma Example](#13-full-express--postgresql--prisma-example)
14. [Practice Tasks](#14-practice-tasks)

---

# 1. What Is PostgreSQL?

PostgreSQL is a relational SQL database.

It stores data in tables.

Example users table:

```txt
id | name      | email
---|-----------|----------------------
1  | Abhishek  | abhishek@example.com
```

PostgreSQL is good for:

- Structured data
- SaaS apps
- Payments
- Orders
- Transactions
- Relationships between data
- Production-grade applications

It is commonly used in PERN stack:

```txt
PostgreSQL
Express
React
Node.js
```

---

# 2. What Is Prisma?

Prisma is an ORM.

ORM means:

```txt
Object Relational Mapper
```

Prisma helps your Node.js app talk to PostgreSQL using JavaScript instead of writing raw SQL all the time.

With Prisma, instead of writing:

```sql
SELECT * FROM todos;
```

You write:

```js
const todos = await prisma.todo.findMany();
```

Prisma gives:

- Clean database queries
- Schema file
- Migrations
- Type-safe client in TypeScript
- Easier database operations
- Relationship support

---

# 3. PostgreSQL Connection String

Example:

```env
DATABASE_URL=postgresql://postgres:password@localhost:5432/todo_app
```

Breakdown:

```txt
postgresql://username:password@host:port/database
```

For this:

```txt
postgresql://postgres:password@localhost:5432/todo_app
```

- `postgres` = database username
- `password` = database password
- `localhost` = database host
- `5432` = default PostgreSQL port
- `todo_app` = database name

---

# 4. Setting Up Prisma

Install packages:

```bash
npm install @prisma/client
npm install -D prisma
```

Initialize Prisma:

```bash
npx prisma init
```

This creates:

```txt
prisma/
  schema.prisma
.env
```

Your `.env` should contain:

```env
DATABASE_URL=postgresql://postgres:password@localhost:5432/todo_app
```

---

# 5. Prisma Schema File

The Prisma schema file lives here:

```txt
prisma/schema.prisma
```

Basic setup:

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}
```

This tells Prisma:

```txt
Use PostgreSQL
Read database URL from .env
Generate Prisma client for JavaScript/TypeScript
```

---

# 6. What Is a Prisma Model?

A Prisma model defines a database table.

Example:

```prisma
model Todo {
  id          Int      @id @default(autoincrement())
  title       String
  description String
  completed   Boolean  @default(false)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

This creates a table called `Todo`.

Each field becomes a database column.

```txt
id          column
title       column
description column
completed   column
createdAt   column
updatedAt   column
```

---

# 7. Creating a User Model

Inside `prisma/schema.prisma`:

```prisma
model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  password  String
  role      Role     @default(USER)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

enum Role {
  USER
  ADMIN
}
```

Important parts:

```prisma
@id
```

Primary key.

```prisma
@default(autoincrement())
```

ID increases automatically.

```prisma
@unique
```

No duplicate emails.

```prisma
@default(now())
```

Current timestamp when record is created.

```prisma
@updatedAt
```

Automatically updates when record changes.

---

# 8. Creating a Todo Model

```prisma
model Todo {
  id          Int      @id @default(autoincrement())
  title       String
  description String
  completed   Boolean  @default(false)
  priority    Priority @default(MEDIUM)
  dueDate     DateTime?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}

enum Priority {
  LOW
  MEDIUM
  HIGH
}
```

Important:

```prisma
DateTime?
```

The `?` means optional.

So `dueDate` is not required.

---

# 9. Relations in Prisma

PostgreSQL is very good at relationships.

Example:

```txt
One user can have many todos
One todo belongs to one user
```

Prisma schema:

```prisma
model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  password  String
  todos     Todo[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Todo {
  id          Int      @id @default(autoincrement())
  title       String
  description String
  completed   Boolean  @default(false)

  userId      Int
  user        User     @relation(fields: [userId], references: [id])

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

Meaning:

```txt
User has many Todo records
Todo belongs to one User
```

---

# 10. Running Migrations

After editing `schema.prisma`, run:

```bash
npx prisma migrate dev --name init
```

This does two things:

1. Creates migration files
2. Updates your PostgreSQL database tables

Example:

```bash
npx prisma migrate dev --name add_todo_model
```

---

## Generate Prisma Client

Usually migration generates Prisma Client automatically.

But you can manually run:

```bash
npx prisma generate
```

---

## Open Prisma Studio

Prisma Studio gives a browser UI to view/edit database data.

```bash
npx prisma studio
```

---

# 11. Creating Prisma Client

Create:

```txt
config/prisma.js
```

```js
const { PrismaClient } = require("@prisma/client");

const prisma = new PrismaClient();

module.exports = prisma;
```

In bigger projects, you import this wherever database access is needed.

---

# 12. Using Prisma in Services

Services contain database logic.

File:

```txt
services/todo.service.js
```

```js
const prisma = require("../config/prisma");
const ApiError = require("../utils/ApiError");

async function getAllTodos() {
  return await prisma.todo.findMany({
    orderBy: {
      createdAt: "desc",
    },
  });
}

async function getTodoById(id) {
  const todo = await prisma.todo.findUnique({
    where: {
      id,
    },
  });

  if (!todo) {
    throw new ApiError(404, "Todo not found");
  }

  return todo;
}

async function createTodo(data) {
  return await prisma.todo.create({
    data: {
      title: data.title,
      description: data.description,
      completed: data.completed ?? false,
    },
  });
}

async function updateTodo(id, data) {
  const existingTodo = await prisma.todo.findUnique({
    where: {
      id,
    },
  });

  if (!existingTodo) {
    throw new ApiError(404, "Todo not found");
  }

  return await prisma.todo.update({
    where: {
      id,
    },
    data,
  });
}

async function deleteTodo(id) {
  const existingTodo = await prisma.todo.findUnique({
    where: {
      id,
    },
  });

  if (!existingTodo) {
    throw new ApiError(404, "Todo not found");
  }

  return await prisma.todo.delete({
    where: {
      id,
    },
  });
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

If your Prisma model uses:

```prisma
id Int @id @default(autoincrement())
```

Then convert route params to number:

```js
const id = Number(req.params.id);
```

Because `req.params.id` is always a string.

---

# 13. Full Express + PostgreSQL + Prisma Example

## Folder Structure

```txt
backend/
  server.js
  app.js
  .env
  prisma/
    schema.prisma
  config/
    prisma.js
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
npm install express dotenv @prisma/client
npm install -D prisma
```

Initialize Prisma:

```bash
npx prisma init
```

---

## .env

```env
PORT=3000
DATABASE_URL=postgresql://postgres:password@localhost:5432/todo_app
```

---

## prisma/schema.prisma

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model Todo {
  id          Int      @id @default(autoincrement())
  title       String
  description String
  completed   Boolean  @default(false)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

Run migration:

```bash
npx prisma migrate dev --name init
```

---

## server.js

```js
require("dotenv").config();

const app = require("./app");

const PORT = process.env.PORT || 3000;

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

## config/prisma.js

```js
const { PrismaClient } = require("@prisma/client");

const prisma = new PrismaClient();

module.exports = prisma;
```

---

## services/todo.service.js

```js
const prisma = require("../config/prisma");
const ApiError = require("../utils/ApiError");

async function getAllTodos() {
  return await prisma.todo.findMany({
    orderBy: {
      createdAt: "desc",
    },
  });
}

async function getTodoById(id) {
  const todo = await prisma.todo.findUnique({
    where: {
      id,
    },
  });

  if (!todo) {
    throw new ApiError(404, "Todo not found");
  }

  return todo;
}

async function createTodo(data) {
  return await prisma.todo.create({
    data: {
      title: data.title,
      description: data.description,
    },
  });
}

async function updateTodo(id, data) {
  const existingTodo = await prisma.todo.findUnique({
    where: {
      id,
    },
  });

  if (!existingTodo) {
    throw new ApiError(404, "Todo not found");
  }

  return await prisma.todo.update({
    where: {
      id,
    },
    data,
  });
}

async function deleteTodo(id) {
  const existingTodo = await prisma.todo.findUnique({
    where: {
      id,
    },
  });

  if (!existingTodo) {
    throw new ApiError(404, "Todo not found");
  }

  return await prisma.todo.delete({
    where: {
      id,
    },
  });
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
  const id = Number(req.params.id);

  const todo = await todoService.getTodoById(id);

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
  const id = Number(req.params.id);

  const todo = await todoService.updateTodo(id, req.body);

  res.status(200).json({
    success: true,
    data: todo,
  });
});

const deleteTodo = asyncHandler(async (req, res) => {
  const id = Number(req.params.id);

  const todo = await todoService.deleteTodo(id);

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

# 14. Practice Tasks

## Task 1

Create a PostgreSQL database called:

```txt
todo_app
```

---

## Task 2

Initialize Prisma:

```bash
npx prisma init
```

---

## Task 3

Create a `Todo` model in `schema.prisma`.

---

## Task 4

Run migration:

```bash
npx prisma migrate dev --name init
```

---

## Task 5

Create `config/prisma.js`.

---

## Task 6

Create todo service functions:

```txt
getAllTodos
getTodoById
createTodo
updateTodo
deleteTodo
```

---

# Quick Revision

PostgreSQL stores data in tables.

Prisma lets your backend interact with PostgreSQL using JavaScript.

Prisma model:

```prisma
model Todo {
  id    Int    @id @default(autoincrement())
  title String
}
```

Migration creates actual database tables:

```bash
npx prisma migrate dev --name init
```

Prisma client is used in services:

```js
const todos = await prisma.todo.findMany();
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
Prisma Client
  |
  v
PostgreSQL
```
