# Backend Architecture Notes: REST API Design, Express Router, Controllers, Services, and Error Handling

These notes are for your full-stack/backend journey.  
Goal: understand how real backend projects are structured instead of writing everything inside one `server.js` file.

---

## Table of Contents

1. [What Is REST API Design?](#1-what-is-rest-api-design)
2. [Why REST API Design Matters](#2-why-rest-api-design-matters)
3. [Resources and Endpoints](#3-resources-and-endpoints)
4. [HTTP Methods in REST](#4-http-methods-in-rest)
5. [Good vs Bad API Routes](#5-good-vs-bad-api-routes)
6. [Status Codes in REST APIs](#6-status-codes-in-rest-apis)
7. [Request Body, Params, and Query](#7-request-body-params-and-query)
8. [REST API Response Format](#8-rest-api-response-format)
9. [Express Router](#9-express-router)
10. [Why Use Express Router?](#10-why-use-express-router)
11. [Controllers](#11-controllers)
12. [Services](#12-services)
13. [Controller vs Service](#13-controller-vs-service)
14. [Folder Structure](#14-folder-structure)
15. [Error Handling Middleware](#15-error-handling-middleware)
16. [Custom Error Class](#16-custom-error-class)
17. [Async Error Handling](#17-async-error-handling)
18. [Full Todo API Example](#18-full-todo-api-example)
19. [Practice Tasks](#19-practice-tasks)

---

# 1. What Is REST API Design?

REST stands for:

```txt
Representational State Transfer
```

For beginners, think of REST as a clean way to design backend APIs using:

- Resources
- URLs
- HTTP methods
- Status codes
- JSON responses

A REST API helps the frontend communicate with the backend in a predictable way.

Example:

```http
GET /todos
POST /todos
GET /todos/1
PUT /todos/1
DELETE /todos/1
```

These routes are easy to understand because they follow a pattern.

---

# 2. Why REST API Design Matters

Without REST design, your routes can become messy.

Bad example:

```http
GET /getAllTodos
POST /createNewTodo
POST /deleteTodo
POST /updateTodo
GET /todoDeleteById
```

This works technically, but it is not clean.

Good REST-style example:

```http
GET /todos
POST /todos
GET /todos/:id
PUT /todos/:id
DELETE /todos/:id
```

This is cleaner because the HTTP method explains the action.

---

# 3. Resources and Endpoints

In REST, a resource is usually a noun.

Examples:

```txt
users
todos
products
orders
posts
comments
messages
```

A resource represents data in your application.

For example, in a todo app:

```txt
Resource = todos
```

So your endpoints should be based on `todos`.

Examples:

```http
GET /todos
POST /todos
GET /todos/:id
PUT /todos/:id
DELETE /todos/:id
```

---

## Endpoint Meaning

```http
GET /todos
```

Get all todos.

```http
POST /todos
```

Create a new todo.

```http
GET /todos/:id
```

Get one todo by ID.

```http
PUT /todos/:id
```

Update one todo by ID.

```http
DELETE /todos/:id
```

Delete one todo by ID.

---

# 4. HTTP Methods in REST

## GET

Used to read data.

```http
GET /todos
```

Should not modify data.

---

## POST

Used to create data.

```http
POST /todos
```

Example body:

```json
{
  "title": "Learn REST API",
  "completed": false
}
```

---

## PUT

Used to update or replace data.

```http
PUT /todos/1
```

Example body:

```json
{
  "title": "Updated Todo",
  "completed": true
}
```

---

## PATCH

Used for partial update.

```http
PATCH /todos/1
```

Example body:

```json
{
  "completed": true
}
```

Difference between PUT and PATCH:

```txt
PUT   = usually replace/update larger object
PATCH = update only selected fields
```

---

## DELETE

Used to delete data.

```http
DELETE /todos/1
```

---

# 5. Good vs Bad API Routes

## Bad

```http
GET /getUsers
POST /createUser
POST /deleteUser
POST /updateUser
```

Why bad?

Because the action is written inside the route name.

---

## Good

```http
GET /users
POST /users
PUT /users/:id
DELETE /users/:id
```

Why good?

Because:

- URL uses nouns
- HTTP method tells the action
- API becomes predictable

---

## More Examples

### Users

```http
GET /users
POST /users
GET /users/:id
PUT /users/:id
DELETE /users/:id
```

### Products

```http
GET /products
POST /products
GET /products/:id
PUT /products/:id
DELETE /products/:id
```

### Blog Posts

```http
GET /posts
POST /posts
GET /posts/:id
PUT /posts/:id
DELETE /posts/:id
```

### Nested Resources

For comments inside posts:

```http
GET /posts/:postId/comments
POST /posts/:postId/comments
DELETE /posts/:postId/comments/:commentId
```

Meaning:

```txt
Get comments of a specific post
Create comment inside a specific post
Delete a specific comment of a specific post
```

---

# 6. Status Codes in REST APIs

Status codes tell the client what happened.

## Common Success Codes

### 200 OK

Used when request succeeds.

Example:

```http
GET /todos
```

Response:

```http
200 OK
```

---

### 201 Created

Used when a new resource is created.

Example:

```http
POST /todos
```

Response:

```http
201 Created
```

---

### 204 No Content

Used when request succeeds but there is no response body.

Example:

```http
DELETE /todos/1
```

Response:

```http
204 No Content
```

You can also return `200` with a message for beginner projects.

---

## Common Error Codes

### 400 Bad Request

Client sent invalid data.

Example:

```json
{
  "title": ""
}
```

---

### 401 Unauthorized

User is not logged in or token is missing.

Example:

```json
{
  "message": "Authentication required"
}
```

---

### 403 Forbidden

User is logged in but not allowed to access this resource.

Example:

```json
{
  "message": "You do not have permission"
}
```

---

### 404 Not Found

Resource does not exist.

Example:

```http
GET /todos/999
```

Response:

```json
{
  "message": "Todo not found"
}
```

---

### 500 Internal Server Error

Something went wrong on the server.

Example:

```json
{
  "message": "Something went wrong"
}
```

---

# 7. Request Body, Params, and Query

In Express, data can come from different places.

---

## 7.1 Request Body

Used when client sends data.

Example:

```http
POST /todos
```

Body:

```json
{
  "title": "Learn Express",
  "description": "Understand backend structure"
}
```

Access in Express:

```js
const { title, description } = req.body;
```

For this to work, you need:

```js
app.use(express.json());
```

---

## 7.2 Route Params

Used for required values in the URL.

Example:

```http
GET /todos/5
```

Route:

```js
app.get("/todos/:id", controller);
```

Access:

```js
const id = req.params.id;
```

Use params when the value identifies a specific resource.

Examples:

```http
GET /users/:id
GET /products/:id
DELETE /posts/:id
```

---

## 7.3 Query Parameters

Used for optional filters, search, sorting, or pagination.

Example:

```http
GET /products?search=laptop&page=1&limit=10
```

Access:

```js
const search = req.query.search;
const page = req.query.page;
const limit = req.query.limit;
```

Use query for optional behavior.

Examples:

```http
GET /todos?completed=true
GET /products?sort=price
GET /users?role=admin
GET /posts?page=2&limit=10
```

---

# 8. REST API Response Format

A good API response should be consistent.

## Success Response

Example:

```json
{
  "success": true,
  "message": "Todos fetched successfully",
  "data": [
    {
      "id": 1,
      "title": "Learn REST"
    }
  ]
}
```

## Error Response

Example:

```json
{
  "success": false,
  "message": "Todo not found"
}
```

Consistency helps the frontend because the frontend knows what shape to expect.

---

# 9. Express Router

Express Router helps split routes into separate files.

Without Express Router, everything goes into `server.js`.

Bad for large projects:

```js
app.get("/todos", ...);
app.post("/todos", ...);
app.put("/todos/:id", ...);
app.delete("/todos/:id", ...);

app.get("/users", ...);
app.post("/users", ...);
app.put("/users/:id", ...);
app.delete("/users/:id", ...);
```

As your project grows, `server.js` becomes messy.

Express Router solves this.

---

## Basic Express Router Example

Create file:

```txt
routes/todo.routes.js
```

```js
const express = require("express");

const router = express.Router();

router.get("/", (req, res) => {
  res.json({
    message: "Get all todos",
  });
});

router.post("/", (req, res) => {
  res.json({
    message: "Create todo",
  });
});

module.exports = router;
```

Use it in `server.js`:

```js
const express = require("express");
const todoRoutes = require("./routes/todo.routes");

const app = express();

app.use(express.json());

app.use("/todos", todoRoutes);

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

Now:

```js
router.get("/");
```

becomes:

```http
GET /todos
```

Because of:

```js
app.use("/todos", todoRoutes);
```

---

# 10. Why Use Express Router?

Express Router helps you organize code by feature.

Example:

```txt
routes/
  todo.routes.js
  user.routes.js
  auth.routes.js
  product.routes.js
```

Then in `server.js`:

```js
app.use("/todos", todoRoutes);
app.use("/users", userRoutes);
app.use("/auth", authRoutes);
app.use("/products", productRoutes);
```

Benefits:

- Cleaner `server.js`
- Routes are grouped by feature
- Easier to maintain
- Easier to scale
- Easier to debug
- Looks more professional

---

# 11. Controllers

A controller is responsible for handling the request and response.

It receives:

```js
req;
```

and sends:

```js
res;
```

Example controller:

```js
function getTodos(req, res) {
  res.json({
    message: "Get all todos",
  });
}
```

Controllers usually:

- Read data from `req.params`, `req.query`, or `req.body`
- Call service functions
- Send response using `res`
- Pass errors to error middleware

---

## Controller Example

File:

```txt
controllers/todo.controller.js
```

```js
function getTodos(req, res) {
  res.json({
    success: true,
    message: "Todos fetched successfully",
    data: [],
  });
}

function createTodo(req, res) {
  const { title, description } = req.body;

  res.status(201).json({
    success: true,
    message: "Todo created successfully",
    data: {
      title,
      description,
    },
  });
}

module.exports = {
  getTodos,
  createTodo,
};
```

Route file:

```js
const express = require("express");

const { getTodos, createTodo } = require("../controllers/todo.controller");

const router = express.Router();

router.get("/", getTodos);
router.post("/", createTodo);

module.exports = router;
```

This makes routes cleaner.

---

# 12. Services

A service contains business logic.

Business logic means actual work your app needs to do.

Examples:

- Create todo
- Find todo by ID
- Update todo
- Delete todo
- Check if user exists
- Hash password
- Generate token
- Calculate price
- Send email
- Save data to database

Services usually do not know about `req` and `res`.

That means services should not directly send HTTP responses.

---

## Service Example

File:

```txt
services/todo.service.js
```

```js
let todos = [];
let idCounter = 1;

function getAllTodos() {
  return todos;
}

function getTodoById(id) {
  return todos.find((todo) => todo.id === id);
}

function createTodo(data) {
  const newTodo = {
    id: idCounter++,
    title: data.title,
    description: data.description,
    completed: false,
  };

  todos.push(newTodo);

  return newTodo;
}

module.exports = {
  getAllTodos,
  getTodoById,
  createTodo,
};
```

Controller uses service:

```js
const todoService = require("../services/todo.service");

function getTodos(req, res) {
  const todos = todoService.getAllTodos();

  res.json({
    success: true,
    data: todos,
  });
}
```

---

# 13. Controller vs Service

This is very important.

## Controller

Controller handles HTTP layer.

It knows about:

```js
req;
res;
next;
```

Controller is responsible for:

- Reading request params/body/query
- Calling service
- Sending response
- Sending error to error middleware

---

## Service

Service handles business logic.

It should not know about:

```js
req;
res;
```

Service is responsible for:

- Working with data
- Applying rules
- Talking to database
- Returning result
- Throwing errors when needed

---

## Simple Mental Model

```txt
Route
  |
  v
Controller
  |
  v
Service
  |
  v
Database / Array / External API
```

Then result comes back:

```txt
Service
  |
  v
Controller
  |
  v
Response
```

---

## Example Flow

Request:

```http
POST /todos
```

Flow:

```txt
Client sends request
      |
      v
todo.routes.js
      |
      v
createTodoController
      |
      v
createTodoService
      |
      v
todos array / database
      |
      v
controller sends JSON response
```

---

# 14. Folder Structure

For a beginner Express project:

```txt
backend/
  server.js
  package.json
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
```

For a larger project:

```txt
backend/
  src/
    app.js
    server.js
    routes/
      todo.routes.js
      user.routes.js
      auth.routes.js
    controllers/
      todo.controller.js
      user.controller.js
      auth.controller.js
    services/
      todo.service.js
      user.service.js
      auth.service.js
    middlewares/
      auth.middleware.js
      error.middleware.js
      logger.middleware.js
    utils/
      ApiError.js
      asyncHandler.js
    config/
      db.js
      env.js
```

---

## app.js vs server.js

In cleaner projects, people separate app setup and server startup.

### app.js

Contains Express app setup:

```js
const express = require("express");
const todoRoutes = require("./routes/todo.routes");
const errorMiddleware = require("./middlewares/error.middleware");

const app = express();

app.use(express.json());

app.use("/todos", todoRoutes);

app.use(errorMiddleware);

module.exports = app;
```

### server.js

Starts the server:

```js
const app = require("./app");

const PORT = 3000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

Why separate?

Because later testing becomes easier.

---

# 15. Error Handling Middleware

Normal middleware has this structure:

```js
function middleware(req, res, next) {}
```

Error handling middleware has four parameters:

```js
function errorMiddleware(err, req, res, next) {}
```

Express knows it is an error middleware because it has four arguments.

---

## Basic Error Middleware

```js
function errorMiddleware(err, req, res, next) {
  console.error(err);

  res.status(500).json({
    success: false,
    message: "Something went wrong",
  });
}

module.exports = errorMiddleware;
```

Use it at the end of all routes:

```js
app.use(errorMiddleware);
```

Important:

> Error middleware should be registered after your routes.

Correct:

```js
app.use("/todos", todoRoutes);

app.use(errorMiddleware);
```

Wrong:

```js
app.use(errorMiddleware);

app.use("/todos", todoRoutes);
```

---

# 16. Custom Error Class

A custom error class helps create errors with status codes.

File:

```txt
utils/ApiError.js
```

```js
class ApiError extends Error {
  constructor(statusCode, message) {
    super(message);

    this.statusCode = statusCode;
    this.success = false;
  }
}

module.exports = ApiError;
```

Use it:

```js
const ApiError = require("../utils/ApiError");

throw new ApiError(404, "Todo not found");
```

Error middleware:

```js
function errorMiddleware(err, req, res, next) {
  const statusCode = err.statusCode || 500;
  const message = err.message || "Something went wrong";

  res.status(statusCode).json({
    success: false,
    message,
  });
}

module.exports = errorMiddleware;
```

Now your services or controllers can throw clean errors.

---

# 17. Async Error Handling

In Express, async route errors need to be passed to `next`.

Bad:

```js
app.get("/todos", async (req, res) => {
  throw new Error("Database failed");
});
```

This may not be handled properly in older Express setups.

Better:

```js
app.get("/todos", async (req, res, next) => {
  try {
    throw new Error("Database failed");
  } catch (error) {
    next(error);
  }
});
```

But writing `try/catch` in every controller is repetitive.

So we create `asyncHandler`.

---

## asyncHandler Utility

File:

```txt
utils/asyncHandler.js
```

```js
function asyncHandler(fn) {
  return function (req, res, next) {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}

module.exports = asyncHandler;
```

Use it:

```js
const asyncHandler = require("../utils/asyncHandler");

const getTodos = asyncHandler(async (req, res) => {
  const todos = await todoService.getAllTodos();

  res.json({
    success: true,
    data: todos,
  });
});
```

If an error happens, `asyncHandler` automatically sends it to:

```js
app.use(errorMiddleware);
```

---

# 18. Full Todo API Example

This example uses:

- REST API design
- Express Router
- Controllers
- Services
- Custom error class
- Error handling middleware
- Async handler

---

## Project Structure

```txt
todo-backend/
  package.json
  server.js
  app.js
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

## Step 1: Install Packages

```bash
npm init -y
npm install express
```

---

## Step 2: server.js

```js
const app = require("./app");

const PORT = 3000;

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

---

## Step 3: app.js

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

// Error middleware should be after routes
app.use(errorMiddleware);

module.exports = app;
```

---

## Step 4: routes/todo.routes.js

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

Because of this line in `app.js`:

```js
app.use("/todos", todoRoutes);
```

These routes become:

```http
GET /todos
GET /todos/:id
POST /todos
PUT /todos/:id
DELETE /todos/:id
```

---

## Step 5: controllers/todo.controller.js

```js
const todoService = require("../services/todo.service");
const asyncHandler = require("../utils/asyncHandler");

const getTodos = asyncHandler(async (req, res) => {
  const todos = await todoService.getAllTodos();

  res.status(200).json({
    success: true,
    message: "Todos fetched successfully",
    data: todos,
  });
});

const getTodoById = asyncHandler(async (req, res) => {
  const id = Number(req.params.id);

  const todo = await todoService.getTodoById(id);

  res.status(200).json({
    success: true,
    message: "Todo fetched successfully",
    data: todo,
  });
});

const createTodo = asyncHandler(async (req, res) => {
  const todo = await todoService.createTodo(req.body);

  res.status(201).json({
    success: true,
    message: "Todo created successfully",
    data: todo,
  });
});

const updateTodo = asyncHandler(async (req, res) => {
  const id = Number(req.params.id);

  const updatedTodo = await todoService.updateTodo(id, req.body);

  res.status(200).json({
    success: true,
    message: "Todo updated successfully",
    data: updatedTodo,
  });
});

const deleteTodo = asyncHandler(async (req, res) => {
  const id = Number(req.params.id);

  const deletedTodo = await todoService.deleteTodo(id);

  res.status(200).json({
    success: true,
    message: "Todo deleted successfully",
    data: deletedTodo,
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

Notice:

- Controller reads `req.params`
- Controller reads `req.body`
- Controller calls service
- Controller sends response
- Controller does not contain main business logic

---

## Step 6: services/todo.service.js

```js
const ApiError = require("../utils/ApiError");

let todos = [];
let idCounter = 1;

async function getAllTodos() {
  return todos;
}

async function getTodoById(id) {
  if (!id) {
    throw new ApiError(400, "Todo ID is required");
  }

  const todo = todos.find((item) => item.id === id);

  if (!todo) {
    throw new ApiError(404, "Todo not found");
  }

  return todo;
}

async function createTodo(data) {
  const { title, description } = data;

  if (!title || !description) {
    throw new ApiError(400, "Title and description are required");
  }

  const newTodo = {
    id: idCounter++,
    title,
    description,
    completed: false,
  };

  todos.push(newTodo);

  return newTodo;
}

async function updateTodo(id, data) {
  if (!id) {
    throw new ApiError(400, "Todo ID is required");
  }

  const todo = todos.find((item) => item.id === id);

  if (!todo) {
    throw new ApiError(404, "Todo not found");
  }

  todo.title = data.title ?? todo.title;
  todo.description = data.description ?? todo.description;
  todo.completed = data.completed ?? todo.completed;

  return todo;
}

async function deleteTodo(id) {
  if (!id) {
    throw new ApiError(400, "Todo ID is required");
  }

  const todoIndex = todos.findIndex((item) => item.id === id);

  if (todoIndex === -1) {
    throw new ApiError(404, "Todo not found");
  }

  const deletedTodo = todos.splice(todoIndex, 1);

  return deletedTodo[0];
}

module.exports = {
  getAllTodos,
  getTodoById,
  createTodo,
  updateTodo,
  deleteTodo,
};
```

Notice:

- Service does not use `req`
- Service does not use `res`
- Service returns data
- Service throws errors
- Service contains main logic

---

## Step 7: utils/ApiError.js

```js
class ApiError extends Error {
  constructor(statusCode, message) {
    super(message);

    this.statusCode = statusCode;
    this.success = false;
  }
}

module.exports = ApiError;
```

---

## Step 8: utils/asyncHandler.js

```js
function asyncHandler(fn) {
  return function (req, res, next) {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}

module.exports = asyncHandler;
```

---

## Step 9: middlewares/error.middleware.js

```js
function errorMiddleware(err, req, res, next) {
  console.error("Error:", err.message);

  const statusCode = err.statusCode || 500;

  res.status(statusCode).json({
    success: false,
    message: err.message || "Internal Server Error",
  });
}

module.exports = errorMiddleware;
```

---

## How the Full Flow Works

Example request:

```http
POST /todos
```

With body:

```json
{
  "title": "Learn backend architecture",
  "description": "Understand routes, controllers and services"
}
```

Flow:

```txt
Client
  |
  v
Express app
  |
  v
/todos route
  |
  v
todo.routes.js
  |
  v
createTodo controller
  |
  v
createTodo service
  |
  v
todos array
  |
  v
service returns new todo
  |
  v
controller sends response
  |
  v
client receives JSON
```

If error happens:

```txt
Service throws ApiError
  |
  v
asyncHandler catches it
  |
  v
next(error)
  |
  v
error.middleware.js
  |
  v
JSON error response
```

---

# 19. Practice Tasks

## Task 1: Convert Single File Server Into Multiple Files

Take a simple Express Todo API and split it into:

```txt
routes/
controllers/
services/
middlewares/
utils/
```

---

## Task 2: Add RESTful Product Routes

Create product routes:

```http
GET /products
POST /products
GET /products/:id
PUT /products/:id
DELETE /products/:id
```

Each product should have:

```js
{
  (id, name, price, description);
}
```

---

## Task 3: Add Error Handling Middleware

Create a route:

```http
GET /error
```

Throw an error inside it:

```js
throw new Error("Testing error middleware");
```

Make sure your error middleware catches it.

---

## Task 4: Add Custom ApiError

Create `ApiError.js`.

Use it when product is not found:

```js
throw new ApiError(404, "Product not found");
```

---

## Task 5: Add asyncHandler

Create `asyncHandler.js`.

Wrap all controllers with it.

---

# Quick Revision

## REST API Design

Use nouns in URLs and HTTP methods for actions.

Good:

```http
GET /todos
POST /todos
PUT /todos/:id
DELETE /todos/:id
```

Bad:

```http
GET /getTodos
POST /createTodo
POST /deleteTodo
```

---

## Express Router

Splits routes into separate files.

```js
app.use("/todos", todoRoutes);
```

---

## Controller

Handles request and response.

```txt
req -> service -> res
```

---

## Service

Contains business logic.

```txt
data validation
database operations
calculation
throwing errors
```

---

## Error Middleware

Central place to handle errors.

```js
function errorMiddleware(err, req, res, next) {}
```

---

# Final Mental Model

```txt
server.js
  |
  v
app.js
  |
  v
routes
  |
  v
controllers
  |
  v
services
  |
  v
database / array / external APIs
```

For errors:

```txt
service/controller throws error
  |
  v
asyncHandler catches error
  |
  v
error middleware sends JSON response
```

---

# Suggested Next Topics

After this, learn:

1. Environment variables
2. MongoDB or PostgreSQL connection
3. Models and schemas
4. Input validation with Zod or Joi
5. JWT authentication
6. Cookies and sessions
7. Password hashing with bcrypt
8. Authorization and roles
9. Pagination, filtering, and sorting
10. Testing APIs with Jest and Supertest
