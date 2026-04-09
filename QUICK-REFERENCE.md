# Express.js 5: Quick Reference & Cheat Sheet

## Common Patterns at a Glance

### Route Definition

```typescript
// Basic
app.get("/path", handler);
app.post("/path", middleware, handler);
app.put("/path/:id", handler);
app.delete("/path/:id", handler);

// Router
const router = Router();
router.get("/", handler);
app.use("/api", router);

// Nested
router.use("/:userId/posts", postsRouter);
```

### Middleware Chain

```typescript
// Global middleware
app.use(cors());
app.use(express.json());

// Route-level middleware
app.post("/admin", auth, adminOnly, handler);

// Async middleware
const asyncMiddleware = async (req, res, next) => {
  try {
    await somePromise();
    next();
  } catch (err) {
    next(err);
  }
};
```

### Error Handling

```typescript
// Throw errors
throw new AppError("Message", 400, "CODE");

// Catch in middleware
try {
  await service.method();
  res.json(data);
} catch (err) {
  next(err);
}

// Global error handler (last)
app.use((err, req, res, next) => {
  res.status(err.status || 500).json({ error: err.message });
});
```

### Request Types

```typescript
// Typed controllers
app.get("/users/:id", (req: Request<{ id: string }>, res: Response) => {
  const id = req.params.id;
});

// Typed body
interface CreateUser {
  email: string;
  name: string;
}
app.post("/users", (req: Request<{}, {}, CreateUser>, res: Response) => {
  const { email, name } = req.body;
});

// Typed query
app.get("/search", (req: Request<{}, {}, {}, { q: string }>, res: Response) => {
  const query = req.query.q;
});
```

### Common Middleware

```typescript
// CORS
app.use(cors({ origin: "http://localhost:3000" }));

// JSON parsing
app.use(express.json({ limit: "10mb" }));

// Rate limiting
app.use(rateLimit({ windowMs: 15 * 60 * 1000, max: 100 }));

// Compression
app.use(compression());

// Helmet security
app.use(helmet());

// Request logging
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next();
});

// Auth
app.use((req, res, next) => {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) return res.status(401).json({ error: "Unauthorized" });
  req.user = verifyToken(token);
  next();
});
```

### Status Codes

```typescript
res.status(200).json(data); // OK
res.status(201).json(newData); // Created
res.status(204).send(); // No Content
res.status(400).json({ error: msg }); // Bad Request
res.status(401).json({ error: msg }); // Unauthorized
res.status(403).json({ error: msg }); // Forbidden
res.status(404).json({ error: msg }); // Not Found
res.status(409).json({ error: msg }); // Conflict
res.status(422).json({ error: msg }); // Unprocessable Entity
res.status(500).json({ error: msg }); // Internal Server Error
```

---

## Setup Checklist

- [ ] `npm init -y`
- [ ] `npm install express@5`
- [ ] `npm install -D typescript @types/node @types/express ts-node nodemon`
- [ ] Create `tsconfig.json`
- [ ] Create `src/index.ts`
- [ ] Create `src/app.ts`
- [ ] Setup routes folder `src/routes/`
- [ ] Setup config `src/config/index.ts`
- [ ] Install middleware: `cors`, `helmet`, `compression`
- [ ] Install validation: `zod` or `express-validator`
- [ ] Install database: `@prisma/client` + `prisma`
- [ ] Setup `prisma/schema.prisma`
- [ ] Create `.env` and `.env.production`
- [ ] Setup error handling middleware
- [ ] Add request logging
- [ ] Setup authentication
- [ ] Create tests folder
- [ ] Setup Docker + docker-compose

---

## Module System Configuration

### CommonJS Setup (Default, Traditional)

**`package.json`:**

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

**`tsconfig.json` (CommonJS):**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "moduleResolution": "node",
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "outDir": "./dist",
    "rootDir": "./src"
  }
}
```

**Usage in `src/index.ts`:**

```typescript
import express from "express"; // Works with esModuleInterop
const config = require("./config"); // Or const config = require()

const app = express();
app.listen(3000);
```

---

### ES Modules Setup (Modern, Recommended)

**`package.json`:**

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

**`tsconfig.json` (ES Modules):**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "esnext",
    "moduleResolution": "bundler",
    "esModuleInterop": false,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "outDir": "./dist",
    "rootDir": "./src"
  }
}
```

**Usage in `src/index.ts`:**

```typescript
import express from "express";
import { config } from "./config.js"; // .js extension required

const app = express();
app.listen(3000);

// Top-level await works in ES Modules
await app.listen(3000);
console.log("Ready");
```

---

### Comparison Table

| Feature                | CommonJS          | ES Modules                   |
| ---------------------- | ----------------- | ---------------------------- |
| **Syntax**             | `require()`       | `import/export`              |
| **Extension required** | ❌ No             | ✅ Yes (.js)                 |
| **Top-level await**    | ❌ No             | ✅ Yes                       |
| **Performance**        | Good              | Better                       |
| **Compatibility**      | All Node versions | v12+ (v14+ native)           |
| **Default**            | ✅ Yes            | ❌ No (use "type": "module") |
| **`__dirname`**        | ✅ Built-in       | ❌ Manual import needed      |
| **Tree-shaking**       | ❌ Limited        | ✅ Yes                       |

---

## Create `__dirname` for ES Modules

```typescript
// src/utils/paths.ts
import { fileURLToPath } from "url";
import { dirname } from "path";

export const __filename = fileURLToPath(import.meta.url);
export const __dirname = dirname(__filename);
```

**Usage:**

```typescript
import { __dirname } from "./utils/paths.js";

const filePath = `${__dirname}/data.json`;
```

---

## Nodemon Configuration

### For CommonJS

**`nodemon.json`:**

```json
{
  "watch": ["src"],
  "ext": "ts",
  "ignore": ["src/**/*.test.ts"],
  "exec": "ts-node",
  "execMap": {
    "ts": "ts-node --transpile-only"
  },
  "env": {
    "NODE_ENV": "development"
  }
}
```

**`package.json`:**

```json
{
  "scripts": {
    "dev": "nodemon"
  }
}
```

Run with:

```bash
npm run dev
```

### For ES Modules

**`nodemon.json`:**

```json
{
  "watch": ["src"],
  "ext": "ts",
  "ignore": ["src/**/*.test.ts"],
  "exec": "node --loader ts-node/esm",
  "env": {
    "NODE_ENV": "development",
    "NODE_OPTIONS": "--loader ts-node/esm"
  }
}
```

**`package.json`:**

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

Run with:

```bash
npm run dev
```

### Nodemon Options Reference

```json
{
  "watch": ["src"], // Directories to watch
  "ext": "ts,json", // Extensions to watch
  "ignore": ["**/*.test.ts"], // Patterns to ignore
  "exec": "ts-node", // Command to execute
  "execMap": {
    // Map file extensions to commands
    "ts": "ts-node --transpile-only"
  },
  "delay": 500, // Delay before restart (ms)
  "quiet": false, // Suppress nodemon logs
  "env": {
    // Environment variables
    "NODE_ENV": "development"
  }
}
```

---

## Environment Variables Template

```bash
# App
NODE_ENV=development
PORT=3000
HOST=0.0.0.0

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/db

# JWT
JWT_SECRET=your-secret-key
JWT_EXPIRES_IN=24h

# CORS
ALLOWED_ORIGINS=http://localhost:3000

# Logging
LOG_LEVEL=debug

# Security
BCRYPT_ROUNDS=12
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100
```

---

## Prisma Commands Quick Reference

```bash
# Initialize Prisma
npx prisma init

# Create migration
npx prisma migrate dev --name add_users_table

# Push schema to database
npx prisma db push

# Generate Prisma Client
npx prisma generate

# Open Prisma Studio
npx prisma studio

# Reset database
npx prisma migrate reset

# Check migration status
npx prisma migrate status
```

---

## Testing Commands

```bash
# Run all tests
npm test

# Watch mode
npm test -- --watch

# Coverage
npm test -- --coverage

# Specific file
npm test -- auth.test.ts

# Verbose
npm test -- --verbose
```

---

## Deployment Commands

```bash
# Build
npm run build

# Start production
NODE_ENV=production npm start

# Docker build
docker build -t myapp:latest .

# Docker run
docker run -p 3000:3000 --env-file .env.production myapp:latest

# Docker compose
docker-compose up -d

# Docker logs
docker logs -f container_name
```

---

## Debugging

```typescript
// Add request ID to all logs
app.use((req, res, next) => {
  req.id = uuidv4();
  next();
});

// Log with context
logger.info('Event', { requestId: req.id, userId: user.id });

// Debug specific module
DEBUG=express:* npm run dev
```

---

## Performance Tips

1. Use `compression()` middleware
2. Enable `gzip` in nginx/proxy
3. Use connection pooling in database
4. Cache static assets
5. Implement request caching
6. Use clustering for multiple CPUs
7. Monitor memory with `process.memoryUsage()`
8. Use pagination for large datasets
9. Add indexes to frequently queried columns
10. Profile with `node --prof`

---

## Security Checklist

- [ ] Use `helmet()` for security headers
- [ ] Configure CORS properly
- [ ] Implement rate limiting
- [ ] Hash passwords with `bcrypt`
- [ ] Use JWT with secret key
- [ ] Validate all inputs
- [ ] Sanitize user input (XSS protection)
- [ ] Use HTTPS in production
- [ ] Set secure cookie options
- [ ] Never log sensitive data
- [ ] Use environment variables for secrets
- [ ] Implement request timeouts
- [ ] Add CSRF protection
- [ ] Database query parameterization

---

## Common Errors & Fixes

| Error                              | Cause                      | Fix                                       |
| ---------------------------------- | -------------------------- | ----------------------------------------- |
| `Cannot find module 'express'`     | Missing dependency         | `npm install express`                     |
| `Cannot use import outside module` | TypeScript not compiled    | Add `"type": "module"` or use `require()` |
| `Port already in use`              | Port 3000 taken            | Change PORT env var or kill process       |
| `ECONNREFUSED`                     | Database not running       | Start Docker/database service             |
| `UnhandledPromiseRejection`        | Async error not caught     | Use `asyncHandler` wrapper                |
| `CORS error`                       | Origin not in whitelist    | Check ALLOWED_ORIGINS env var             |
| `JWT verification failed`          | Token expired/invalid      | Generate new token                        |
| `Validation error`                 | Input doesn't match schema | Check request body against schema         |

---

## Performance Benchmarks (Reference)

- Healthy endpoint: < 1ms
- API endpoint: < 50ms
- Database query: < 100ms
- File upload (10MB): < 5s
- Response time (p99): < 1s

---

## File Size Limits

```typescript
// Recommended defaults
app.use(express.json({ limit: "10mb" }));
app.use(express.urlencoded({ limit: "10mb" }));

// For file uploads
import multer from "multer";
const upload = multer({ limits: { fileSize: 50 * 1024 * 1024 } }); // 50MB
```

---

## Useful npm Packages

```bash
# HTTP
npm install cors helmet compression cookie-parser

# Validation
npm install zod express-validator

# Auth
npm install jsonwebtoken bcrypt passport

# Database
npm install @prisma/client prisma

# Logging
npm install winston pino

# Rate limiting
npm install express-rate-limit

# File uploads
npm install multer

# HTTP client
npm install axios

# UUID
npm install uuid

# Testing
npm install -D jest @types/jest supertest ts-jest

# Linting
npm install -D eslint prettier @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

---

## Express vs Alternatives

| Feature            | Express   | Fastify   | Koa    | NestJS    |
| ------------------ | --------- | --------- | ------ | --------- |
| Learning Curve     | Easy      | Medium    | Hard   | Hard      |
| Opinionated        | No        | No        | No     | Yes       |
| TypeScript Support | Good      | Excellent | Good   | Excellent |
| Middleware         | Yes       | Yes       | Yes    | Yes       |
| Performance        | Good      | Excellent | Good   | Good      |
| ORM Integration    | Any       | Any       | Any    | Any       |
| Community          | Huge      | Growing   | Medium | Growing   |
| Production Ready   | Excellent | Yes       | Yes    | Yes       |

**For this guide**: Express is chosen for flexibility + ecosystem.

---

## Resources

- Docs: https://expressjs.com
- Community: https://stackoverflow.com/questions/tagged/express
- Security: https://owasp.org/www-project-api-security/
- Packages: https://www.npmjs.com/

---

**Version**: 1.0 | **Date**: 2026
