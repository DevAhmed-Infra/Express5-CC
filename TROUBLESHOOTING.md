# Express.js 5: Common Issues & Troubleshooting Guide

## Startup & Configuration Issues

### Issue: "Cannot find module 'express'"

**Symptoms**:

```
Error: Cannot find module 'express'
```

**Causes**:

- Package not installed
- Wrong node_modules directory
- Corrupted installation

**Solution**:

```bash
# Clean and reinstall
rm -rf node_modules package-lock.json
npm install

# Verify installation
npm list express
```

---

### Issue: "Port already in use"

**Symptoms**:

```
Error: listen EADDRINUSE: address already in use :::3000
```

**Solution**:

```bash
# Find process using port
lsof -i :3000

# Kill process (macOS/Linux)
kill -9 <PID>

# Or change port
PORT=3001 npm run dev

# Or use environment variable
process.env.PORT || 3000
```

---

### Issue: Top-level await not supported

**Symptoms**:

```
SyntaxError: await is only valid in async functions and at the top level of modules
```

**Solution**:
Update `tsconfig.json`:

```json
{
  "compilerOptions": {
    "module": "esnext",
    "target": "ES2020"
  }
}
```

---

### Issue: ".env file not loading"

**Symptoms**:

```
process.env.DATABASE_URL is undefined
```

**Solution**:

```typescript
// At app start (BEFORE any other imports)
import dotenv from "dotenv";
dotenv.config();

// Verify config loaded
console.log(process.env.DATABASE_URL); // Should be defined
```

**Or** use `-r` flag:

```bash
node -r dotenv/config dist/index.js
```

---

### Issue: "Cannot use import/export syntax" or "require is not defined"

**Symptoms**:

```
SyntaxError: Cannot use import statement outside a module
// OR
ReferenceError: require is not defined
```

**Cause**: Module system mismatch between package.json and tsconfig.json

**Solution**: Choose and configure your module system properly.

#### Option 1: CommonJS (recommended for most projects)

**`package.json`** (no "type" field needed, default is CommonJS):

```json
{
  "name": "express-app",
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
    "moduleResolution": "node",
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

**`src/index.ts` (CommonJS syntax)**:

```typescript
// CommonJS: require()
import express from "express"; // Works via esModuleInterop
import dotenv from "dotenv";

const app = express();
const config = require("./config"); // Or use import

// Start server
app.listen(3000, () => console.log("Running"));
```

#### Option 2: ES Modules (modern, faster)

**`package.json` (with "type": "module")**:

```json
{
  "name": "express-app",
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
    "moduleResolution": "bundler",
    "esModuleInterop": false,
    "allowSyntheticDefaultImports": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

**`src/index.ts` (ES Modules syntax)**:

```typescript
// ES Modules: import/export
import express from "express";
import dotenv from "dotenv";
import { config } from "./config.js"; // Must include .js extension

const app = express();

// Top-level await works in ES Modules
const startServer = async () => {
  await app.listen(3000);
  console.log("Running");
};

startServer();
```

#### Module System Comparison

| Feature                | CommonJS              | ES Modules                    |
| ---------------------- | --------------------- | ----------------------------- |
| Syntax                 | `require()`           | `import/export`               |
| Asynchronous           | No                    | Yes (top-level await)         |
| Performance            | Good                  | Better (faster tree-shaking)  |
| Node.js Support        | Native (all versions) | v12+ (with flag), v14+ native |
| Browser Compatible     | No                    | Yes                           |
| Default in Node        | Yes                   | No (use "type": "module")     |
| .js extension required | No                    | Yes                           |
| `__dirname` available  | Yes                   | Need to create it             |

#### If Using ES Modules, create `__dirname`:

```typescript
import { fileURLToPath } from "url";
import { dirname } from "path";

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

#### Mixed Setup (not recommended):

```json
{
  "type": "module",
  "exports": {
    "import": "./dist/index.js",
    "require": "./dist/index.cjs"
  }
}
```

---

### Issue: "Nodemon not working with ES Modules" or "Cannot find module"

**Symptoms**:

```
Module not found: Cannot find module 'express'
or
Nodemon crashes immediately with ES Modules
```

**Causes**:

- Nodemon not configured for ES Modules
- Missing ts-node loaders
- File extensions not included in imports

**Solution**:

#### For CommonJS (no special config needed):

```json
{
  "scripts": {
    "dev": "nodemon"
  }
}
```

```js
// nodemon.json
{
  "watch": ["src"],
  "ext": "ts",
  "exec": "ts-node",
  "execMap": {
    "ts": "ts-node --transpile-only"
  }
}
```

#### For ES Modules:

**`package.json`**:

```json
{
  "type": "module",
  "scripts": {
    "dev": "nodemon --ext ts --exec node --loader ts-node/esm",
    "build": "tsc",
    "start": "node dist/index.js"
  }
}
```

Or with `nodemon.json`:

```json
{
  "watch": ["src"],
  "ext": "ts",
  "exec": "node --loader ts-node/esm",
  "env": {
    "NODE_OPTIONS": "--loader ts-node/esm"
  }
}
```

#### Ensure `.ts` files have proper imports for ES Modules:

```typescript
// ✅ Correct (include .js extension)
import { config } from "./config.js";
import express from "express";

// ❌ Wrong (missing extension)
import { config } from "./config";
```

#### Install required dependencies:

```bash
npm install -D ts-node nodemon
npm install -D @types/node typescript
```

---

## Middleware Issues

### Issue: Middleware not executing

**Symptoms**:
Middleware logs not showing, requests not processed

**Causes**:

- Middleware registered after route
- Route handler not calling `next()`
- Middleware chain broken

**Solution**:

```typescript
// ❌ Wrong: Middleware after route
app.get("/users", handler);
app.use(cors());

// ✅ Correct: Middleware before routes
app.use(cors());
app.use(express.json());
app.get("/users", handler);

// ❌ Wrong: Middleware doesn't call next()
app.use((req, res, next) => {
  console.log("Log");
  // Missing: next();
});

// ✅ Correct: Always call next() in middleware
app.use((req, res, next) => {
  console.log("Log");
  next(); // Pass to next middleware
});
```

---

### Issue: CORS errors in browser

**Symptoms**:

```
Access to XMLHttpRequest at 'http://localhost:3000/api'
from origin 'http://localhost:3001' has been blocked by CORS policy
```

**Solution**:

```typescript
import cors from "cors";

// ✅ Allow specific origin
app.use(
  cors({
    origin: "http://localhost:3001",
    credentials: true,
  }),
);

// ✅ Allow multiple origins
const allowedOrigins = ["http://localhost:3000", "https://example.com"];
app.use(
  cors({
    origin: (origin, callback) => {
      if (allowedOrigins.includes(origin || "")) {
        callback(null, true);
      } else {
        callback(new Error("Not allowed by CORS"));
      }
    },
  }),
);

// ✅ For development (allow all)
if (process.env.NODE_ENV === "development") {
  app.use(cors());
}
```

---

### Issue: Asynchronous middleware errors not caught

**Symptoms**:

```
UnhandledPromiseRejection: User not found
```

**Solution**:

```typescript
// ❌ Wrong: Async error not caught
app.get("/users/:id", async (req, res) => {
  const user = await db.getUser(req.params.id); // Throws!
  res.json(user);
});

// ✅ Correct: Use try-catch
app.get("/users/:id", async (req, res, next) => {
  try {
    const user = await db.getUser(req.params.id);
    res.json(user);
  } catch (error) {
    next(error); // Pass to error handler
  }
});

// ✅ Or use async wrapper
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get(
  "/users/:id",
  asyncHandler(async (req, res) => {
    const user = await db.getUser(req.params.id);
    res.json(user);
  }),
);
```

---

## Routing Issues

### Issue: Route not matching

**Symptoms**:

```
404 Not Found: /users
```

**Causes**:

- Wrong HTTP method
- Route pattern mismatch
- Conflicting routes

**Solution**:

```typescript
// ❌ Wrong: Pattern doesn't match
app.get("/users/:id", handler); // No match for /users/123/posts

// ✅ Correct: Parameterized route
app.get("/users/:id", handler); // Matches /users/123

// ✅ Wildcard for nested
app.get("/users/:id/*", handler); // Matches /users/123/anything

// ✅ Route priority (order matters)
app.get("/users/me", meHandler); // Must come BEFORE
app.get("/users/:id", userHandler); // This route
```

---

### Issue: URL parameters undefined

**Symptoms**:

```typescript
console.log(req.params.id); // undefined
```

**Solution**:

```typescript
// ✅ Correct: Parameter in route definition
app.get("/users/:id", (req, res) => {
  console.log(req.params.id); // ✓ Defined
});

// ✅ Type-safe with TypeScript
app.get<{ id: string }>("/users/:id", (req, res) => {
  const id: string = req.params.id; // Type-safe
});
```

---

## Request/Response Issues

### Issue: Request body undefined

**Symptoms**:

```typescript
console.log(req.body); // undefined
```

**Causes**:

- Missing JSON parser middleware
- Content-Type header mismatch

**Solution**:

```typescript
// ✅ Add JSON parser BEFORE routes
app.use(express.json());

// ✅ Set correct Content-Type in client
fetch("/api/users", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ email: "user@example.com" }),
});
```

---

### Issue: Sending response twice

**Symptoms**:

```
Error: Cannot set headers after they are sent to the client
```

**Causes**:

- `res.send()` called multiple times
- Async race condition

**Solution**:

```typescript
// ❌ Wrong: Multiple responses
app.get("/users", async (req, res) => {
  const users = await db.getUsers();
  res.json(users);
  res.send("Done"); // ERROR!
});

// ✅ Correct: Single response
app.get("/users", async (req, res) => {
  const users = await db.getUsers();
  res.json(users);
  return; // Explicit return
});

// ✅ Handle async properly
app.get("/users/:id", async (req, res, next) => {
  try {
    const user = await db.getUser(req.params.id);
    if (!user) {
      return res.status(404).json({ error: "Not found" });
    }
    return res.json(user); // Explicit return
  } catch (error) {
    next(error);
  }
});
```

---

### Issue: Large file upload fails

**Symptoms**:

```
413 Payload Too Large
```

**Solution**:

```typescript
app.use(express.json({ limit: "50mb" }));
app.use(express.urlencoded({ limit: "50mb" }));

// For file uploads
import multer from "multer";
const upload = multer({
  limits: { fileSize: 100 * 1024 * 1024 }, // 100MB
});

app.post("/upload", upload.single("file"), (req, res) => {
  res.json({ filename: req.file?.filename });
});
```

---

## Validation Issues

### Issue: Validation not working

**Symptoms**:

```
Invalid data accepted, or validation errors not returned
```

**Solution**:

```typescript
import { body, validationResult } from "express-validator";

// ✅ Proper validation
app.post(
  "/users",
  body("email").isEmail(),
  body("password").isLength({ min: 8 }),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    res.json({ created: true });
  },
);

// ✅ With Zod
import { z } from "zod";

const userSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

app.post("/users", (req, res) => {
  try {
    const data = userSchema.parse(req.body);
    res.json({ created: true });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});
```

---

## Authentication Issues

### Issue: JWT verification fails

**Symptoms**:

```
401 Unauthorized: Invalid token
```

**Solution**:

```typescript
// ✅ Proper JWT handling
const authMiddleware = (req, res, next) => {
  const authHeader = req.headers.authorization;

  if (!authHeader?.startsWith("Bearer ")) {
    return res.status(401).json({ error: "Missing token" });
  }

  const token = authHeader.substring(7);

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    if (error.name === "TokenExpiredError") {
      return res.status(401).json({ error: "Token expired" });
    }
    return res.status(401).json({ error: "Invalid token" });
  }
};
```

---

### Issue: Passwords not secure

**Symptoms**:

```
Passwords stored in plain text or weak hash
```

**Solution**:

```typescript
import bcrypt from "bcrypt";

// ✅ Hash password on registration
app.post("/register", async (req, res, next) => {
  try {
    const { email, password } = req.body;

    // Hash with 12 rounds (production standard)
    const passwordHash = await bcrypt.hash(password, 12);

    const user = await db.createUser({
      email,
      passwordHash,
    });

    res.status(201).json({ id: user.id });
  } catch (error) {
    next(error);
  }
});

// ✅ Verify password on login
const isValid = await bcrypt.compare(password, user.passwordHash);
if (!isValid) {
  return res.status(401).json({ error: "Invalid credentials" });
}
```

---

## Database Issues

### Issue: "Database connection refused"

**Symptoms**:

```
Error: connect ECONNREFUSED 127.0.0.1:5432
```

**Solution**:

```bash
# Check database is running
docker ps | grep postgres

# Or start it
docker-compose up -d

# Verify connection string
echo $DATABASE_URL

# Test connection
psql $DATABASE_URL -c "SELECT 1;"
```

---

### Issue: Database migration fails

**Symptoms**:

```
Prisma Migration error: Table already exists
```

**Solution (for development)**:

```bash
# Reset database (WARNING: deletes data)
npx prisma migrate reset

# Or manually
npx prisma migrate deploy
npx prisma generate
```

---

### Issue: N+1 query problem

**Symptoms**:

```
Many database queries for simple operation (slow)
```

**Solution**:

```typescript
// ❌ N+1: Query for each item
const users = await prisma.user.findMany();
for (const user of users) {
  user.posts = await prisma.post.findMany({
    where: { authorId: user.id },
  });
}

// ✅ Use include
const users = await prisma.user.findMany({
  include: { posts: true },
});

// ✅ Or select specific fields
const users = await prisma.user.findMany({
  select: {
    id: true,
    email: true,
    posts: { select: { id: true, title: true } },
  },
});
```

---

## Error Handling Issues

### Issue: Error handler not catching errors

**Symptoms**:

```
UnhandledPromiseRejection or error not formatted properly
```

**Solution**:

```typescript
// ✅ Error handler MUST be last
app.use("/api", routes);

// 404 handler (before error handler)
app.use((req, res) => {
  res.status(404).json({ error: "Not found" });
});

// Error handler (LAST)
app.use((err, req, res, next) => {
  const status = err.status || 500;
  const message = err.message || "Internal Server Error";

  res.status(status).json({
    error: {
      code: err.code || "INTERNAL_ERROR",
      message,
      requestId: req.id,
    },
  });
});

// ✅ Catch Promise rejections
process.on("unhandledRejection", (reason) => {
  console.error("Unhandled Rejection:", reason);
  process.exit(1);
});

process.on("uncaughtException", (error) => {
  console.error("Uncaught Exception:", error);
  process.exit(1);
});
```

---

## Performance Issues

### Issue: Slow response times

**Symptoms**:

```
Endpoint takes > 1 second
```

**Diagnosis**:

```typescript
// Add timing middleware
const timingMiddleware = (req, res, next) => {
  const start = Date.now();
  res.on("finish", () => {
    const duration = Date.now() - start;
    if (duration > 1000) {
      console.warn(`Slow route: ${req.path} - ${duration}ms`);
    }
  });
  next();
};

app.use(timingMiddleware);
```

**Solutions**:

```typescript
// 1. Add compression
app.use(compression());

// 2. Cache responses
const cache = {};
const cacheMiddleware = (duration) => (req, res, next) => {
  if (cache[req.path]) {
    return res.json(cache[req.path]);
  }
  const originalJson = res.json.bind(res);
  res.json = (data) => {
    cache[req.path] = data;
    setTimeout(() => delete cache[req.path], duration);
    return originalJson(data);
  };
  next();
};

// 3. Use pagination
app.get('/items', (req, res) => {
  const page = req.query.page || 1;
  const limit = 20;
  const offset = (page - 1) * limit;
  // Query with LIMIT/OFFSET
});

// 4. Add database indexes
// In Prisma schema
model User {
  id String @id
  email String @unique // ← Database index
  @@index([createdAt])
}
```

---

### Issue: Memory leak

**Symptoms**:

```
Memory usage constantly growing
Process crashes after hours
```

**Diagnosis**:

```typescript
// Monitor memory
setInterval(() => {
  const mem = process.memoryUsage();
  console.log({
    rss: `${Math.round(mem.rss / 1024 / 1024)}MB`,
    heap: `${Math.round(mem.heapUsed / 1024 / 1024)}MB`,
  });
}, 60000);
```

**Solutions**:

```typescript
// 1. Clear cache periodically
const cache = new Map();
setInterval(() => {
  if (cache.size > 1000) cache.clear();
}, 3600000); // Every hour

// 2. Avoid global state
// ❌ Wrong
let users = []; // Grows forever

// ✅ Correct
app.get("/users", async (req, res) => {
  const users = await db.getUsers(); // Fresh each time
  res.json(users);
});

// 3. Properly close database connections
process.on("SIGTERM", async () => {
  await prisma.$disconnect();
  process.exit(0);
});
```

---

## Testing Issues

### Issue: Tests timing out

**Symptoms**:

```
Jest timeout after 5000ms
```

**Solution**:

```typescript
describe("User API", () => {
  jest.setTimeout(10000); // Increase timeout

  it("should fetch user", async () => {
    const user = await request(app).get("/api/users/1");

    expect(user.status).toBe(200);
  }, 10000); // Or per test
});
```

---

### Issue: Database not resetting between tests

**Symptoms**:

```
Tests affect each other / Duplicate key errors
```

**Solution**:

```typescript
describe("User API", () => {
  beforeEach(async () => {
    // Clear database before each test
    await prisma.user.deleteMany();
  });

  afterAll(async () => {
    // Close database connection
    await prisma.$disconnect();
  });

  it("should create user", async () => {
    // ...
  });
});
```

---

## Docker Issues

### Issue: "Cannot connect to Docker daemon"

**Solution**:

```bash
# Start Docker daemon
docker daemon

# Or on macOS
open /Applications/Docker.app

# Verify
docker ps
```

---

### Issue: Container exits immediately

**Symptoms**:

```
docker ps shows container exited
docker logs shows compilation error
```

**Solution**:

```bash
# Check logs
docker logs <container_id>

# Build with verbose output
docker build -t app . --progress=plain

# Run with interactive terminal
docker run -it app sh
```

---

### Issue: Port binding in Docker

**Symptoms**:

```
Cannot bind port 3000
```

**Solution**:

```bash
# Check what's using port
lsof -i :3000

# Use different port
docker run -p 3001:3000 app

# Or kill existing container
docker stop <container_id>
```

---

## Production Deployment Issues

### Issue: Environment variables not loaded

**Solution**:

```dockerfile
# In Dockerfile
ENV NODE_ENV=production
ENV PORT=3000

# Or via docker-compose
environment:
  - NODE_ENV=production
  - DATABASE_URL=${DATABASE_URL}

# Or via .env file
docker run --env-file .env.production app
```

---

### Issue: Application crashes after deploy

**Solution**:

```typescript
// Add health check
app.get("/health", (req, res) => {
  res.json({ status: "ok" });
});

// Add startup delay
setTimeout(() => {
  app.listen(PORT, () => console.log("Ready"));
}, 2000);

// Add graceful shutdown
process.on("SIGTERM", async () => {
  server.close(async () => {
    await db.disconnect();
    process.exit(0);
  });
});
```

---

## Debugging Techniques

### Enable Debug Logs

```bash
# Express debug
DEBUG=express* npm run dev

# Prisma debug
DEBUG=prisma* npm run dev

# All debug
DEBUG=* npm run dev
```

### Add Request Tracing

```typescript
import { randomUUID } from "crypto";

app.use((req, res, next) => {
  req.id = randomUUID();
  console.log(`[${req.id}] ${req.method} ${req.path}`);
  next();
});

// Pass request ID to services
service.method(userId, { requestId: req.id });
```

### Use Node Debugger

```bash
# Start with debugger
node --inspect dist/index.js

# Visit: chrome://inspect
```

---

**Version**: 1.0 | **Date**: 2026 | **Maintained By**: Backend Team
