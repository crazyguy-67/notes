# Backend Fundamentals Notes: Headers, Fetch API, HTTP Server, Routes, Middleware, and CORS

These notes are for a full-stack developer learning how the frontend and backend communicate.

---

## Table of Contents

1. [What Are HTTP Headers?](#1-what-are-http-headers)
2. [Common Request Headers](#2-common-request-headers)
3. [Common Response Headers](#3-common-response-headers)
4. [The Fetch API](#4-the-fetch-api)
5. [GET Request Using Fetch](#5-get-request-using-fetch)
6. [POST Request Using Fetch](#6-post-request-using-fetch)
7. [PUT Request Using Fetch](#7-put-request-using-fetch)
8. [DELETE Request Using Fetch](#8-delete-request-using-fetch)
9. [HTTP Server With Node Core Module](#9-http-server-with-node-core-module)
10. [HTTP Server With Routes](#10-http-server-with-routes)
11. [GET, POST, PUT, DELETE Routes](#11-get-post-put-delete-routes)
12. [What Is Middleware?](#12-what-is-middleware)
13. [Global Middleware](#13-global-middleware)
14. [Route-Specific Middleware](#14-route-specific-middleware)
15. [CORS in Detail](#15-cors-in-detail)
16. [Full Express Example](#16-full-express-example)
17. [Practice Tasks](#17-practice-tasks)

---

# 1. What Are HTTP Headers?

HTTP headers are extra information sent along with an HTTP request or response.

Think of an HTTP request like a parcel.

The main content is the actual package.

Headers are like labels on the package:

- Who is sending it?
- What type of content is inside?
- Is authentication required?
- Is it JSON, HTML, text, or something else?
- Can the browser cache this?
- Which origin is allowed to access the response?

Example HTTP request:

```http
POST /users HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer my_token

{
  "name": "Abhishek"
}
```

Here:

```http
Content-Type: application/json
Authorization: Bearer my_token
```

These are headers.

---

# 2. Common Request Headers

Request headers are sent by the client to the server.

## 2.1 Content-Type

Tells the server what type of data the client is sending.

Example:

```http
Content-Type: application/json
```

This means:

> I am sending JSON data in the request body.

Used mostly with:

- POST
- PUT
- PATCH

Example fetch:

```js
fetch("http://localhost:3000/users", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    name: "Abhishek",
  }),
});
```

---

## 2.2 Authorization

Used to send authentication information.

Example:

```http
Authorization: Bearer my_jwt_token
```

Example fetch:

```js
fetch("http://localhost:3000/profile", {
  headers: {
    Authorization: "Bearer my_jwt_token",
  },
});
```

Usually used for:

- JWT auth
- API keys
- protected routes

---

## 2.3 Accept

Tells the server what kind of response the client expects.

Example:

```http
Accept: application/json
```

Meaning:

> Please send me JSON if possible.

---

## 2.4 Origin

Automatically sent by the browser when making cross-origin requests.

Example:

```http
Origin: http://localhost:5173
```

This tells the server:

> This request came from `http://localhost:5173`.

Very important for CORS.

---

# 3. Common Response Headers

Response headers are sent by the server to the client.

## 3.1 Content-Type

Tells the client what type of data the server is sending back.

Example:

```http
Content-Type: application/json
```

Example Node server:

```js
res.writeHead(200, {
  "Content-Type": "application/json",
});
```

---

## 3.2 Access-Control-Allow-Origin

Used for CORS.

Example:

```http
Access-Control-Allow-Origin: http://localhost:5173
```

Meaning:

> I allow this frontend origin to access my response.

---

## 3.3 Set-Cookie

Used by the server to set cookies in the browser.

Example:

```http
Set-Cookie: token=abc123; HttpOnly; Secure
```

---

## 3.4 Cache-Control

Controls browser caching.

Example:

```http
Cache-Control: no-store
```

Meaning:

> Do not cache this response.

---

# 4. The Fetch API

`fetch()` is a browser API used to make HTTP requests from frontend JavaScript.

Basic syntax:

```js
fetch(url, options);
```

Example:

```js
fetch("http://localhost:3000/todos");
```

`fetch()` returns a Promise.

So we usually use it with:

- `.then()`
- `async/await`

Modern style:

```js
async function getTodos() {
  const response = await fetch("http://localhost:3000/todos");
  const data = await response.json();

  console.log(data);
}
```

Important:

```js
await response.json();
```

This converts the JSON response into a JavaScript object or array.

---

# 5. GET Request Using Fetch

GET is used to read data.

Example:

```js
async function getTodos() {
  const response = await fetch("http://localhost:3000/todos");

  const todos = await response.json();

  console.log(todos);
}

getTodos();
```

No body is usually sent with GET requests.

---

# 6. POST Request Using Fetch

POST is used to create new data.

Example:

```js
async function createTodo() {
  const response = await fetch("http://localhost:3000/todos", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      title: "Learn Backend",
      description: "Understand HTTP routes",
    }),
  });

  const data = await response.json();

  console.log(data);
}

createTodo();
```

Important lines:

```js
headers: {
  "Content-Type": "application/json",
}
```

This tells the server:

> The data I am sending is JSON.

```js
body: JSON.stringify(...)
```

This converts a JavaScript object into a JSON string.

Why?

Because HTTP sends text/data over the network, not direct JavaScript objects.

---

# 7. PUT Request Using Fetch

PUT is usually used to update existing data.

Example:

```js
async function updateTodo(id) {
  const response = await fetch(`http://localhost:3000/todos/${id}`, {
    method: "PUT",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      title: "Updated Todo",
      description: "Updated description",
    }),
  });

  const data = await response.json();

  console.log(data);
}

updateTodo(1);
```

---

# 8. DELETE Request Using Fetch

DELETE is used to remove data.

Example:

```js
async function deleteTodo(id) {
  const response = await fetch(`http://localhost:3000/todos/${id}`, {
    method: "DELETE",
  });

  const data = await response.json();

  console.log(data);
}

deleteTodo(1);
```

---

# 9. HTTP Server With Node Core Module

Node.js has a built-in `http` module.

You can create a server without Express.

Example:

```js
const http = require("http");

const server = http.createServer((req, res) => {
  res.writeHead(200, {
    "Content-Type": "text/plain",
  });

  res.end("Hello from Node HTTP server");
});

server.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

Here:

```js
req;
```

means request.

It contains information about the incoming request.

Example:

- URL
- method
- headers
- body stream

```js
res;
```

means response.

It is used to send data back to the client.

---

# 10. HTTP Server With Routes

A route means:

> If the client visits this URL with this method, run this logic.

Example:

```http
GET /todos
POST /todos
PUT /todos/1
DELETE /todos/1
```

In Node core HTTP module, routing is manual.

Example:

```js
const http = require("http");

const server = http.createServer((req, res) => {
  if (req.method === "GET" && req.url === "/") {
    res.writeHead(200, {
      "Content-Type": "text/plain",
    });

    return res.end("Home Page");
  }

  if (req.method === "GET" && req.url === "/about") {
    res.writeHead(200, {
      "Content-Type": "text/plain",
    });

    return res.end("About Page");
  }

  res.writeHead(404, {
    "Content-Type": "application/json",
  });

  res.end(
    JSON.stringify({
      message: "Route not found",
    }),
  );
});

server.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

---

# 11. GET, POST, PUT, DELETE Routes

Below is a basic Todo API using only Node's core `http` module.

## Full Example

```js
const http = require("http");

let todos = [];
let idCounter = 1;

function sendJSON(res, statusCode, data) {
  res.writeHead(statusCode, {
    "Content-Type": "application/json",
  });

  res.end(JSON.stringify(data));
}

function getRequestBody(req) {
  return new Promise((resolve, reject) => {
    let body = "";

    req.on("data", (chunk) => {
      body += chunk.toString();
    });

    req.on("end", () => {
      try {
        const parsedBody = body ? JSON.parse(body) : {};
        resolve(parsedBody);
      } catch (error) {
        reject(error);
      }
    });
  });
}

const server = http.createServer(async (req, res) => {
  const url = new URL(req.url, `http://${req.headers.host}`);

  // GET /todos
  if (req.method === "GET" && url.pathname === "/todos") {
    return sendJSON(res, 200, todos);
  }

  // GET /todo?id=1
  if (req.method === "GET" && url.pathname === "/todo") {
    const id = Number(url.searchParams.get("id"));

    const todo = todos.find((item) => item.id === id);

    if (!todo) {
      return sendJSON(res, 404, {
        message: "Todo not found",
      });
    }

    return sendJSON(res, 200, todo);
  }

  // POST /todos
  if (req.method === "POST" && url.pathname === "/todos") {
    try {
      const body = await getRequestBody(req);

      const newTodo = {
        id: idCounter++,
        title: body.title,
        description: body.description,
      };

      todos.push(newTodo);

      return sendJSON(res, 201, {
        message: "Todo created successfully",
        todos,
      });
    } catch (error) {
      return sendJSON(res, 400, {
        message: "Invalid JSON body",
      });
    }
  }

  // PUT /todo?id=1
  if (req.method === "PUT" && url.pathname === "/todo") {
    try {
      const id = Number(url.searchParams.get("id"));
      const body = await getRequestBody(req);

      const todo = todos.find((item) => item.id === id);

      if (!todo) {
        return sendJSON(res, 404, {
          message: "Todo not found",
        });
      }

      todo.title = body.title ?? todo.title;
      todo.description = body.description ?? todo.description;

      return sendJSON(res, 200, {
        message: "Todo updated successfully",
        todo,
      });
    } catch (error) {
      return sendJSON(res, 400, {
        message: "Invalid JSON body",
      });
    }
  }

  // DELETE /todo?id=1
  if (req.method === "DELETE" && url.pathname === "/todo") {
    const id = Number(url.searchParams.get("id"));

    const todoIndex = todos.findIndex((item) => item.id === id);

    if (todoIndex === -1) {
      return sendJSON(res, 404, {
        message: "Todo not found",
      });
    }

    todos.splice(todoIndex, 1);

    return sendJSON(res, 200, {
      message: "Todo deleted successfully",
      todos,
    });
  }

  return sendJSON(res, 404, {
    message: "Route not found",
  });
});

server.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

---

## How This Server Works

When a request comes:

```js
const server = http.createServer(async (req, res) => {
```

Node gives us two objects:

```js
req;
```

The request object.

```js
res;
```

The response object.

Then we check:

```js
req.method;
```

Example:

```js
GET;
POST;
PUT;
DELETE;
```

And we check:

```js
url.pathname;
```

Example:

```js
/todos
/todo
```

Then we decide which block of code should run.

---

# 12. What Is Middleware?

Middleware is a function that runs between the request and the final route handler.

Request flow:

```txt
Client
  |
  v
Middleware 1
  |
  v
Middleware 2
  |
  v
Route Handler
  |
  v
Response
```

Middleware can be used for:

- Logging requests
- Checking authentication
- Validating API keys
- Parsing JSON body
- Handling CORS
- Checking user roles
- Error handling

---

## Middleware in Express

Basic middleware syntax:

```js
function middlewareName(req, res, next) {
  // logic here

  next();
}
```

Important:

```js
next();
```

means:

> Go to the next middleware or final route handler.

If you do not call `next()` and do not send a response, the request will hang.

---

# 13. Global Middleware

Global middleware runs for every request.

Example:

```js
const express = require("express");

const app = express();

function logger(req, res, next) {
  console.log(`${req.method} ${req.url}`);

  next();
}

app.use(logger);

app.get("/", (req, res) => {
  res.json({
    message: "Home route",
  });
});

app.get("/todos", (req, res) => {
  res.json({
    message: "Todos route",
  });
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

Here:

```js
app.use(logger);
```

means the `logger` middleware will run for every route.

---

## Example Output

If client requests:

```http
GET /todos
```

Console:

```txt
GET /todos
```

---

# 14. Route-Specific Middleware

Route-specific middleware runs only for selected routes.

Example:

```js
const express = require("express");

const app = express();

function checkAuth(req, res, next) {
  const token = req.headers.authorization;

  if (!token) {
    return res.status(401).json({
      message: "Unauthorized: token missing",
    });
  }

  next();
}

app.get("/", (req, res) => {
  res.json({
    message: "Public route",
  });
});

app.get("/profile", checkAuth, (req, res) => {
  res.json({
    message: "Protected profile route",
  });
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

Here:

```js
app.get("/profile", checkAuth, handler);
```

means:

> Run `checkAuth` only when someone visits `/profile`.

---

## Multiple Route-Specific Middlewares

```js
function checkAuth(req, res, next) {
  console.log("Checking authentication");
  next();
}

function checkAdmin(req, res, next) {
  console.log("Checking admin role");
  next();
}

app.get("/admin", checkAuth, checkAdmin, (req, res) => {
  res.json({
    message: "Welcome admin",
  });
});
```

Flow:

```txt
Request /admin
  |
  v
checkAuth
  |
  v
checkAdmin
  |
  v
final route handler
```

---

# 15. CORS in Detail

CORS stands for:

```txt
Cross-Origin Resource Sharing
```

It is a browser security mechanism.

It decides whether a frontend running on one origin is allowed to access a backend running on another origin.

---

## What Is an Origin?

An origin is made of three things:

```txt
protocol + domain + port
```

Example:

```txt
http://localhost:5173
```

Here:

```txt
protocol = http
domain = localhost
port = 5173
```

Another origin:

```txt
http://localhost:3000
```

Here:

```txt
protocol = http
domain = localhost
port = 3000
```

These two are different origins because their ports are different.

So this is cross-origin:

```txt
Frontend: http://localhost:5173
Backend:  http://localhost:3000
```

---

## Why Does CORS Exist?

Imagine you are logged into your bank website.

A malicious website could try to make requests to your bank in the background.

CORS helps browsers stop random websites from reading responses from another website unless the server allows it.

Important:

> CORS is enforced by the browser.

Backend tools like Postman, Thunder Client, curl, or server-to-server requests are not blocked by browser CORS in the same way.

---

## Common CORS Error

You may see something like:

```txt
Access to fetch at 'http://localhost:3000/todos'
from origin 'http://localhost:5173'
has been blocked by CORS policy.
```

Meaning:

> The browser sent the request, but it did not allow your frontend JavaScript to read the response because the backend did not allow that origin.

---

## Same-Origin vs Cross-Origin

Same origin:

```txt
Frontend: http://localhost:3000
Backend:  http://localhost:3000
```

Cross origin:

```txt
Frontend: http://localhost:5173
Backend:  http://localhost:3000
```

Different port means different origin.

---

## How Server Allows CORS

Backend sends this response header:

```http
Access-Control-Allow-Origin: http://localhost:5173
```

This tells the browser:

> It is okay for frontend from `http://localhost:5173` to read this response.

---

## Manual CORS in Node HTTP Server

```js
const http = require("http");

const server = http.createServer((req, res) => {
  res.setHeader("Access-Control-Allow-Origin", "http://localhost:5173");
  res.setHeader(
    "Access-Control-Allow-Methods",
    "GET, POST, PUT, DELETE, OPTIONS",
  );
  res.setHeader("Access-Control-Allow-Headers", "Content-Type, Authorization");

  if (req.method === "OPTIONS") {
    res.writeHead(204);
    return res.end();
  }

  res.writeHead(200, {
    "Content-Type": "application/json",
  });

  res.end(
    JSON.stringify({
      message: "CORS enabled",
    }),
  );
});

server.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

---

## CORS With Express

Install CORS package:

```bash
npm install cors
```

Use it:

```js
const express = require("express");
const cors = require("cors");

const app = express();

app.use(
  cors({
    origin: "http://localhost:5173",
  }),
);

app.get("/todos", (req, res) => {
  res.json([
    {
      id: 1,
      title: "Learn CORS",
    },
  ]);
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

---

## What Is a Preflight Request?

For some requests, the browser first sends an `OPTIONS` request before the real request.

This is called a preflight request.

Example:

Before sending:

```http
PUT /todos/1
Content-Type: application/json
Authorization: Bearer token
```

Browser may first send:

```http
OPTIONS /todos/1
Origin: http://localhost:5173
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Content-Type, Authorization
```

The server must reply with allowed methods and headers:

```http
Access-Control-Allow-Origin: http://localhost:5173
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
```

If the server allows it, then the browser sends the real request.

---

## Simple Requests vs Preflight Requests

Some requests are considered simple.

Example:

```http
GET
POST with basic content types
HEAD
```

But requests with custom headers or methods like PUT/DELETE often trigger preflight.

Preflight usually happens when using:

- `PUT`
- `DELETE`
- `PATCH`
- `Authorization` header
- `Content-Type: application/json`

---

## CORS With Credentials

Credentials means cookies, authorization headers, or TLS client certificates.

If frontend needs to send cookies:

```js
fetch("http://localhost:3000/profile", {
  credentials: "include",
});
```

Backend must allow credentials:

```js
app.use(
  cors({
    origin: "http://localhost:5173",
    credentials: true,
  }),
);
```

Important:

If using credentials, you cannot use:

```http
Access-Control-Allow-Origin: *
```

You must specify the exact origin:

```http
Access-Control-Allow-Origin: http://localhost:5173
```

---

## Common CORS Mistakes

### Mistake 1: Backend does not use CORS

Frontend:

```txt
http://localhost:5173
```

Backend:

```txt
http://localhost:3000
```

If backend does not allow frontend origin, browser blocks the response.

---

### Mistake 2: Using credentials with wildcard origin

Wrong:

```js
cors({
  origin: "*",
  credentials: true,
});
```

Correct:

```js
cors({
  origin: "http://localhost:5173",
  credentials: true,
});
```

---

### Mistake 3: Missing allowed headers

If frontend sends:

```js
headers: {
  Authorization: "Bearer token",
}
```

Then backend must allow:

```http
Access-Control-Allow-Headers: Authorization
```

The `cors` Express package usually handles this for you.

---

# 16. Full Express Example

This is a clean Express backend with:

- JSON parsing middleware
- Logger middleware
- Route-specific auth middleware
- GET route
- POST route
- PUT route
- DELETE route
- CORS support

## Installation

```bash
npm init -y
npm install express cors
```

## server.js

```js
const express = require("express");
const cors = require("cors");

const app = express();

let todos = [];
let idCounter = 1;

// CORS middleware
app.use(
  cors({
    origin: "http://localhost:5173",
    credentials: true,
  }),
);

// JSON body parser middleware
app.use(express.json());

// Global logger middleware
function logger(req, res, next) {
  console.log(`${req.method} ${req.url}`);
  next();
}

app.use(logger);

// Route-specific middleware
function checkApiKey(req, res, next) {
  const apiKey = req.headers["x-api-key"];

  if (apiKey !== "secret123") {
    return res.status(401).json({
      message: "Invalid or missing API key",
    });
  }

  next();
}

// Public route
app.get("/", (req, res) => {
  res.json({
    message: "API is running",
  });
});

// GET all todos
app.get("/todos", (req, res) => {
  res.json(todos);
});

// GET single todo
app.get("/todos/:id", (req, res) => {
  const id = Number(req.params.id);

  const todo = todos.find((item) => item.id === id);

  if (!todo) {
    return res.status(404).json({
      message: "Todo not found",
    });
  }

  res.json(todo);
});

// POST create todo
app.post("/todos", checkApiKey, (req, res) => {
  const { title, description } = req.body;

  if (!title || !description) {
    return res.status(400).json({
      message: "Title and description are required",
    });
  }

  const newTodo = {
    id: idCounter++,
    title,
    description,
  };

  todos.push(newTodo);

  res.status(201).json({
    message: "Todo created successfully",
    todo: newTodo,
  });
});

// PUT update todo
app.put("/todos/:id", checkApiKey, (req, res) => {
  const id = Number(req.params.id);

  const todo = todos.find((item) => item.id === id);

  if (!todo) {
    return res.status(404).json({
      message: "Todo not found",
    });
  }

  todo.title = req.body.title ?? todo.title;
  todo.description = req.body.description ?? todo.description;

  res.json({
    message: "Todo updated successfully",
    todo,
  });
});

// DELETE todo
app.delete("/todos/:id", checkApiKey, (req, res) => {
  const id = Number(req.params.id);

  const todoIndex = todos.findIndex((item) => item.id === id);

  if (todoIndex === -1) {
    return res.status(404).json({
      message: "Todo not found",
    });
  }

  const deletedTodo = todos.splice(todoIndex, 1);

  res.json({
    message: "Todo deleted successfully",
    deletedTodo: deletedTodo[0],
  });
});

app.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

---

## Frontend Fetch Examples for This Express Server

### GET Todos

```js
async function getTodos() {
  const response = await fetch("http://localhost:3000/todos");
  const data = await response.json();

  console.log(data);
}
```

---

### POST Todo

```js
async function createTodo() {
  const response = await fetch("http://localhost:3000/todos", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "x-api-key": "secret123",
    },
    body: JSON.stringify({
      title: "Learn Express",
      description: "Understand routes and middleware",
    }),
  });

  const data = await response.json();

  console.log(data);
}
```

---

### PUT Todo

```js
async function updateTodo(id) {
  const response = await fetch(`http://localhost:3000/todos/${id}`, {
    method: "PUT",
    headers: {
      "Content-Type": "application/json",
      "x-api-key": "secret123",
    },
    body: JSON.stringify({
      title: "Updated title",
    }),
  });

  const data = await response.json();

  console.log(data);
}
```

---

### DELETE Todo

```js
async function deleteTodo(id) {
  const response = await fetch(`http://localhost:3000/todos/${id}`, {
    method: "DELETE",
    headers: {
      "x-api-key": "secret123",
    },
  });

  const data = await response.json();

  console.log(data);
}
```

---

# 17. Practice Tasks

## Task 1: Headers

Create an Express route:

```http
GET /headers
```

Return the request headers as JSON.

Hint:

```js
req.headers;
```

---

## Task 2: Logger Middleware

Create a middleware that logs:

```txt
METHOD URL TIME
```

Example:

```txt
GET /todos 2026-05-20T10:00:00.000Z
```

---

## Task 3: API Key Middleware

Create a middleware that checks:

```http
x-api-key: mysecretkey
```

If missing or wrong, return:

```js
{
  message: "Unauthorized";
}
```

---

## Task 4: Route-Specific Middleware

Protect only these routes:

```http
POST /todos
PUT /todos/:id
DELETE /todos/:id
```

Keep these routes public:

```http
GET /
GET /todos
GET /todos/:id
```

---

## Task 5: CORS

Create a backend on:

```txt
http://localhost:3000
```

Create a frontend on:

```txt
http://localhost:5173
```

Make a fetch request from frontend to backend.

Then enable CORS correctly.

---

# Quick Revision

## HTTP Headers

Headers are extra metadata sent with requests and responses.

## Fetch API

`fetch()` is used by frontend JavaScript to make HTTP requests.

## HTTP Server

A server receives requests and sends responses.

## Routes

Routes decide what logic should run for a specific URL and HTTP method.

## Middleware

Middleware runs before the final route handler.

## Route-Specific Middleware

Middleware that runs only for selected routes.

## CORS

CORS decides whether a frontend from one origin can read responses from a backend on another origin.

---

# Final Mental Model

```txt
Frontend fetch()
      |
      v
HTTP Request
      |
      v
Backend server
      |
      v
Global middleware
      |
      v
Route-specific middleware
      |
      v
Route handler
      |
      v
HTTP Response
      |
      v
Frontend receives data
```

Example:

```txt
React app on localhost:5173
      |
      | fetch("http://localhost:3000/todos")
      v
Express backend on localhost:3000
      |
      | CORS allows frontend
      v
Response sent back as JSON
```

---

# Suggested Next Topics

After this, learn:

1. REST API design
2. Express Router
3. Controllers and services
4. Error handling middleware
5. Authentication with JWT
6. Cookies and sessions
7. Databases with PostgreSQL or MongoDB
8. Input validation with Zod or Joi
9. Environment variables
10. Deployment basics
