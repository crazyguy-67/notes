# Backend Notes: Environment Variables

These notes explain environment variables for backend development.

Goal: understand how to safely store configuration values like database URLs, secrets, API keys, ports, and frontend URLs outside your source code.

---

## Table of Contents

1. [What Are Environment Variables?](#1-what-are-environment-variables)
2. [Why Environment Variables Matter](#2-why-environment-variables-matter)
3. [What Is a `.env` File?](#3-what-is-a-env-file)
4. [Using `dotenv` in Node.js](#4-using-dotenv-in-nodejs)
5. [Common Environment Variables](#5-common-environment-variables)
6. [`.env` vs `.env.example`](#6-env-vs-envexample)
7. [Why You Must Add `.env` to `.gitignore`](#7-why-you-must-add-env-to-gitignore)
8. [Environment Variables in Express](#8-environment-variables-in-express)
9. [Environment Variables with CORS](#9-environment-variables-with-cors)
10. [Environment Variables with Database URLs](#10-environment-variables-with-database-urls)
11. [Local vs Production Environment](#11-local-vs-production-environment)
12. [Best Practices](#12-best-practices)
13. [Practice Tasks](#13-practice-tasks)

---

# 1. What Are Environment Variables?

Environment variables are values that live outside your source code.

They are used to store configuration values such as:

- Server port
- Database URL
- JWT secret
- API keys
- Frontend URL
- Backend URL
- Email credentials
- Cloudinary credentials
- Stripe keys
- OAuth client IDs and secrets

Example:

```env
PORT=3000
DATABASE_URL=mongodb://localhost:27017/todo-app
JWT_SECRET=my_super_secret_key
CLIENT_URL=http://localhost:5173
```

Instead of hardcoding important values inside your code, you read them from `process.env`.

Bad:

```js
const PORT = 3000;
const JWT_SECRET = "mysecret";
const DATABASE_URL = "mongodb://localhost:27017/todo-app";
```

Good:

```js
const PORT = process.env.PORT;
const JWT_SECRET = process.env.JWT_SECRET;
const DATABASE_URL = process.env.DATABASE_URL;
```

---

# 2. Why Environment Variables Matter

Environment variables are important because they help with:

1. Security
2. Flexibility
3. Deployment
4. Different environments
5. Clean code

---

## 2.1 Security

You should not push secrets to GitHub.

Bad:

```js
const JWT_SECRET = "super_secret_key";
```

If this goes to GitHub, anyone can see it.

Good:

```js
const JWT_SECRET = process.env.JWT_SECRET;
```

The secret stays outside your code.

---

## 2.2 Flexibility

Your local database and production database are usually different.

Local:

```env
DATABASE_URL=mongodb://localhost:27017/myapp
```

Production:

```env
DATABASE_URL=mongodb+srv://username:password@cluster.mongodb.net/myapp
```

Your code stays the same:

```js
mongoose.connect(process.env.DATABASE_URL);
```

Only the environment value changes.

---

## 2.3 Deployment

Deployment platforms like Render, Railway, Fly.io, Vercel, Netlify, and AWS allow you to add environment variables from their dashboard.

Usually you do not upload your local `.env` file to production.

You manually add values in the hosting dashboard.

---

# 3. What Is a `.env` File?

A `.env` file stores environment variables for local development.

Example `.env`:

```env
PORT=3000
NODE_ENV=development
DATABASE_URL=mongodb://localhost:27017/todo-app
JWT_SECRET=my_super_secret_key
CLIENT_URL=http://localhost:5173
```

Rules:

- Usually use uppercase names
- No spaces around `=`
- Do not push `.env` to GitHub
- Use meaningful variable names
- Keep one variable per line

Correct:

```env
PORT=3000
JWT_SECRET=mysecret
```

Avoid:

```env
PORT = 3000
JWT_SECRET = mysecret
```

---

# 4. Using `dotenv` in Node.js

Node.js does not automatically read `.env` files.

You need the `dotenv` package.

Install:

```bash
npm install dotenv
```

---

## CommonJS Style

If your project uses `require`:

```js
require("dotenv").config();
```

Example:

```js
require("dotenv").config();

const express = require("express");

const app = express();

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

## ES Module Style

If your `package.json` has:

```json
{
  "type": "module"
}
```

Use:

```js
import dotenv from "dotenv";

dotenv.config();
```

Example:

```js
import express from "express";
import dotenv from "dotenv";

dotenv.config();

const app = express();

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

# 5. Common Environment Variables

## 5.1 PORT

Used to decide which port your backend runs on.

```env
PORT=3000
```

Use:

```js
const PORT = process.env.PORT || 3000;
```

Why fallback?

If `process.env.PORT` is missing, your server still runs on `3000`.

---

## 5.2 NODE_ENV

Tells your app which environment it is running in.

```env
NODE_ENV=development
```

Common values:

```txt
development
production
test
```

Example:

```js
if (process.env.NODE_ENV === "development") {
  console.log("Development mode");
}
```

---

## 5.3 DATABASE_URL

Stores your database connection string.

MongoDB:

```env
DATABASE_URL=mongodb://localhost:27017/todo-app
```

PostgreSQL:

```env
DATABASE_URL=postgresql://postgres:password@localhost:5432/todo_app
```

Use:

```js
const databaseUrl = process.env.DATABASE_URL;
```

---

## 5.4 JWT_SECRET

Used to sign and verify JWT tokens.

```env
JWT_SECRET=my_super_secret_key
```

Use:

```js
const token = jwt.sign(payload, process.env.JWT_SECRET);
```

---

## 5.5 CLIENT_URL

Used for CORS.

```env
CLIENT_URL=http://localhost:5173
```

Use:

```js
app.use(
  cors({
    origin: process.env.CLIENT_URL,
    credentials: true,
  }),
);
```

---

## 5.6 API Keys

Example:

```env
OPENAI_API_KEY=your_api_key_here
CLOUDINARY_API_KEY=your_cloudinary_key_here
STRIPE_SECRET_KEY=your_stripe_key_here
```

Use:

```js
const apiKey = process.env.OPENAI_API_KEY;
```

---

# 6. `.env` vs `.env.example`

## `.env`

This file contains real values.

Example:

```env
PORT=3000
DATABASE_URL=mongodb://localhost:27017/todo-app
JWT_SECRET=actual_secret_key
CLIENT_URL=http://localhost:5173
```

You should not push this file to GitHub.

---

## `.env.example`

This file contains placeholder values.

Example:

```env
PORT=3000
DATABASE_URL=your_database_url_here
JWT_SECRET=your_jwt_secret_here
CLIENT_URL=http://localhost:5173
```

You can push this to GitHub.

Why?

Because it tells other developers what environment variables are needed without exposing real secrets.

---

# 7. Why You Must Add `.env` to `.gitignore`

Your `.env` file may contain private data.

Examples:

- Database password
- JWT secret
- API keys
- Stripe secret key
- OAuth client secret
- Email credentials

Add `.env` to `.gitignore`.

Example `.gitignore`:

```gitignore
node_modules
.env
dist
build
coverage
```

This prevents Git from tracking `.env`.

---

# 8. Environment Variables in Express

Basic Express example:

```js
require("dotenv").config();

const express = require("express");

const app = express();

app.use(express.json());

const PORT = process.env.PORT || 3000;

app.get("/", (req, res) => {
  res.json({
    success: true,
    message: "API is running",
    environment: process.env.NODE_ENV,
  });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

Example `.env`:

```env
PORT=5000
NODE_ENV=development
```

If you run the server, it will run on port `5000`.

---

# 9. Environment Variables with CORS

Install:

```bash
npm install cors
```

Example `.env`:

```env
CLIENT_URL=http://localhost:5173
```

Express setup:

```js
require("dotenv").config();

const express = require("express");
const cors = require("cors");

const app = express();

app.use(
  cors({
    origin: process.env.CLIENT_URL,
    credentials: true,
  }),
);

app.use(express.json());

app.get("/", (req, res) => {
  res.json({
    message: "CORS configured using environment variables",
  });
});

app.listen(process.env.PORT || 3000);
```

Why this is useful:

In development:

```env
CLIENT_URL=http://localhost:5173
```

In production:

```env
CLIENT_URL=https://myapp.com
```

Your code remains the same.

---

# 10. Environment Variables with Database URLs

## MongoDB Example

`.env`:

```env
DATABASE_URL=mongodb://localhost:27017/todo-app
```

Connection:

```js
const mongoose = require("mongoose");

async function connectDB() {
  await mongoose.connect(process.env.DATABASE_URL);
  console.log("MongoDB connected");
}

module.exports = connectDB;
```

---

## PostgreSQL Example

`.env`:

```env
DATABASE_URL=postgresql://postgres:password@localhost:5432/todo_app
```

With Prisma:

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

With `pg`:

```js
const { Pool } = require("pg");

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
});
```

---

# 11. Local vs Production Environment

Your local `.env` file is for your own machine.

Example local `.env`:

```env
PORT=3000
NODE_ENV=development
DATABASE_URL=mongodb://localhost:27017/todo-app
CLIENT_URL=http://localhost:5173
```

Production environment variables are added to your hosting provider.

Example production values:

```env
PORT=10000
NODE_ENV=production
DATABASE_URL=mongodb+srv://user:password@cluster.mongodb.net/todo-app
CLIENT_URL=https://todo-app.com
```

Same code works in both places:

```js
const PORT = process.env.PORT || 3000;
```

---

# 12. Best Practices

## 12.1 Never Commit `.env`

Always keep `.env` inside `.gitignore`.

---

## 12.2 Commit `.env.example`

This helps others know what values are needed.

---

## 12.3 Use Clear Names

Good:

```env
DATABASE_URL=
JWT_SECRET=
CLIENT_URL=
```

Bad:

```env
DB=
SECRET=
URL=
```

---

## 12.4 Use Strong Secrets

Bad:

```env
JWT_SECRET=123
```

Better:

```env
JWT_SECRET=a_long_random_secret_string_here
```

---

## 12.5 Validate Required Env Variables

In bigger projects, you can check if required variables exist.

Example:

```js
const requiredEnvVars = ["PORT", "DATABASE_URL", "JWT_SECRET"];

for (const envVar of requiredEnvVars) {
  if (!process.env[envVar]) {
    throw new Error(`${envVar} is missing in .env`);
  }
}
```

---

# 13. Practice Tasks

## Task 1

Create a `.env` file:

```env
PORT=3000
NODE_ENV=development
DATABASE_URL=your_database_url
JWT_SECRET=your_secret
CLIENT_URL=http://localhost:5173
```

---

## Task 2

Create a `.env.example` file:

```env
PORT=3000
NODE_ENV=development
DATABASE_URL=your_database_url_here
JWT_SECRET=your_jwt_secret_here
CLIENT_URL=http://localhost:5173
```

---

## Task 3

Add `.env` to `.gitignore`.

```gitignore
node_modules
.env
```

---

## Task 4

Use `process.env.PORT` in Express.

```js
const PORT = process.env.PORT || 3000;
```

---

## Task 5

Use `process.env.CLIENT_URL` in CORS config.

```js
app.use(
  cors({
    origin: process.env.CLIENT_URL,
    credentials: true,
  }),
);
```

---

# Quick Revision

Environment variables store configuration outside your code.

```js
process.env.PORT;
process.env.DATABASE_URL;
process.env.JWT_SECRET;
```

Use `.env` locally.

Do not push `.env`.

Push `.env.example`.

Use `dotenv` to load `.env`.

```js
require("dotenv").config();
```

Final mental model:

```txt
.env file
  |
  v
dotenv.config()
  |
  v
process.env
  |
  v
your backend code
```
