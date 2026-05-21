# Backend Notes: Input Validation with Zod and Joi

These notes explain input validation for backend development.

Goal: understand why we validate request data, where validation fits in an Express backend, and how to use Zod or Joi to validate incoming data before it reaches controllers/services/database.

---

## Table of Contents

1. [What Is Input Validation?](#1-what-is-input-validation)
2. [Why Input Validation Matters](#2-why-input-validation-matters)
3. [Where Does Input Validation Happen?](#3-where-does-input-validation-happen)
4. [What Data Should You Validate?](#4-what-data-should-you-validate)
5. [Manual Validation](#5-manual-validation)
6. [Why Use a Validation Library?](#6-why-use-a-validation-library)
7. [Zod Validation](#7-zod-validation)
8. [Zod with Express Middleware](#8-zod-with-express-middleware)
9. [Joi Validation](#9-joi-validation)
10. [Joi with Express Middleware](#10-joi-with-express-middleware)
11. [Zod vs Joi](#11-zod-vs-joi)
12. [Validation in Real Backend Architecture](#12-validation-in-real-backend-architecture)
13. [Full Express + Zod Todo Example](#13-full-express--zod-todo-example)
14. [Common Validation Patterns](#14-common-validation-patterns)
15. [Practice Tasks](#15-practice-tasks)
16. [Quick Revision](#16-quick-revision)

---

# 1. What Is Input Validation?

Input validation means checking the data sent by the client before your backend uses it.

Example request:

```http
POST /todos
Content-Type: application/json
```

Body:

```json
{
  "title": "Learn validation",
  "description": "Understand Zod and Joi"
}
```

Before saving this data to the database, the backend should check:

```txt
Is title present?
Is title a string?
Is description present?
Is description a string?
Is completed a boolean if provided?
```

If the data is valid, continue.

If the data is invalid, reject the request.

---

# 2. Why Input Validation Matters

Frontend validation is not enough.

A user can bypass frontend validation by using:

- Postman
- Thunder Client
- curl
- Browser devtools
- Custom scripts
- Another frontend

So backend validation is necessary.

Without validation, a user could send:

```json
{
  "title": "",
  "description": 123,
  "completed": "yes"
}
```

Problems:

```txt
title is empty
description should be string
completed should be boolean
```

Backend should reject it:

```json
{
  "success": false,
  "message": "Invalid input",
  "errors": [
    {
      "field": "title",
      "message": "Title is required"
    },
    {
      "field": "description",
      "message": "Description must be a string"
    }
  ]
}
```

Bad data can cause:

- Database errors
- Broken UI
- Security bugs
- App crashes
- Bad data in database
- Unexpected backend behavior

---

# 3. Where Does Input Validation Happen?

In a clean Express backend, validation usually happens before the controller.

Flow:

```txt
Client request
  |
  v
Route
  |
  v
Validation middleware
  |
  v
Controller
  |
  v
Service
  |
  v
Database
```

Example:

```js
router.post("/", validate(createTodoSchema), createTodo);
```

This means:

```txt
POST /todos
  |
  v
validate request body
  |
  v
if valid, run createTodo controller
```

If validation fails, the controller does not run.

---

# 4. What Data Should You Validate?

You should validate all user-controlled input.

## 4.1 Request Body

Used in:

```http
POST /todos
PUT /todos/:id
PATCH /todos/:id
POST /auth/register
POST /auth/login
```

Access:

```js
req.body
```

---

## 4.2 Route Params

Used in:

```http
GET /todos/:id
DELETE /todos/:id
```

Access:

```js
req.params.id
```

You should check:

```txt
Is id present?
Is id valid?
Is id a number or valid MongoDB ObjectId?
```

---

## 4.3 Query Parameters

Used in:

```http
GET /products?page=1&limit=10&search=laptop
```

Access:

```js
req.query.page
req.query.limit
req.query.search
```

You should check:

```txt
Is page a number?
Is limit a number?
Is search a string?
```

---

## 4.4 Headers

Sometimes you validate headers.

Example:

```js
req.headers.authorization
req.headers["x-api-key"]
```

You might check:

```txt
Is Authorization header present?
Does API key exist?
Is token format correct?
```

---

# 5. Manual Validation

You can validate manually without libraries.

Example:

```js
app.post("/todos", (req, res) => {
  const { title, description } = req.body;

  if (!title || typeof title !== "string") {
    return res.status(400).json({
      success: false,
      message: "Title is required and must be a string",
    });
  }

  if (!description || typeof description !== "string") {
    return res.status(400).json({
      success: false,
      message: "Description is required and must be a string",
    });
  }

  res.status(201).json({
    success: true,
    message: "Todo created",
  });
});
```

This works for small examples.

But for bigger apps, manual validation becomes messy because every route needs many `if` checks.

---

# 6. Why Use a Validation Library?

Validation libraries help you define rules cleanly.

Popular options:

```txt
Zod
Joi
Yup
Valibot
```

Benefits:

- Cleaner validation code
- Reusable schemas
- Better error messages
- Less repetitive code
- Easier to validate complex objects
- Works well as middleware
- Zod works especially well with TypeScript

---

# 7. Zod Validation

Zod is a validation library.

Install:

```bash
npm install zod
```

Import:

```js
const { z } = require("zod");
```

For ES modules:

```js
import { z } from "zod";
```

## Basic Zod Schema

```js
const { z } = require("zod");

const createTodoSchema = z.object({
  title: z.string().min(1, "Title is required"),
  description: z.string().min(1, "Description is required"),
  completed: z.boolean().optional(),
});
```

This says:

```txt
title must be string and at least 1 character
description must be string and at least 1 character
completed is optional, but if present, must be boolean
```

## Zod `parse`

```js
const data = createTodoSchema.parse(req.body);
```

If valid, it returns validated data.

If invalid, it throws an error.

## Zod `safeParse`

```js
const result = createTodoSchema.safeParse(req.body);
```

If valid:

```js
{
  success: true,
  data: {
    title: "Learn Zod",
    description: "Validation"
  }
}
```

If invalid:

```js
{
  success: false,
  error: ...
}
```

For Express middleware, `safeParse` is beginner-friendly because it does not throw automatically.

## Example User Schema

```js
const { z } = require("zod");

const registerSchema = z.object({
  name: z.string().min(1, "Name is required"),
  email: z.string().email("Invalid email address"),
  password: z.string().min(6, "Password must be at least 6 characters"),
});
```

---

# 8. Zod with Express Middleware

Instead of writing validation inside every controller, create reusable middleware.

File:

```txt
middlewares/validate.middleware.js
```

```js
function validate(schema) {
  return function (req, res, next) {
    const result = schema.safeParse(req.body);

    if (!result.success) {
      return res.status(400).json({
        success: false,
        message: "Invalid input",
        errors: result.error.issues.map((issue) => ({
          field: issue.path.join("."),
          message: issue.message,
        })),
      });
    }

    req.body = result.data;

    next();
  };
}

module.exports = validate;
```

Why this line?

```js
req.body = result.data;
```

Because Zod returns validated data. The controller receives the cleaned version of the body.

## Todo Validation Schema

File:

```txt
validations/todo.validation.js
```

```js
const { z } = require("zod");

const createTodoSchema = z.object({
  title: z.string().min(1, "Title is required"),
  description: z.string().min(1, "Description is required"),
  completed: z.boolean().optional(),
});

const updateTodoSchema = z.object({
  title: z.string().min(1, "Title cannot be empty").optional(),
  description: z.string().min(1, "Description cannot be empty").optional(),
  completed: z.boolean().optional(),
});

module.exports = {
  createTodoSchema,
  updateTodoSchema,
};
```

## Use Validation Middleware in Routes

```js
const express = require("express");
const validate = require("../middlewares/validate.middleware");

const {
  createTodoSchema,
  updateTodoSchema,
} = require("../validations/todo.validation");

const {
  createTodo,
  updateTodo,
} = require("../controllers/todo.controller");

const router = express.Router();

router.post("/", validate(createTodoSchema), createTodo);
router.put("/:id", validate(updateTodoSchema), updateTodo);

module.exports = router;
```

---

# 9. Joi Validation

Joi is another popular validation library for Node.js.

Install:

```bash
npm install joi
```

Import:

```js
const Joi = require("joi");
```

## Basic Joi Schema

```js
const Joi = require("joi");

const createTodoSchema = Joi.object({
  title: Joi.string().min(1).required(),
  description: Joi.string().min(1).required(),
  completed: Joi.boolean().optional(),
});
```

This says:

```txt
title must be string and required
description must be string and required
completed is optional boolean
```

## Joi Validate

```js
const { error, value } = createTodoSchema.validate(req.body);
```

If invalid, `error` contains validation details.

If valid, `value` contains validated data.

---

# 10. Joi with Express Middleware

File:

```txt
middlewares/validateJoi.middleware.js
```

```js
function validateJoi(schema) {
  return function (req, res, next) {
    const { error, value } = schema.validate(req.body, {
      abortEarly: false,
      stripUnknown: true,
    });

    if (error) {
      return res.status(400).json({
        success: false,
        message: "Invalid input",
        errors: error.details.map((detail) => ({
          field: detail.path.join("."),
          message: detail.message,
        })),
      });
    }

    req.body = value;

    next();
  };
}

module.exports = validateJoi;
```

## What is `abortEarly: false`?

```js
abortEarly: false
```

means:

```txt
show all validation errors, not just the first one
```

## What is `stripUnknown: true`?

```js
stripUnknown: true
```

means:

```txt
remove fields that are not defined in the schema
```

---

# 11. Zod vs Joi

Both are good validation libraries.

| Feature | Zod | Joi |
|---|---|---|
| TypeScript support | Excellent | Good |
| Works with JavaScript | Yes | Yes |
| Syntax | Modern | Mature |
| Popular in modern full-stack apps | Very popular | Popular |
| Error handling | Clean | Detailed |
| Schema inference in TypeScript | Excellent | Limited |
| Good for Express | Yes | Yes |

For your journey, prefer:

```txt
Zod
```

Why?

Because you are learning modern full-stack development, and Zod is commonly used with:

- Express
- Next.js
- tRPC
- Hono
- React Hook Form
- TypeScript apps

---

# 12. Validation in Real Backend Architecture

A clean backend usually looks like this:

```txt
backend/
  server.js
  app.js
  routes/
    todo.routes.js
  controllers/
    todo.controller.js
  services/
    todo.service.js
  validations/
    todo.validation.js
  middlewares/
    validate.middleware.js
    error.middleware.js
  utils/
    asyncHandler.js
    ApiError.js
```

Request flow:

```txt
Client
  |
  v
Route
  |
  v
Validation middleware
  |
  v
Controller
  |
  v
Service
  |
  v
Database
```

Error flow:

```txt
Invalid input
  |
  v
Validation middleware returns 400
```

Valid input flow:

```txt
Valid input
  |
  v
Controller runs
```

---

# 13. Full Express + Zod Todo Example

## Install

```bash
npm init -y
npm install express zod
```

## Folder Structure

```txt
backend/
  server.js
  app.js
  routes/
    todo.routes.js
  controllers/
    todo.controller.js
  services/
    todo.service.js
  validations/
    todo.validation.js
  middlewares/
    validate.middleware.js
    error.middleware.js
  utils/
    asyncHandler.js
    ApiError.js
```

## server.js

```js
const app = require("./app");

const PORT = 3000;

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

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

## validations/todo.validation.js

```js
const { z } = require("zod");

const createTodoSchema = z.object({
  title: z.string().min(1, "Title is required"),
  description: z.string().min(1, "Description is required"),
  completed: z.boolean().optional(),
});

const updateTodoSchema = z.object({
  title: z.string().min(1, "Title cannot be empty").optional(),
  description: z.string().min(1, "Description cannot be empty").optional(),
  completed: z.boolean().optional(),
});

module.exports = {
  createTodoSchema,
  updateTodoSchema,
};
```

## middlewares/validate.middleware.js

```js
function validate(schema) {
  return function (req, res, next) {
    const result = schema.safeParse(req.body);

    if (!result.success) {
      return res.status(400).json({
        success: false,
        message: "Invalid input",
        errors: result.error.issues.map((issue) => ({
          field: issue.path.join("."),
          message: issue.message,
        })),
      });
    }

    req.body = result.data;

    next();
  };
}

module.exports = validate;
```

## routes/todo.routes.js

```js
const express = require("express");
const validate = require("../middlewares/validate.middleware");

const {
  createTodoSchema,
  updateTodoSchema,
} = require("../validations/todo.validation");

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
router.post("/", validate(createTodoSchema), createTodo);
router.put("/:id", validate(updateTodoSchema), updateTodo);
router.delete("/:id", deleteTodo);

module.exports = router;
```

---

# 14. Common Validation Patterns

## 14.1 Register User Schema

```js
const { z } = require("zod");

const registerSchema = z.object({
  name: z.string().min(1, "Name is required"),
  email: z.string().email("Invalid email address"),
  password: z.string().min(6, "Password must be at least 6 characters"),
});
```

## 14.2 Login Schema

```js
const loginSchema = z.object({
  email: z.string().email("Invalid email address"),
  password: z.string().min(1, "Password is required"),
});
```

## 14.3 Product Schema

```js
const productSchema = z.object({
  name: z.string().min(1, "Product name is required"),
  price: z.number().positive("Price must be positive"),
  stock: z.number().int().min(0, "Stock cannot be negative"),
  description: z.string().optional(),
});
```

## 14.4 Pagination Query Schema

For query params, values usually come as strings.

Example URL:

```http
GET /products?page=1&limit=10
```

Zod schema:

```js
const paginationSchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  limit: z.coerce.number().int().positive().max(100).default(10),
});
```

Why `z.coerce.number()`?

Because query params come as strings:

```js
req.query.page // "1"
```

`z.coerce.number()` converts `"1"` to `1`.

## 14.5 MongoDB ObjectId Validation

MongoDB IDs look like this:

```txt
665f1ae2c95b2b4c89e9c123
```

Simple Zod validation:

```js
const objectIdSchema = z.object({
  id: z.string().regex(/^[0-9a-fA-F]{24}$/, "Invalid MongoDB ObjectId"),
});
```

## 14.6 PostgreSQL Number ID Validation

PostgreSQL IDs are often numbers.

```js
const paramsSchema = z.object({
  id: z.coerce.number().int().positive("Invalid ID"),
});
```

---

# 15. Practice Tasks

## Task 1

Create a Zod schema for todo creation:

```txt
title required string
description required string
completed optional boolean
```

## Task 2

Create a Zod schema for user registration:

```txt
name required
email valid email
password minimum 6 characters
```

## Task 3

Create a reusable validation middleware:

```js
validate(schema)
```

Use it on:

```http
POST /todos
```

## Task 4

Create a Joi version of the same todo validation.

## Task 5

Validate query params:

```http
GET /products?page=1&limit=10
```

Make sure:

```txt
page is positive number
limit is positive number
limit is max 100
```

## Task 6

Validate route params:

```http
GET /todos/:id
```

For PostgreSQL:

```txt
id should be a number
```

For MongoDB:

```txt
id should be a valid ObjectId
```

---

# 16. Quick Revision

Input validation means checking user input before using it.

Validate:

```txt
req.body
req.params
req.query
req.headers
```

Manual validation works, but gets messy.

Zod and Joi make validation cleaner.

Recommended for modern full-stack:

```txt
Zod
```

Clean request flow:

```txt
Client
  |
  v
Route
  |
  v
Validation middleware
  |
  v
Controller
  |
  v
Service
  |
  v
Database
```

If validation fails:

```txt
Return 400 Bad Request
```

If validation passes:

```txt
Controller runs with clean data
```

Final mental model:

```txt
Validation protects your backend from bad input.
```
