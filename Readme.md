# Express.js 5: Complete Production Engineering Guide

**Author**: Me "Ahmed Arafa"
**Date**: 2026  
**Target Audience**: Backend Engineers, Production Team Leads  
**Quality Level**: FAANG-Production Ready

---

## Table of Contents

1. [Introduction](#introduction)
2. [Project Setup](#project-setup)
3. [Express Core Concepts](#express-core-concepts)
4. [Routing Architecture](#routing-architecture)
5. [Middleware Deep Dive](#middleware-deep-dive)
6. [Request & Response Handling](#request--response-handling)
7. [Environment Configuration](#environment-configuration)
8. [MVC Architecture](#mvc-architecture)
9. [Production Folder Structure](#production-folder-structure)
10. [Validation Strategy](#validation-strategy)
11. [Authentication & Authorization](#authentication--authorization)
12. [Error Handling](#error-handling)
13. [Logging & Observability](#logging--observability)
14. [Security Best Practices](#security-best-practices)
15. [Database Integration](#database-integration)
16. [Performance Optimization](#performance-optimization)
17. [Testing Strategy](#testing-strategy)
18. [CI/CD Pipeline](#cicd-pipeline)
19. [Deployment](#deployment)
20. [Production Example Project](#production-example-project)

---

## 1. Introduction

### What is Express.js?

Express.js is a **minimal, flexible Node.js web application framework** that provides a robust set of features for building HTTP servers and APIs. Express 5.x introduces improved error handling, better TypeScript support, and modern patterns while maintaining backward compatibility where possible.

### Why Express for Production?

- **Minimal overhead**: Unopinionated, you control architecture
- **Massive ecosystem**: Middleware for any requirement exists
- **Battle-tested**: Used at scale by companies like Uber, Netflix, IBM
- **TypeScript-first**: Native type support since v5
- **Performance**: With proper optimization, handles millions of RPS

### Common Junior Mistakes to Avoid

1. ❌ Putting all routes in a single `app.ts` file
2. ❌ No environment separation (dev/staging/prod)
3. ❌ Synchronous error handling in async code
4. ❌ No rate limiting on public endpoints
5. ❌ Logging everything at INFO level
6. ❌ No request tracing/correlation IDs
7. ❌ Mixing business logic with route handlers
8. ❌ No graceful shutdown handling

---

## 2. Project Setup

### Prerequisites

```bash
# Node.js 18+ (Express 5 requirement)
node --version  # v18.0.0 or higher

# npm 9+
npm --version
```

### Initial Project Structure

```bash
# Create project
mkdir express-api-prod
cd express-api-prod

# Initialize
npm init -y

# Install core dependencies
npm install express@5

# Install dev dependencies
npm install -D typescript @types/node @types/express \
  ts-node nodemon \
  eslint prettier @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

### TypeScript & Module System Configuration

Choose your preferred module system:

#### Option A: CommonJS (Default, Traditional)

**`package.json` (CommonJS)**:

```json
{
  "name": "express-api",
  "version": "1.0.0",
  "main": "dist/index.js",
  "scripts": {
    "dev": "nodemon",
    "build": "tsc",
    "start": "node dist/index.js"
  }
}
```

**`tsconfig.json` (CommonJS)**:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "removeComments": false,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "types": ["node"]
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

**Code Example (CommonJS)**:

```typescript
// src/index.ts
import express from "express";
import dotenv from "dotenv";

dotenv.config();
const app = express();

app.use(express.json());
app.listen(3000, () => console.log("Running on port 3000"));
```

#### Option B: ES Modules (Modern, Recommended for new projects)

**`package.json` (ES Modules)**:

```json
{
  "name": "express-api",
  "version": "1.0.0",
  "type": "module",
  "main": "dist/index.js",
  "scripts": {
    "dev": "nodemon",
    "build": "tsc",
    "start": "node dist/index.js"
  }
}
```

**`tsconfig.json` (ES Modules)**:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "esnext",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": false,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "removeComments": false,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "types": ["node"]
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

**Code Example (ES Modules)**:

```typescript
// src/index.ts
import express from "express";
import dotenv from "dotenv";

dotenv.config();
const app = express();

app.use(express.json());

// ES Modules: Top-level await supported
await app.listen(3000);
console.log("Running on port 3000");
```

#### Which Should You Choose?

| Aspect                | CommonJS                 | ES Modules         |
| --------------------- | ------------------------ | ------------------ |
| **Compatibility**     | ✅ All Node versions     | ✅ v14+ (native)   |
| **Performance**       | Good                     | ⭐ Better (faster) |
| **Ecosystem**         | ✅ Most packages         | Improving          |
| **Top-level await**   | ❌ No                    | ✅ Yes             |
| **For new projects**  | ✅ Safe choice           | ⭐ Recommended     |
| **For existing code** | ✅ Usually already using | Consider migration |

---

### Package Scripts

**`package.json` (CommonJS)**:

```json
{
  "scripts": {
    "dev": "nodemon",
    "build": "tsc",
    "start": "node dist/index.js",
    "lint": "eslint src --ext .ts",
    "lint:fix": "eslint src --ext .ts --fix",
    "format": "prettier --write \"src/**/*.ts\"",
    "type-check": "tsc --noEmit",
    "test": "jest",
    "test:watch": "jest --watch"
  }
}
```

**`package.json` (ES Modules)**:

```json
{
  "type": "module",
  "scripts": {
    "dev": "nodemon --ext ts --exec node --loader ts-node/esm",
    "build": "tsc",
    "start": "node dist/index.js",
    "lint": "eslint src --ext .ts",
    "lint:fix": "eslint src --ext .ts --fix",
    "format": "prettier --write \"src/**/*.ts\"",
    "type-check": "tsc --noEmit",
    "test": "node --loader ts-node/esm node_modules/.bin/jest",
    "test:watch": "node --loader ts-node/esm node_modules/.bin/jest --watch"
  }
}
```

### Nodemon Configuration

#### For CommonJS

**`nodemon.json`**:

```json
{
  "watch": ["src"],
  "ext": "ts",
  "ignore": ["src/**/*.test.ts"],
  "exec": "ts-node",
  "execMap": {
    "ts": "ts-node --transpile-only"
  },
  "delay": 500,
  "env": {
    "NODE_ENV": "development"
  }
}
```

**`package.json` with nodemon**:

```json
{
  "scripts": {
    "dev": "nodemon"
  }
}
```

#### For ES Modules

**`nodemon.json`**:

```json
{
  "watch": ["src"],
  "ext": "ts",
  "ignore": ["src/**/*.test.ts"],
  "exec": "node --loader ts-node/esm",
  "delay": 500,
  "env": {
    "NODE_ENV": "development",
    "NODE_OPTIONS": "--loader ts-node/esm"
  }
}
```

**`package.json` with nodemon (ES Modules)**:

```json
{
  "type": "module",
  "scripts": {
    "dev": "nodemon --ext ts --exec node --loader ts-node/esm"
  }
}
```

---

## 3. Express Core Concepts

### The Application Object (`app`)

The Express application is your HTTP server factory. Every handler is middleware.

```typescript
import express, { Express, Request, Response } from "express";

const app: Express = express();

// All HTTP methods
app.get("/health", (req: Request, res: Response) => {
  res.json({ status: "ok" });
});

app.post("/api/users", (req: Request, res: Response) => {
  res.status(201).json({ created: true });
});

// Listen
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### Request Object (`req`)

The incoming HTTP request with parsed headers, body, query params.

```typescript
app.post("/api/users/:id", (req: Request, res: Response) => {
  // URL parameters
  const userId: string = req.params.id;

  // Query string
  const limit: string | undefined = req.query.limit as string;

  // Request body (requires middleware)
  const email: string = req.body.email;

  // Headers
  const token: string | undefined = req.headers.authorization;

  // Cookies (requires cookie middleware)
  const sessionId: string | undefined = req.cookies.sessionId;

  // IP address
  const clientIp: string = req.ip!;

  // Request method
  const method: string = req.method;

  // Request path
  const path: string = req.path;

  res.json({ received: true });
});
```

### Response Object (`res`)

Sending responses to clients.

```typescript
// JSON response
app.get("/api/data", (req: Request, res: Response) => {
  res.json({ data: "value" });
});

// HTML response
app.get("/page", (req: Request, res: Response) => {
  res.send("<h1>Hello</h1>");
});

// File download
app.get("/download", (req: Request, res: Response) => {
  res.download("./file.pdf");
});

// Streaming (for large files)
app.get("/stream", (req: Request, res: Response) => {
  const stream = fs.createReadStream("./large-file.mp4");
  stream.pipe(res);
});

// Status codes
res.status(404).json({ error: "Not found" });
res.sendStatus(204); // No content
res.redirect("/new-location");

// Headers
res.set("X-Custom-Header", "value");
res.type("application/json");

// Cookies
res.cookie("token", "value", {
  httpOnly: true,
  secure: true,
  sameSite: "strict",
  maxAge: 3600000,
});
```

### Middleware Pattern

**Middleware is a function that has access to req, res, and next.**

```typescript
import { Request, Response, NextFunction } from "express";

// Logging middleware
const loggingMiddleware = (
  req: Request,
  res: Response,
  next: NextFunction,
): void => {
  console.log(`${req.method} ${req.path}`);
  next(); // Pass control to next middleware
};

// Authentication middleware
const authMiddleware = (
  req: Request,
  res: Response,
  next: NextFunction,
): void => {
  const token = req.headers.authorization?.split(" ")[1];

  if (!token) {
    res.status(401).json({ error: "Unauthorized" });
    return;
  }

  // Verify token and attach to req
  req.user = { id: "123", role: "admin" };
  next();
};

// Conditional middleware
const adminOnly = (req: Request, res: Response, next: NextFunction): void => {
  if ((req.user as any)?.role !== "admin") {
    res.status(403).json({ error: "Forbidden" });
    return;
  }
  next();
};

// Register middleware
app.use(loggingMiddleware);
app.use("/api/admin", authMiddleware, adminOnly);
```

---

## 4. Routing Architecture

### Basic Routing

```typescript
// GET request
app.get("/users", (req: Request, res: Response) => {
  res.json({ users: [] });
});

// POST request
app.post("/users", (req: Request, res: Response) => {
  res.status(201).json({ created: true });
});

// PUT (full replacement)
app.put("/users/:id", (req: Request, res: Response) => {
  res.json({ updated: true });
});

// PATCH (partial update)
app.patch("/users/:id", (req: Request, res: Response) => {
  res.json({ patched: true });
});

// DELETE
app.delete("/users/:id", (req: Request, res: Response) => {
  res.json({ deleted: true });
});

// Multiple methods
app.all("/webhook", (req: Request, res: Response) => {
  res.json({ received: true });
});
```

### Route Parameters

```typescript
// Single parameter
app.get("/users/:id", (req: Request, res: Response) => {
  const id = req.params.id;
  res.json({ id });
});

// Multiple parameters
app.get("/users/:userId/posts/:postId", (req: Request, res: Response) => {
  const { userId, postId } = req.params;
  res.json({ userId, postId });
});

// Optional parameter (regex)
app.get("/files/:filename?", (req: Request, res: Response) => {
  const filename = req.params.filename || "default.txt";
  res.json({ filename });
});

// Numeric parameter
app.get("/items/:id(\\d+)", (req: Request, res: Response) => {
  const id = parseInt(req.params.id, 10);
  res.json({ id, type: typeof id });
});

// Wildcard
app.get("/api/*", (req: Request, res: Response) => {
  res.json({ path: req.path });
});
```

### Modular Routing (Best Practice)

This is **critical** for scalable applications. Never put all routes in one file.

**`src/routes/users.ts`**

```typescript
import { Router, Request, Response } from "express";

const router = Router();

// GET /api/users
router.get("/", (req: Request, res: Response) => {
  res.json([{ id: 1, name: "Alice" }]);
});

// POST /api/users
router.post("/", (req: Request, res: Response) => {
  res.status(201).json({ id: 2, name: "Bob" });
});

// GET /api/users/:id
router.get("/:id", (req: Request, res: Response) => {
  res.json({ id: req.params.id, name: "Alice" });
});

// PUT /api/users/:id
router.put("/:id", (req: Request, res: Response) => {
  res.json({ id: req.params.id, updated: true });
});

// DELETE /api/users/:id
router.delete("/:id", (req: Request, res: Response) => {
  res.json({ deleted: true });
});

export default router;
```

**`src/routes/products.ts`**

```typescript
import { Router, Request, Response } from "express";

const router = Router();

router.get("/", (req: Request, res: Response) => {
  res.json([{ id: 1, name: "Product A" }]);
});

router.post("/", (req: Request, res: Response) => {
  res.status(201).json({ id: 2 });
});

export default router;
```

**`src/routes/index.ts`**

```typescript
import { Router } from "express";
import usersRouter from "./users";
import productsRouter from "./products";

const router = Router();

router.use("/users", usersRouter);
router.use("/products", productsRouter);

export default router;
```

**`src/index.ts`**

```typescript
import express from "express";
import apiRoutes from "./routes";

const app = express();

app.use(express.json());
app.use("/api", apiRoutes);

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

### Nested Routes (Advanced)

```typescript
// src/routes/posts.ts
import { Router, Request, Response } from "express";

const router = Router({ mergeParams: true });

// GET /api/users/:userId/posts
router.get("/", (req: Request, res: Response) => {
  const userId = req.params.userId;
  res.json([{ id: 1, userId, title: "Post 1" }]);
});

// GET /api/users/:userId/posts/:postId
router.get("/:postId", (req: Request, res: Response) => {
  const { userId, postId } = req.params;
  res.json({ id: postId, userId, title: "Post 1" });
});

// POST /api/users/:userId/posts/:postId/comments
router.post("/:postId/comments", (req: Request, res: Response) => {
  res.status(201).json({ id: 1, text: "Great post!" });
});

export default router;
```

**`src/routes/users.ts` (updated with nested posts)`**

```typescript
import { Router, Request, Response } from "express";
import postsRouter from "./posts";

const router = Router();

router.use("/:userId/posts", postsRouter);

router.get("/", (req: Request, res: Response) => {
  res.json([]);
});

export default router;
```

---

## 5. Middleware Deep Dive

### Built-in Middleware

**JSON Parser** (Express 5 has this built-in)

```typescript
app.use(express.json({ limit: "10mb" }));
app.use(express.urlencoded({ extended: true, limit: "10mb" }));
```

**Static Files**

```typescript
app.use(express.static("public"));
app.use("/static", express.static("public"));
```

**Cookie Parser**

```typescript
npm install cookie-parser
```

```typescript
import cookieParser from "cookie-parser";

app.use(cookieParser("your-secret-key"));

app.get("/", (req: Request, res: Response) => {
  const sessionId = req.cookies.sessionId;
  res.json({ sessionId });
});
```

### Custom Middleware Patterns

**Request Timing Middleware**

```typescript
const timingMiddleware = (
  req: Request,
  res: Response,
  next: NextFunction,
): void => {
  const startTime = Date.now();

  // Capture the original end function
  const originalEnd = res.end;

  res.end = function (...args: any[]) {
    const duration = Date.now() - startTime;
    console.log(`${req.method} ${req.path} - ${duration}ms`);
    originalEnd.apply(res, args);
  };

  next();
};

app.use(timingMiddleware);
```

**Request ID Middleware (Critical for Observability)**

```typescript
import { v4 as uuidv4 } from "uuid";

declare global {
  namespace Express {
    interface Request {
      id?: string;
    }
  }
}

const requestIdMiddleware = (
  req: Request,
  res: Response,
  next: NextFunction,
): void => {
  req.id = (req.headers["x-request-id"] as string) || uuidv4();
  res.setHeader("X-Request-ID", req.id);
  next();
};

app.use(requestIdMiddleware);
```

**Authentication Middleware (Bearer Token)**

```typescript
interface AuthRequest extends Request {
  userId?: string;
  userRole?: string;
}

const authMiddleware = (
  req: AuthRequest,
  res: Response,
  next: NextFunction,
): void => {
  const authHeader = req.headers.authorization;

  if (!authHeader?.startsWith("Bearer ")) {
    res.status(401).json({ error: "Missing authorization header" });
    return;
  }

  const token = authHeader.substring(7);

  try {
    const decoded = verifyJWT(token);
    req.userId = decoded.id;
    req.userRole = decoded.role;
    next();
  } catch (error) {
    res.status(401).json({ error: "Invalid token" });
  }
};

app.use("/api/protected", authMiddleware);
```

**CORS Middleware**

```typescript
npm install cors
```

```typescript
import cors from "cors";

// Simple CORS (allow all)
app.use(cors());

// Strict CORS (production)
app.use(
  cors({
    origin: process.env.ALLOWED_ORIGINS?.split(",") || [
      "http://localhost:3000",
    ],
    credentials: true,
    methods: ["GET", "POST", "PUT", "DELETE", "PATCH"],
    allowedHeaders: ["Content-Type", "Authorization"],
    maxAge: 86400,
  }),
);
```

**Validation Middleware Factory**

```typescript
import { ValidationChain, validationResult } from "express-validator";

const validate = (validations: ValidationChain[]) => {
  return async (req: Request, res: Response, next: NextFunction) => {
    await Promise.all(validations.map((v) => v.run(req)));
    const errors = validationResult(req);

    if (!errors.isEmpty()) {
      res.status(400).json({ errors: errors.array() });
      return;
    }

    next();
  };
};

// Usage
app.post(
  "/users",
  validate([body("email").isEmail(), body("password").isLength({ min: 8 })]),
  (req: Request, res: Response) => {
    res.json({ created: true });
  },
);
```

### Error-Handling Middleware

**Critical: Must be last in middleware chain**

```typescript
interface CustomError extends Error {
  status?: number;
  code?: string;
}

// Global error handler
const errorHandler = (
  err: CustomError,
  req: Request,
  res: Response,
  next: NextFunction,
): void => {
  const status = err.status || 500;
  const message = err.message || "Internal Server Error";
  const code = err.code || "INTERNAL_ERROR";

  // Log error (critical for debugging)
  console.error({
    requestId: (req as any).id,
    status,
    code,
    message,
    stack: err.stack,
  });

  res.status(status).json({
    error: {
      code,
      message,
      requestId: (req as any).id,
      ...(process.env.NODE_ENV === "development" && { stack: err.stack }),
    },
  });
};

// 404 handler (before error handler)
app.use((req: Request, res: Response) => {
  res.status(404).json({
    error: {
      code: "NOT_FOUND",
      message: `Route ${req.method} ${req.path} not found`,
    },
  });
});

// Register last
app.use(errorHandler);
```

---

## 6. Request & Response Handling

### Parsing Different Content Types

```typescript
// JSON
app.use(express.json());

// URL-encoded form
app.use(express.urlencoded({ extended: true }));

// Raw bytes
app.use(express.raw({ type: "application/octet-stream" }));

// Text
app.use(express.text({ type: "text/plain" }));
```

### Type-Safe Request Body

```typescript
interface CreateUserRequest {
  email: string;
  name: string;
  age: number;
}

app.post("/users", (req: Request<{}, {}, CreateUserRequest>, res: Response) => {
  const { email, name, age } = req.body;
  // TypeScript knows these types
  console.log(email, name, age);
  res.status(201).json({ id: 1 });
});
```

### Query Parameters

```typescript
interface PaginationQuery {
  page?: string;
  limit?: string;
  sort?: string;
}

app.get(
  "/users",
  (req: Request<{}, {}, {}, PaginationQuery>, res: Response) => {
    const page = parseInt(req.query.page || "1", 10);
    const limit = parseInt(req.query.limit || "20", 10);
    const sort = req.query.sort || "createdAt";

    const offset = (page - 1) * limit;

    res.json({
      data: [],
      pagination: { page, limit, offset },
    });
  },
);
```

### Streaming Large Responses

```typescript
import fs from "fs";

// Stream CSV file
app.get("/export/users", (req: Request, res: Response) => {
  res.setHeader("Content-Type", "text/csv");
  res.setHeader("Content-Disposition", 'attachment; filename="users.csv"');

  const stream = fs.createReadStream("./users.csv");
  stream.pipe(res);

  stream.on("error", (err) => {
    console.error("Stream error", err);
    res.status(500).json({ error: "Failed to export" });
  });
});

// Chunked JSON response
app.get("/stream/data", (req: Request, res: Response) => {
  res.setHeader("Content-Type", "application/x-ndjson"); // newline-delimited JSON

  const items = [{ id: 1 }, { id: 2 }, { id: 3 }];

  items.forEach((item) => {
    res.write(JSON.stringify(item) + "\n");
  });

  res.end();
});
```

### Request Validation Pattern

```typescript
import { body, param, query, validationResult } from "express-validator";

const handleValidationErrors = (
  req: Request,
  res: Response,
  next: NextFunction,
) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    res.status(400).json({ errors: errors.array() });
    return;
  }
  next();
};

app.post(
  "/users",
  [
    body("email").isEmail().normalizeEmail(),
    body("password").isLength({ min: 8 }),
    body("name").trim().notEmpty(),
    body("age").isInt({ min: 18, max: 120 }),
  ],
  handleValidationErrors,
  (req: Request, res: Response) => {
    res.status(201).json({ created: true });
  },
);

app.get(
  "/users/:id",
  [param("id").isInt()],
  handleValidationErrors,
  (req: Request, res: Response) => {
    res.json({ user: {} });
  },
);
```

---

## 7. Environment Configuration

### Configuration Management

**`src/config.ts`**

```typescript
import dotenv from "dotenv";

dotenv.config({
  path: process.env.NODE_ENV === "production" ? ".env.production" : ".env",
});

interface Config {
  app: {
    env: string;
    port: number;
    host: string;
  };
  database: {
    url: string;
    maxPoolSize: number;
    minPoolSize: number;
  };
  jwt: {
    secret: string;
    expiresIn: string;
  };
  logging: {
    level: string;
    format: string;
  };
  cors: {
    origins: string[];
  };
  security: {
    bcryptRounds: number;
    rateLimitWindowMs: number;
    rateLimitMaxRequests: number;
  };
}

function getConfig(): Config {
  const requiredVars = ["DATABASE_URL", "JWT_SECRET"];

  for (const variable of requiredVars) {
    if (!process.env[variable]) {
      throw new Error(`Missing required environment variable: ${variable}`);
    }
  }

  return {
    app: {
      env: process.env.NODE_ENV || "development",
      port: parseInt(process.env.PORT || "3000", 10),
      host: process.env.HOST || "0.0.0.0",
    },
    database: {
      url: process.env.DATABASE_URL!,
      maxPoolSize: parseInt(process.env.DB_MAX_POOL_SIZE || "20", 10),
      minPoolSize: parseInt(process.env.DB_MIN_POOL_SIZE || "5", 10),
    },
    jwt: {
      secret: process.env.JWT_SECRET!,
      expiresIn: process.env.JWT_EXPIRES_IN || "24h",
    },
    logging: {
      level: process.env.LOG_LEVEL || "info",
      format: process.env.LOG_FORMAT || "json",
    },
    cors: {
      origins: (process.env.ALLOWED_ORIGINS || "http://localhost:3000")
        .split(",")
        .map((o) => o.trim()),
    },
    security: {
      bcryptRounds: parseInt(process.env.BCRYPT_ROUNDS || "12", 10),
      rateLimitWindowMs: parseInt(
        process.env.RATE_LIMIT_WINDOW_MS || "900000",
        10,
      ),
      rateLimitMaxRequests: parseInt(
        process.env.RATE_LIMIT_MAX_REQUESTS || "100",
        10,
      ),
    },
  };
}

export const config = getConfig();
```

### Environment Files

**`.env`**

```bash
NODE_ENV=development
PORT=3000
HOST=0.0.0.0

DATABASE_URL=postgresql://user:password@localhost:5432/express_db
DB_MAX_POOL_SIZE=20
DB_MIN_POOL_SIZE=5

JWT_SECRET=your-super-secret-jwt-key-change-in-production
JWT_EXPIRES_IN=24h

LOG_LEVEL=debug
LOG_FORMAT=json

ALLOWED_ORIGINS=http://localhost:3000,http://localhost:3001

BCRYPT_ROUNDS=12
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100
```

**`.env.production`**

```bash
NODE_ENV=production
PORT=3000
HOST=0.0.0.0

DATABASE_URL=${DATABASE_URL}
DB_MAX_POOL_SIZE=50
DB_MIN_POOL_SIZE=10

JWT_SECRET=${JWT_SECRET}
JWT_EXPIRES_IN=24h

LOG_LEVEL=error
LOG_FORMAT=json

ALLOWED_ORIGINS=https://api.example.com,https://example.com

BCRYPT_ROUNDS=12
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=1000
```

---

## 8. MVC Architecture

### Model Layer

**`src/models/User.ts`**

```typescript
import prisma from "./prismaClient";

export interface User {
  id: string;
  email: string;
  name: string;
  passwordHash: string;
  createdAt: Date;
  updatedAt: Date;
}

export class UserModel {
  static async create(data: {
    email: string;
    name: string;
    passwordHash: string;
  }): Promise<User> {
    return prisma.user.create({
      data,
    });
  }

  static async findByEmail(email: string): Promise<User | null> {
    return prisma.user.findUnique({
      where: { email },
    });
  }

  static async findById(id: string): Promise<User | null> {
    return prisma.user.findUnique({
      where: { id },
    });
  }

  static async update(id: string, data: Partial<User>): Promise<User> {
    return prisma.user.update({
      where: { id },
      data,
    });
  }

  static async delete(id: string): Promise<void> {
    await prisma.user.delete({
      where: { id },
    });
  }

  static async findAll(skip: number = 0, take: number = 20): Promise<User[]> {
    return prisma.user.findMany({
      skip,
      take,
      orderBy: { createdAt: "desc" },
    });
  }
}
```

### Service Layer

**`src/services/UserService.ts`**

```typescript
import bcrypt from "bcrypt";
import jwt from "jsonwebtoken";
import { UserModel } from "../models/User";
import { config } from "../config";

export class UserService {
  static async registerUser(email: string, name: string, password: string) {
    // Check if user exists
    const existing = await UserModel.findByEmail(email);
    if (existing) {
      throw new Error("User already exists");
    }

    // Hash password
    const passwordHash = await bcrypt.hash(
      password,
      config.security.bcryptRounds,
    );

    // Create user
    const user = await UserModel.create({
      email,
      name,
      passwordHash,
    });

    return this.sanitizeUser(user);
  }

  static async login(email: string, password: string) {
    const user = await UserModel.findByEmail(email);

    if (!user) {
      throw new Error("Invalid credentials");
    }

    const isValid = await bcrypt.compare(password, user.passwordHash);

    if (!isValid) {
      throw new Error("Invalid credentials");
    }

    const token = jwt.sign(
      { id: user.id, email: user.email },
      config.jwt.secret,
      { expiresIn: config.jwt.expiresIn },
    );

    return {
      user: this.sanitizeUser(user),
      token,
    };
  }

  static async getUserById(id: string) {
    const user = await UserModel.findById(id);

    if (!user) {
      throw new Error("User not found");
    }

    return this.sanitizeUser(user);
  }

  static async updateUser(id: string, data: Partial<any>) {
    const user = await UserModel.update(id, data);
    return this.sanitizeUser(user);
  }

  static async deleteUser(id: string) {
    await UserModel.delete(id);
  }

  private static sanitizeUser(user: any) {
    const { passwordHash, ...sanitized } = user;
    return sanitized;
  }
}
```

### Controller Layer

**`src/controllers/UserController.ts`**

```typescript
import { Request, Response, NextFunction } from "express";
import { UserService } from "../services/UserService";

export class UserController {
  static async register(req: Request, res: Response, next: NextFunction) {
    try {
      const { email, name, password } = req.body;
      const user = await UserService.registerUser(email, name, password);
      res.status(201).json(user);
    } catch (error) {
      next(error);
    }
  }

  static async login(req: Request, res: Response, next: NextFunction) {
    try {
      const { email, password } = req.body;
      const result = await UserService.login(email, password);
      res.json(result);
    } catch (error) {
      next(error);
    }
  }

  static async getUser(req: Request, res: Response, next: NextFunction) {
    try {
      const { id } = req.params;
      const user = await UserService.getUserById(id);
      res.json(user);
    } catch (error) {
      next(error);
    }
  }

  static async updateUser(req: Request, res: Response, next: NextFunction) {
    try {
      const { id } = req.params;
      const user = await UserService.updateUser(id, req.body);
      res.json(user);
    } catch (error) {
      next(error);
    }
  }

  static async deleteUser(req: Request, res: Response, next: NextFunction) {
    try {
      const { id } = req.params;
      await UserService.deleteUser(id);
      res.status(204).send();
    } catch (error) {
      next(error);
    }
  }
}
```

### Routes Integration

**`src/routes/auth.ts`**

```typescript
import { Router } from "express";
import { UserController } from "../controllers/UserController";

const router = Router();

router.post("/register", UserController.register);
router.post("/login", UserController.login);

export default router;
```

---

## 9. Production Folder Structure

```
express-api-prod/
├── src/
│   ├── config/
│   │   ├── database.ts
│   │   ├── logger.ts
│   │   └── index.ts
│   ├── controllers/
│   │   ├── UserController.ts
│   │   ├── ProductController.ts
│   │   └── index.ts
│   ├── services/
│   │   ├── UserService.ts
│   │   ├── ProductService.ts
│   │   ├── EmailService.ts
│   │   └── index.ts
│   ├── models/
│   │   ├── User.ts
│   │   ├── Product.ts
│   │   ├── prismaClient.ts
│   │   └── index.ts
│   ├── middleware/
│   │   ├── auth.ts
│   │   ├── errorHandler.ts
│   │   ├── validation.ts
│   │   ├── requestId.ts
│   │   ├── cors.ts
│   │   └── index.ts
│   ├── routes/
│   │   ├── auth.ts
│   │   ├── users.ts
│   │   ├── products.ts
│   │   └── index.ts
│   ├── utils/
│   │   ├── errors.ts
│   │   ├── validators.ts
│   │   ├── jwt.ts
│   │   └── helpers.ts
│   ├── types/
│   │   ├── express.d.ts
│   │   ├── index.ts
│   │   └── api.ts
│   ├── index.ts
│   └── app.ts
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
├── dist/
├── .env
├── .env.production
├── .env.test
├── package.json
├── tsconfig.json
├── nodemon.json
├── docker-compose.yml
├── Dockerfile
├── .dockerignore
├── .gitignore
└── README.md
```

### Type Definitions

**`src/types/express.d.ts`**

```typescript
declare global {
  namespace Express {
    interface Request {
      id?: string;
      userId?: string;
      user?: {
        id: string;
        email: string;
        role: string;
      };
    }
  }
}
```

### App Factory

**`src/app.ts`**

```typescript
import express, { Express } from "express";
import cors from "cors";
import helmet from "helmet";
import compression from "compression";

import { config } from "./config";
import { errorHandler } from "./middleware/errorHandler";
import { requestIdMiddleware } from "./middleware/requestId";
import { corsMiddleware } from "./middleware/cors";
import apiRoutes from "./routes";

export function createApp(): Express {
  const app = express();

  // Trust proxy in production
  if (config.app.env === "production") {
    app.set("trust proxy", 1);
  }

  // Security
  app.use(helmet());

  // Compression
  app.use(compression());

  // CORS
  app.use(corsMiddleware);

  // Parsing
  app.use(express.json({ limit: "10mb" }));
  app.use(express.urlencoded({ extended: true, limit: "10mb" }));

  // Request metadata
  app.use(requestIdMiddleware);

  // Routes
  app.use("/api", apiRoutes);

  // 404 handler
  app.use((req, res) => {
    res.status(404).json({
      error: {
        code: "NOT_FOUND",
        message: `Route ${req.method} ${req.path} not found`,
      },
    });
  });

  // Error handler (must be last)
  app.use(errorHandler);

  return app;
}

export default createApp;
```

**`src/index.ts`**

```typescript
import createApp from "./app";
import { config } from "./config";
import logger from "./config/logger";

const app = createApp();

const server = app.listen(config.app.port, config.app.host, () => {
  logger.info(`Server running on ${config.app.host}:${config.app.port}`);
});

// Graceful shutdown
process.on("SIGTERM", () => {
  logger.info("SIGTERM received, shutting down gracefully");
  server.close(() => {
    logger.info("Server closed");
    process.exit(0);
  });
});

process.on("SIGINT", () => {
  logger.info("SIGINT received, shutting down gracefully");
  server.close(() => {
    logger.info("Server closed");
    process.exit(0);
  });
});

process.on("uncaughtException", (error) => {
  logger.error("Uncaught Exception", error);
  process.exit(1);
});

process.on("unhandledRejection", (reason) => {
  logger.error("Unhandled Rejection", reason);
  process.exit(1);
});
```

---

## 10. Validation Strategy

### Zod Validation (Recommended)

```bash
npm install zod
```

**`src/validators/user.ts`**

```typescript
import { z } from "zod";

export const registerSchema = z.object({
  email: z.string().email("Invalid email format"),
  name: z.string().min(2).max(100),
  password: z.string().min(8).max(128),
});

export const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

export const updateUserSchema = z.object({
  name: z.string().min(2).max(100).optional(),
  email: z.string().email().optional(),
});

export const userIdParamSchema = z.object({
  id: z.string().uuid(),
});

export type RegisterInput = z.infer<typeof registerSchema>;
export type LoginInput = z.infer<typeof loginSchema>;
export type UpdateUserInput = z.infer<typeof updateUserSchema>;
```

### Validation Middleware

**`src/middleware/validation.ts`**

```typescript
import { Request, Response, NextFunction } from "express";
import { ZodSchema, ZodError } from "zod";

export const validateRequest = (schema: ZodSchema) => {
  return async (req: Request, res: Response, next: NextFunction) => {
    try {
      const validated = await schema.parseAsync({
        body: req.body,
        query: req.query,
        params: req.params,
      });

      req.body = validated.body;
      req.query = validated.query;
      req.params = validated.params;

      next();
    } catch (error) {
      if (error instanceof ZodError) {
        res.status(400).json({
          error: {
            code: "VALIDATION_ERROR",
            messages: error.errors.map((e) => ({
              field: e.path.join("."),
              message: e.message,
            })),
          },
        });
        return;
      }

      next(error);
    }
  };
};
```

### Using Validation in Routes

```typescript
import { registerSchema, loginSchema } from "../validators/user";
import { validateRequest } from "../middleware/validation";
import { UserController } from "../controllers/UserController";

const router = Router();

router.post(
  "/register",
  validateRequest(registerSchema),
  UserController.register,
);

router.post("/login", validateRequest(loginSchema), UserController.login);

export default router;
```

### Express-Validator Alternative

```typescript
npm install express-validator
```

```typescript
import { body, param, query, validationResult } from "express-validator";

app.post(
  "/users",
  body("email").isEmail().normalizeEmail().withMessage("Invalid email"),
  body("password")
    .isLength({ min: 8 })
    .withMessage("Password must be at least 8 characters"),
  body("age").isInt({ min: 18 }).withMessage("Age must be at least 18"),
  (req: Request, res: Response) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      res.status(400).json({ errors: errors.array() });
      return;
    }

    res.json({ created: true });
  },
);
```

---

## 11. Authentication & Authorization

### JWT Authentication

**`src/utils/jwt.ts`**

```typescript
import jwt, { SignOptions, VerifyOptions } from "jsonwebtoken";
import { config } from "../config";

export interface JWTPayload {
  id: string;
  email: string;
  role: string;
}

export class JWTService {
  static sign(payload: Partial<JWTPayload>): string {
    const options: SignOptions = {
      expiresIn: config.jwt.expiresIn,
      algorithm: "HS256",
    };

    return jwt.sign(payload, config.jwt.secret, options);
  }

  static verify(token: string): JWTPayload {
    const options: VerifyOptions = {
      algorithms: ["HS256"],
    };

    return jwt.verify(token, config.jwt.secret, options) as JWTPayload;
  }

  static decode(token: string): JWTPayload | null {
    return jwt.decode(token) as JWTPayload | null;
  }

  static isExpired(token: string): boolean {
    try {
      this.verify(token);
      return false;
    } catch (error) {
      return true;
    }
  }
}
```

### Authentication Middleware

**`src/middleware/auth.ts`**

```typescript
import { Request, Response, NextFunction } from "express";
import { JWTService } from "../utils/jwt";

export interface AuthRequest extends Request {
  user?: {
    id: string;
    email: string;
    role: string;
  };
}

export const authMiddleware = (
  req: AuthRequest,
  res: Response,
  next: NextFunction,
): void => {
  const authHeader = req.headers.authorization;

  if (!authHeader?.startsWith("Bearer ")) {
    res.status(401).json({
      error: {
        code: "UNAUTHORIZED",
        message: "Missing or invalid authorization header",
      },
    });
    return;
  }

  const token = authHeader.substring(7);

  try {
    const payload = JWTService.verify(token);
    req.user = payload;
    next();
  } catch (error) {
    res.status(401).json({
      error: {
        code: "UNAUTHORIZED",
        message: "Invalid or expired token",
      },
    });
  }
};

// Authorization middleware
export const requireRole = (...roles: string[]) => {
  return (req: AuthRequest, res: Response, next: NextFunction): void => {
    if (!req.user || !roles.includes(req.user.role)) {
      res.status(403).json({
        error: {
          code: "FORBIDDEN",
          message: "Insufficient permissions",
        },
      });
      return;
    }

    next();
  };
};

export const requireAdmin = requireRole("admin");
export const requireModerator = requireRole("admin", "moderator");
```

### Login Flow

```typescript
import { Router } from "express";
import { UserController } from "../controllers/UserController";

const router = Router();

// Public routes
router.post("/register", UserController.register);
router.post("/login", UserController.login);
router.post("/refresh", UserController.refreshToken);

// Protected routes
router.post("/logout", authMiddleware, UserController.logout);
router.get("/me", authMiddleware, UserController.getCurrentUser);

export default router;
```

### OAuth Conceptual Pattern

```typescript
/**
 * OAuth 2.0 flow (conceptual - not fully implemented)
 *
 * 1. User clicks "Login with Google"
 * 2. Frontend redirects to: GET /auth/google
 * 3. Server redirects to Google OAuth endpoint
 * 4. User grants permission
 * 5. Google redirects back to: GET /auth/google/callback
 * 6. Server exchanges code for access_token
 * 7. Server fetches user profile
 * 8. Server creates or updates user in DB
 * 9. Server creates JWT and sends to frontend
 */

npm install passport passport-google-oauth20 passport-jwt

// Implementation would use passport strategies:
// - Set up passport strategies
// - Route: GET /auth/google -> passport.authenticate('google')
// - Route: GET /auth/google/callback -> exchange auth code
// - Generate JWT for user
```

---

## 12. Error Handling

### Custom Error Classes

**`src/utils/errors.ts`**

```typescript
export class AppError extends Error {
  constructor(
    public message: string,
    public status: number = 500,
    public code: string = "INTERNAL_ERROR",
  ) {
    super(message);
    Object.setPrototypeOf(this, AppError.prototype);
  }
}

export class ValidationError extends AppError {
  constructor(
    message: string,
    public fields?: Record<string, string>,
  ) {
    super(message, 400, "VALIDATION_ERROR");
    Object.setPrototypeOf(this, ValidationError.prototype);
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, 404, "NOT_FOUND");
    Object.setPrototypeOf(this, NotFoundError.prototype);
  }
}

export class UnauthorizedError extends AppError {
  constructor(message: string = "Unauthorized") {
    super(message, 401, "UNAUTHORIZED");
    Object.setPrototypeOf(this, UnauthorizedError.prototype);
  }
}

export class ForbiddenError extends AppError {
  constructor(message: string = "Forbidden") {
    super(message, 403, "FORBIDDEN");
    Object.setPrototypeOf(this, ForbiddenError.prototype);
  }
}

export class ConflictError extends AppError {
  constructor(message: string) {
    super(message, 409, "CONFLICT");
    Object.setPrototypeOf(this, ConflictError.prototype);
  }
}

export class InternalError extends AppError {
  constructor(message: string = "Internal server error") {
    super(message, 500, "INTERNAL_ERROR");
    Object.setPrototypeOf(this, InternalError.prototype);
  }
}
```

### Centralized Error Handler

**`src/middleware/errorHandler.ts`**

```typescript
import { Request, Response, NextFunction } from "express";
import { AppError } from "../utils/errors";
import logger from "../config/logger";

export const errorHandler = (
  err: Error | AppError,
  req: Request,
  res: Response,
  next: NextFunction,
): void => {
  // Request ID for debugging
  const requestId = (req as any).id || "unknown";

  // Determine error details
  let status = 500;
  let code = "INTERNAL_ERROR";
  let message = "Internal server error";

  if (err instanceof AppError) {
    status = err.status;
    code = err.code;
    message = err.message;
  }

  // Log error
  if (status >= 500) {
    logger.error("Unhandled error", {
      requestId,
      status,
      code,
      message,
      stack: err.stack,
      url: req.url,
      method: req.method,
    });
  } else {
    logger.warn("Client error", {
      requestId,
      status,
      code,
      message,
      url: req.url,
      method: req.method,
    });
  }

  // Send response
  res.status(status).json({
    error: {
      code,
      message,
      requestId,
      ...(process.env.NODE_ENV === "development" && { stack: err.stack }),
    },
  });
};
```

### Async Error Wrapper

**`src/utils/asyncHandler.ts`**

```typescript
import { Request, Response, NextFunction } from "express";

// Wrap async route handlers to catch errors
export const asyncHandler = (
  fn: (req: Request, res: Response, next: NextFunction) => Promise<any>,
) => {
  return (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};

// Usage in controllers
import { asyncHandler } from "../utils/asyncHandler";

export class UserController {
  static getUser = asyncHandler(async (req: Request, res: Response) => {
    const user = await UserService.getUserById(req.params.id);
    res.json(user);
  });
}
```

### Error Usage in Services

```typescript
import { NotFoundError, ValidationError } from "../utils/errors";

export class UserService {
  static async getUserById(id: string) {
    const user = await UserModel.findById(id);

    if (!user) {
      throw new NotFoundError("User");
    }

    return user;
  }

  static async registerUser(email: string, password: string) {
    const existing = await UserModel.findByEmail(email);

    if (existing) {
      throw new ValidationError("Email already registered");
    }

    // ... rest of registration
  }
}
```

---

## 13. Logging & Observability

### Winston Logger Setup

```bash
npm install winston
```

**`src/config/logger.ts`**

```typescript
import winston from "winston";
import { config } from "./index";

const levels = {
  error: 0,
  warn: 1,
  info: 2,
  debug: 3,
};

const colors = {
  error: "red",
  warn: "yellow",
  info: "green",
  debug: "blue",
};

winston.addColors(colors);

const formatDate = () => new Date().toISOString();

const format = winston.format.combine(
  winston.format.timestamp({ format: "YYYY-MM-DD HH:mm:ss" }),
  winston.format.errors({ stack: true }),
  winston.format.splat(),
  config.app.env === "production"
    ? winston.format.json()
    : winston.format.combine(
        winston.format.colorize({ all: true }),
        winston.format.printf(
          (info) => `${info.timestamp} [${info.level}]: ${info.message}`,
        ),
      ),
);

const transports = [
  new winston.transports.Console(),
  new winston.transports.File({
    filename: "logs/error.log",
    level: "error",
  }),
  new winston.transports.File({
    filename: "logs/combined.log",
  }),
];

const logger = winston.createLogger({
  level: config.logging.level,
  levels,
  format,
  transports,
});

export default logger;
```

### Request Logging Middleware

```typescript
import { Request, Response, NextFunction } from "express";
import logger from "../config/logger";

export const requestLoggerMiddleware = (
  req: Request,
  res: Response,
  next: NextFunction,
): void => {
  const startTime = Date.now();

  res.on("finish", () => {
    const duration = Date.now() - startTime;
    const level = res.statusCode >= 400 ? "warn" : "info";

    logger[level as "warn" | "info"]("HTTP Request", {
      requestId: (req as any).id,
      method: req.method,
      path: req.path,
      query: req.query,
      status: res.statusCode,
      duration: `${duration}ms`,
      contentLength: res.get("content-length") || "unknown",
      userAgent: req.get("user-agent"),
    });
  });

  next();
};

// Register in app
app.use(requestLoggerMiddleware);
```

### Structured Logging

```typescript
// Good: Structured logging
logger.info("User login", {
  userId: user.id,
  email: user.email,
  timestamp: new Date(),
  requestId: req.id,
});

// Bad: Unstructured
console.log("User logged in: " + user.email);

// For searches/filtering in production
logger.error("Database connection failed", {
  code: "DB_CONNECTION_ERROR",
  host: config.database.url,
  retries: 3,
  nextRetry: new Date(Date.now() + 5000),
});
```

---

## 14. Security Best Practices

### Helmet Security Headers

```bash
npm install helmet
```

```typescript
import helmet from "helmet";

app.use(helmet());

// Customized for API
app.use(
  helmet({
    contentSecurityPolicy: false, // For APIs
    hsts: { maxAge: 31536000, includeSubDomains: true, preload: true },
    frameguard: { action: "deny" },
    noSniff: true,
    xssFilter: true,
  }),
);
```

### CORS Best Practices

```typescript
npm install cors
```

```typescript
import cors from "cors";

app.use(
  cors({
    origin: process.env.ALLOWED_ORIGINS?.split(","),
    credentials: true,
    methods: ["GET", "POST", "PUT", "DELETE", "PATCH"],
    allowedHeaders: ["Content-Type", "Authorization"],
    exposedHeaders: ["X-Total-Count", "X-Page-Number"],
    maxAge: 86400,
    optionsSuccessStatus: 200,
  }),
);
```

### Rate Limiting

```bash
npm install express-rate-limit
```

**`src/middleware/rateLimiter.ts`**

```typescript
import rateLimit from "express-rate-limit";
import { config } from "../config";

// Global rate limiter
export const globalLimiter = rateLimit({
  windowMs: config.security.rateLimitWindowMs,
  max: config.security.rateLimitMaxRequests,
  message: "Too many requests from this IP, please try again later",
  standardHeaders: true,
  legacyHeaders: false,
});

// Strict rate limiter for sensitive endpoints
export const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts
  skipSuccessfulRequests: true, // Don't count successful logins
  message: "Too many login attempts, please try again after 15 minutes",
});

// API route limiter
export const apiLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 100,
});
```

**Usage:**

```typescript
app.use(globalLimiter);
app.post("/auth/login", authLimiter, UserController.login);
app.use("/api/", apiLimiter);
```

### Input Sanitization

```bash
npm install xss
```

```typescript
import xss from "xss";

const sanitizeUserInput = (input: string): string => {
  return xss(input, {
    whiteList: {},
    stripIgnoredTag: true,
  });
};

// In validation
export const sanitizeSchema = z.object({
  name: z.string().transform(sanitizeUserInput),
  bio: z
    .string()
    .optional()
    .transform((v) => (v ? sanitizeUserInput(v) : v)),
});
```

### SQL Injection Prevention

Always use parameterized queries (Prisma does this automatically):

```typescript
// ✅ Safe: Prisma handles escaping
const user = await prisma.user.findUnique({
  where: { email: userInput },
});

// ❌ Dangerous: SQL concatenation
const user = await db.query(`SELECT * FROM users WHERE email = '${userInput}'`);
```

### Environment Variable Protection

```typescript
// ❌ Never log sensitive data
console.log(config.jwt.secret); // DANGER!
logger.info("Auth successful", { token }); // DANGER!

// ✅ Sanitize logs
logger.info("Auth successful", {
  userId: user.id,
  // Don't include token or secrets
});
```

### HTTPS Enforcement (In Production)

```typescript
if (process.env.NODE_ENV === "production") {
  app.use((req, res, next) => {
    if (req.header("x-forwarded-proto") !== "https") {
      res.redirect(`https://${req.header("host")}${req.url}`);
    } else {
      next();
    }
  });
}
```

---

## 15. Database Integration

### Prisma Setup

```bash
npm install @prisma/client
npm install -D prisma
npx prisma init
```

**`prisma/schema.prisma`**

```prisma
// Schema configuration
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

// Models
model User {
  id        String     @id @default(cuid())
  email     String     @unique
  name      String
  passwordHash String
  role      String     @default("user")
  createdAt DateTime   @default(now())
  updatedAt DateTime   @updatedAt
  posts     Post[]
}

model Post {
  id        String     @id @default(cuid())
  title     String
  content   String     @db.Text
  published Boolean    @default(false)
  authorId  String
  author    User       @relation(fields: [authorId], references: [id], onDelete: Cascade)
  createdAt DateTime   @default(now())
  updatedAt DateTime   @updatedAt

  @@index([authorId])
}
```

### Database Client

**`src/models/prismaClient.ts`**

```typescript
import { PrismaClient } from "@prisma/client";
import logger from "../config/logger";

const prisma = new PrismaClient({
  log: [
    { emit: "event", level: "query" },
    { emit: "event", level: "error" },
    { emit: "event", level: "warn" },
  ],
});

// Log queries in development
if (process.env.NODE_ENV === "development") {
  prisma.$on("query", (e) => {
    logger.debug("Database Query", {
      query: e.query,
      params: e.params,
      duration: `${e.duration}ms`,
    });
  });
}

// Error handling
prisma.$on("error", (e) => {
  logger.error("Database Error", {
    code: e.code,
    message: e.message,
  });
});

// Graceful shutdown
process.on("SIGTERM", async () => {
  await prisma.$disconnect();
});

export default prisma;
```

### Repository Pattern (Optional but Recommended)

**`src/repositories/UserRepository.ts`**

```typescript
import prisma from "../models/prismaClient";
import { User } from "@prisma/client";

export class UserRepository {
  async create(data: {
    email: string;
    name: string;
    passwordHash: string;
  }): Promise<User> {
    return prisma.user.create({ data });
  }

  async findById(id: string): Promise<User | null> {
    return prisma.user.findUnique({ where: { id } });
  }

  async findByEmail(email: string): Promise<User | null> {
    return prisma.user.findUnique({ where: { email } });
  }

  async findAll(skip: number = 0, take: number = 20): Promise<User[]> {
    return prisma.user.findMany({ skip, take, orderBy: { createdAt: "desc" } });
  }

  async update(id: string, data: Partial<User>): Promise<User> {
    return prisma.user.update({
      where: { id },
      data,
    });
  }

  async delete(id: string): Promise<void> {
    await prisma.user.delete({ where: { id } });
  }

  async count(): Promise<number> {
    return prisma.user.count();
  }
}
```

### Transactions

```typescript
// Atomic operations
const result = await prisma.$transaction(async (tx) => {
  const user = await tx.user.create({
    data: { email: "user@example.com", name: "User", passwordHash: "..." },
  });

  const post = await tx.post.create({
    data: {
      title: "First Post",
      content: "Content",
      authorId: user.id,
    },
  });

  return { user, post };
});
```

---

## 16. Performance Optimization

### Response Compression

```typescript
import compression from "compression";

app.use(
  compression({
    level: 6, // 0-9 (6 is default)
    threshold: 1024, // Only compress if > 1KB
    filter: (req, res) => {
      if (req.headers["x-no-compression"]) {
        return false;
      }
      return compression.filter(req, res);
    },
  }),
);
```

### Caching Headers

```typescript
// Cache static assets
app.use(
  express.static("public", {
    maxAge: "1d",
    etag: false,
  }),
);

// Cache API responses (use with caution)
const cacheMiddleware = (duration: number) => {
  return (req: Request, res: Response, next: NextFunction) => {
    let key = "__express__" + req.originalUrl || req.url;
    let cachedBody = app.get(key);

    if (cachedBody) {
      res.send(cachedBody);
      return;
    }

    res.sendResponse = res.send;
    res.send = function (body: any) {
      app.set(key, body);
      setTimeout(() => app.delete(key), duration);
      res.sendResponse(body);
    };

    next();
  };
};

// Usage
app.get("/api/public-data", cacheMiddleware(60000), (req, res) => {
  res.json({ data: "expensive computation" });
});
```

### Query Optimization

```typescript
// ❌ N+1 problem
const users = await prisma.user.findMany();
for (const user of users) {
  const posts = await prisma.post.findMany({ where: { authorId: user.id } });
  user.posts = posts;
}

// ✅ Use includes
const users = await prisma.user.findMany({
  include: { posts: true },
});

// ✅ Pagination
const users = await prisma.user.findMany({
  skip: (page - 1) * limit,
  take: limit,
});

// ✅ Select specific fields
const users = await prisma.user.findMany({
  select: { id: true, email: true, name: true },
});
```

### Connection Pooling (Prisma)

```typescript
// DATABASE_URL="postgresql://user:password@localhost:5432/db?schema=public&connection_limit=20"
// Prisma automatically handles this
```

### Clustering & Load Balancing

```typescript
import cluster from "cluster";
import os from "os";
import app from "./app";

if (cluster.isPrimary) {
  const numCPUs = os.cpus().length;

  console.log(`Master process ${process.pid} is running`);

  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on("exit", (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork(); // Restart
  });
} else {
  const PORT = process.env.PORT || 3000;
  app.listen(PORT, () => {
    console.log(`Worker ${process.pid} listening on port ${PORT}`);
  });
}

// Run with: node --trace-warnings dist/index.js
```

### Memory Monitoring

```typescript
setInterval(() => {
  const memUsage = process.memoryUsage();
  console.log("Memory Usage:", {
    rss: `${Math.round(memUsage.rss / 1024 / 1024)}MB`,
    heapTotal: `${Math.round(memUsage.heapTotal / 1024 / 1024)}MB`,
    heapUsed: `${Math.round(memUsage.heapUsed / 1024 / 1024)}MB`,
  });
}, 60000); // Every minute
```

---

## 17. Testing Strategy

### Setup Jest

```bash
npm install -D jest ts-jest @types/jest supertest @types/supertest
```

**`jest.config.js`**

```javascript
module.exports = {
  preset: "ts-jest",
  testEnvironment: "node",
  roots: ["<rootDir>/tests"],
  testMatch: ["**/__tests__/**/*.ts", "**/?(*.)+(spec|test).ts"],
  collectCoverageFrom: ["src/**/*.ts", "!src/**/*.d.ts", "!src/index.ts"],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70,
    },
  },
};
```

### Unit Tests

**`tests/unit/services/UserService.test.ts`**

```typescript
import { UserService } from "../../../src/services/UserService";
import { UserModel } from "../../../src/models/User";
import bcrypt from "bcrypt";
import jwt from "jsonwebtoken";

jest.mock("../../../src/models/User");
jest.mock("bcrypt");
jest.mock("jsonwebtoken");

describe("UserService", () => {
  afterEach(() => {
    jest.clearAllMocks();
  });

  describe("registerUser", () => {
    it("should create a new user with hashed password", async () => {
      const mockUser = {
        id: "123",
        email: "test@example.com",
        name: "Test User",
        passwordHash: "hashed_password",
      };

      (UserModel.findByEmail as jest.Mock).mockResolvedValueOnce(null);
      (bcrypt.hash as jest.Mock).mockResolvedValueOnce("hashed_password");
      (UserModel.create as jest.Mock).mockResolvedValueOnce(mockUser);

      const result = await UserService.registerUser(
        "test@example.com",
        "Test User",
        "password123",
      );

      expect(result).toEqual({
        id: "123",
        email: "test@example.com",
        name: "Test User",
      });
      expect(bcrypt.hash).toHaveBeenCalledWith("password123", 12);
    });

    it("should throw error if user exists", async () => {
      (UserModel.findByEmail as jest.Mock).mockResolvedValueOnce({
        id: "123",
        email: "exist@example.com",
      });

      await expect(
        UserService.registerUser("exist@example.com", "Name", "password"),
      ).rejects.toThrow("User already exists");
    });
  });

  describe("login", () => {
    it("should return user and token on valid credentials", async () => {
      const mockUser = {
        id: "123",
        email: "test@example.com",
        name: "Test User",
        passwordHash: "hashed_password",
      };

      (UserModel.findByEmail as jest.Mock).mockResolvedValueOnce(mockUser);
      (bcrypt.compare as jest.Mock).mockResolvedValueOnce(true);
      (jwt.sign as jest.Mock).mockReturnValueOnce("jwt_token");

      const result = await UserService.login("test@example.com", "password");

      expect(result).toHaveProperty("user");
      expect(result).toHaveProperty("token", "jwt_token");
      expect(result.user).not.toHaveProperty("passwordHash");
    });

    it("should throw error on invalid credentials", async () => {
      (UserModel.findByEmail as jest.Mock).mockResolvedValueOnce(null);

      await expect(
        UserService.login("test@example.com", "password"),
      ).rejects.toThrow("Invalid credentials");
    });
  });
});
```

### Integration Tests

**`tests/integration/routes/auth.test.ts`**

```typescript
import request from "supertest";
import app from "../../../src/app";
import prisma from "../../../src/models/prismaClient";

jest.mock("../../../src/models/prismaClient");

describe("Auth Routes", () => {
  afterEach(() => {
    jest.clearAllMocks();
  });

  describe("POST /api/auth/register", () => {
    it("should register a new user", async () => {
      const newUser = {
        id: "123",
        email: "newuser@example.com",
        name: "New User",
      };

      (prisma.user.findUnique as jest.Mock).mockResolvedValueOnce(null);
      (prisma.user.create as jest.Mock).mockResolvedValueOnce(newUser);

      const response = await request(app).post("/api/auth/register").send({
        email: "newuser@example.com",
        name: "New User",
        password: "SecurePassword123",
      });

      expect(response.status).toBe(201);
      expect(response.body).toHaveProperty("id");
      expect(response.body.email).toBe("newuser@example.com");
    });

    it("should return 400 on invalid email", async () => {
      const response = await request(app).post("/api/auth/register").send({
        email: "invalid-email",
        name: "User",
        password: "password",
      });

      expect(response.status).toBe(400);
      expect(response.body).toHaveProperty("error");
    });
  });

  describe("POST /api/auth/login", () => {
    it("should return token on valid credentials", async () => {
      const user = {
        id: "123",
        email: "user@example.com",
        passwordHash: "hashed",
      };

      (prisma.user.findUnique as jest.Mock).mockResolvedValueOnce(user);

      const response = await request(app).post("/api/auth/login").send({
        email: "user@example.com",
        password: "password",
      });

      expect(response.status).toBe(200);
      expect(response.body).toHaveProperty("token");
    });
  });
});
```

### Test Utilities

**`tests/fixtures/users.ts`**

```typescript
export const mockUsers = {
  admin: {
    id: "user-1",
    email: "admin@example.com",
    name: "Admin User",
    role: "admin",
    createdAt: new Date("2024-01-01"),
  },
  user: {
    id: "user-2",
    email: "user@example.com",
    name: "Regular User",
    role: "user",
    createdAt: new Date("2024-01-02"),
  },
};

export const validUserInput = {
  email: "newuser@example.com",
  name: "New User",
  password: "SecurePassword123",
};
```

---

## 18. CI/CD Pipeline

### GitHub Actions Example

**`.github/workflows/test.yml`**

```yaml
name: Test & Build

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npm run type-check

      - name: Run tests
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
        run: npm test -- --coverage

      - name: Build
        run: npm run build

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
```

### Pre-commit Hooks

**`.husky/pre-commit`**

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npm run lint-staged
```

**`package.json`**

```json
{
  "lint-staged": {
    "src/**/*.ts": ["eslint --fix", "prettier --write"]
  }
}
```

---

## 19. Deployment

### Docker Setup

**`Dockerfile`**

```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY tsconfig.json ./
COPY src ./src

RUN npm run build

# Production stage
FROM node:18-alpine

WORKDIR /app

ENV NODE_ENV=production

COPY package*.json ./
RUN npm ci --only=production

COPY --from=builder /app/dist ./dist

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {if (r.statusCode !== 200) throw new Error(r.statusCode)})"

CMD ["node", "dist/index.js"]
```

**`.dockerignore`**

```
node_modules
npm-debug.log
.git
.gitignore
.env
dist
coverage
tests
.vscode
.DS_Store
```

### Docker Compose

**`docker-compose.yml`**

```yaml
version: "3.9"

services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@db:5432/express_db
      - JWT_SECRET=your-secret
      - LOG_LEVEL=info
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: express_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

volumes:
  postgres_data:
```

### Deployment to Cloud (AWS ECS Example)

```bash
# Build and push image
docker build -t express-api:latest .
docker tag express-api:latest $AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/express-api:latest
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com
docker push $AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/express-api:latest

# Update ECS service
aws ecs update-service \
  --cluster production \
  --service express-api \
  --force-new-deployment
```

### Nginx Reverse Proxy Config

**`nginx.conf`**

```nginx
upstream express_api {
  server api:3000;
}

server {
  listen 80;
  server_name api.example.com;

  client_max_body_size 10M;

  location / {
    proxy_pass http://express_api;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_cache_bypass $http_upgrade;

    # Timeouts
    proxy_connect_timeout 60s;
    proxy_send_timeout 60s;
    proxy_read_timeout 60s;
  }

  # Health check
  location /health {
    proxy_pass http://express_api;
    access_log off;
  }
}
```

---

## 20. Production Example Project

### Complete Project Structure

```
express-realworld-api/
├── src/
│   ├── config/
│   │   └── logger.ts
│   ├── controllers/
│   │   ├── AuthController.ts
│   │   ├── ArticleController.ts
│   │   └── CommentController.ts
│   ├── services/
│   │   ├── AuthService.ts
│   │   ├── ArticleService.ts
│   │   └── CommentService.ts
│   ├── models/
│   │   └── prismaClient.ts
│   ├── middleware/
│   │   ├── auth.ts
│   │   ├── errorHandler.ts
│   │   ├── validation.ts
│   │   └── rateLimiter.ts
│   ├── routes/
│   │   ├── auth.ts
│   │   ├── articles.ts
│   │   ├── comments.ts
│   │   └── index.ts
│   ├── utils/
│   │   ├── errors.ts
│   │   ├── validators.ts
│   │   └── jwt.ts
│   ├── types/
│   │   └── express.d.ts
│   ├── app.ts
│   └── index.ts
└── prisma/
    └── schema.prisma
```

### App Factory (Complete)

**`src/app.ts`**

```typescript
import express, { Express } from "express";
import cors from "cors";
import helmet from "helmet";
import compression from "compression";
import rateLimit from "express-rate-limit";

import { config } from "./config";
import { errorHandler } from "./middleware/errorHandler";
import { requestIdMiddleware } from "./middleware/requestId";
import { corsMiddleware } from "./middleware/cors";
import { authMiddleware } from "./middleware/auth";
import { requestLoggerMiddleware } from "./middleware/logger";
import { globalLimiter } from "./middleware/rateLimiter";
import apiRoutes from "./routes";

export function createApp(): Express {
  const app = express();

  // Trust proxy
  app.set("trust proxy", 1);

  // Security
  app.use(helmet());
  app.use(compression());
  app.use(corsMiddleware);
  app.use(globalLimiter);

  // Parsing
  app.use(express.json({ limit: "10mb" }));
  app.use(express.urlencoded({ extended: true, limit: "10mb" }));

  // Infrastructure
  app.use(requestIdMiddleware);
  app.use(requestLoggerMiddleware);

  // Health check
  app.get("/health", (req, res) => {
    res.json({ status: "ok" });
  });

  // API routes
  app.use("/api/v1", apiRoutes);

  // 404
  app.use((req, res) => {
    res.status(404).json({
      error: {
        code: "NOT_FOUND",
        message: `Route ${req.method} ${req.path} not found`,
      },
    });
  });

  // Error handler
  app.use(errorHandler);

  return app;
}

export default createApp;
```

### Routes Configuration

**`src/routes/index.ts`**

```typescript
import { Router } from "express";
import authRoutes from "./auth";
import articleRoutes from "./articles";
import commentRoutes from "./comments";
import { authMiddleware } from "../middleware/auth";

const router = Router();

// Public
router.use("/auth", authRoutes);

// Protected
router.use("/articles", articleRoutes);
router.use("/comments", authMiddleware, commentRoutes);

export default router;
```

### Article Routes (Nested)

**`src/routes/articles.ts`**

```typescript
import { Router } from "express";
import { body, param, query, validationResult } from "express-validator";
import { ArticleController } from "../controllers/ArticleController";
import { authMiddleware } from "../middleware/auth";
import commentRoutes from "./comments";

const router = Router({ mergeParams: true });

const handleValidationErrors = (req: any, res: any, next: any) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    res.status(400).json({ errors: errors.array() });
    return;
  }
  next();
};

// Public
router.get("/", ArticleController.listArticles);
router.get(
  "/:id",
  [param("id").isUUID()],
  handleValidationErrors,
  ArticleController.getArticle,
);

// Protected
router.post(
  "/",
  authMiddleware,
  [
    body("title").notEmpty().trim(),
    body("content").notEmpty(),
    body("tags").optional().isArray(),
  ],
  handleValidationErrors,
  ArticleController.createArticle,
);

router.put(
  "/:id",
  authMiddleware,
  [param("id").isUUID()],
  handleValidationErrors,
  ArticleController.updateArticle,
);

router.delete(
  "/:id",
  authMiddleware,
  [param("id").isUUID()],
  handleValidationErrors,
  ArticleController.deleteArticle,
);

// Nested comments
router.use("/:articleId/comments", commentRoutes);

export default router;
```

### Article Controller (Complete)

**`src/controllers/ArticleController.ts`**

```typescript
import { Request, Response, NextFunction } from "express";
import { ArticleService } from "../services/ArticleService";
import { AppError } from "../utils/errors";

export class ArticleController {
  static async listArticles(req: Request, res: Response, next: NextFunction) {
    try {
      const page = parseInt(req.query.page as string) || 1;
      const limit = parseInt(req.query.limit as string) || 20;
      const tags = (req.query.tags as string)?.split(",") || [];

      const result = await ArticleService.listArticles({
        page,
        limit,
        tags,
      });

      res.json(result);
    } catch (error) {
      next(error);
    }
  }

  static async getArticle(req: Request, res: Response, next: NextFunction) {
    try {
      const { id } = req.params;
      const article = await ArticleService.getArticleById(id);

      res.json(article);
    } catch (error) {
      next(error);
    }
  }

  static async createArticle(req: Request, res: Response, next: NextFunction) {
    try {
      const userId = (req as any).user.id;
      const { title, content, tags } = req.body;

      const article = await ArticleService.createArticle({
        title,
        content,
        tags,
        authorId: userId,
      });

      res.status(201).json(article);
    } catch (error) {
      next(error);
    }
  }

  static async updateArticle(req: Request, res: Response, next: NextFunction) {
    try {
      const { id } = req.params;
      const userId = (req as any).user.id;
      const updates = req.body;

      const article = await ArticleService.updateArticle(id, userId, updates);

      res.json(article);
    } catch (error) {
      next(error);
    }
  }

  static async deleteArticle(req: Request, res: Response, next: NextFunction) {
    try {
      const { id } = req.params;
      const userId = (req as any).user.id;

      await ArticleService.deleteArticle(id, userId);

      res.status(204).send();
    } catch (error) {
      next(error);
    }
  }
}
```

### Article Service (Business Logic)

**`src/services/ArticleService.ts`**

```typescript
import prisma from "../models/prismaClient";
import { AppError, NotFoundError } from "../utils/errors";

export class ArticleService {
  static async listArticles(options: {
    page: number;
    limit: number;
    tags?: string[];
  }) {
    const { page, limit, tags } = options;
    const skip = (page - 1) * limit;

    const [articles, total] = await Promise.all([
      prisma.article.findMany({
        where: tags?.length ? { tags: { hasSome: tags } } : {},
        skip,
        take: limit,
        include: { author: { select: { id: true, name: true, email: true } } },
        orderBy: { createdAt: "desc" },
      }),
      prisma.article.count({
        where: tags?.length ? { tags: { hasSome: tags } } : {},
      }),
    ]);

    return {
      data: articles,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit),
      },
    };
  }

  static async getArticleById(id: string) {
    const article = await prisma.article.findUnique({
      where: { id },
      include: {
        author: { select: { id: true, name: true, email: true } },
        comments: {
          include: { author: { select: { id: true, name: true } } },
        },
      },
    });

    if (!article) {
      throw new NotFoundError("Article");
    }

    return article;
  }

  static async createArticle(data: {
    title: string;
    content: string;
    authorId: string;
    tags?: string[];
  }) {
    return prisma.article.create({
      data,
      include: { author: { select: { id: true, name: true } } },
    });
  }

  static async updateArticle(
    id: string,
    userId: string,
    updates: Partial<any>,
  ) {
    const article = await prisma.article.findUnique({ where: { id } });

    if (!article) {
      throw new NotFoundError("Article");
    }

    if (article.authorId !== userId) {
      throw new AppError("Not authorized", 403, "FORBIDDEN");
    }

    return prisma.article.update({
      where: { id },
      data: {
        title: updates.title,
        content: updates.content,
        tags: updates.tags,
      },
      include: { author: { select: { id: true, name: true } } },
    });
  }

  static async deleteArticle(id: string, userId: string) {
    const article = await prisma.article.findUnique({ where: { id } });

    if (!article) {
      throw new NotFoundError("Article");
    }

    if (article.authorId !== userId) {
      throw new AppError("Not authorized", 403, "FORBIDDEN");
    }

    await prisma.article.delete({ where: { id } });
  }
}
```

### Server Startup (Complete)

**`src/index.ts`**

```typescript
import createApp from "./app";
import { config } from "./config";
import logger from "./config/logger";

const app = createApp();

const server = app.listen(config.app.port, config.app.host, () => {
  logger.info(`Server running on ${config.app.host}:${config.app.port}`);
});

// Graceful shutdown
const shutdown = (signal: string) => {
  logger.info(`${signal} received, initiating graceful shutdown`);

  server.close(() => {
    logger.info("HTTP server closed");
    process.exit(0);
  });

  // Force close after 30 seconds
  setTimeout(() => {
    logger.error("Forcefully closing server");
    process.exit(1);
  }, 30000);
};

process.on("SIGTERM", () => shutdown("SIGTERM"));
process.on("SIGINT", () => shutdown("SIGINT"));

process.on("uncaughtException", (error) => {
  logger.error("Uncaught Exception", { stack: error.stack });
  process.exit(1);
});

process.on("unhandledRejection", (reason, promise) => {
  logger.error("Unhandled Rejection", { reason });
  process.exit(1);
});
```

---

## Key Takeaways for Production

### ✅ DO

1. **Use TypeScript** - Type safety catches bugs early
2. **Organize by feature** - Routes, controllers, services in separate files
3. **Centralize error handling** - One error handler for all routes
4. **Log structurally** - JSON logs with request IDs
5. **Validate inputs** - Use Zod or express-validator
6. **Implement authentication** - JWT or OAuth
7. **Rate limit** - Protect against abuse
8. **Use environment variables** - Never hardcode secrets
9. **Monitor performance** - Track response times
10. **Test thoroughly** - Unit + integration tests

### ❌ DON'T

1. ❌ Put everything in one file
2. ❌ Mix layers (controllers calling DB directly)
3. ❌ Ignore CORS in production
4. ❌ Log sensitive data
5. ❌ Use `console.log` in production
6. ❌ Block async operations
7. ❌ Trust user input
8. ❌ Run without rate limiting
9. ❌ Skip error boundaries
10. ❌ Deploy without testing

---

## Additional Resources

- [Express.js Official Documentation](https://expressjs.com/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [Prisma ORM Documentation](https://www.prisma.io/docs/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

---

**Document Version**: 1.0  
**Last Updated**: 2026  
**Maintenance**: ME "Ahmed Arafa"
