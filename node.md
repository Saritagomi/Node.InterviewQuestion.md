# Node.js Interview Questions & Answers (Most Asked)

A revision handbook of the questions that come up in almost every Node.js interview. Each answer is written the way you'd say it out loud in the interview — direct, with a quick example where it helps.

---

## 1. What is Node.js?

Node.js is a runtime that lets you run JavaScript outside the browser, built on Chrome's V8 engine. It's used mainly for building fast, scalable server-side applications and APIs. It uses an event-driven, non-blocking I/O model, which makes it lightweight and efficient for handling many connections at once.

---

## 2. Is Node.js single-threaded?

Yes, the JavaScript execution runs on a single thread. But Node handles I/O operations (like file reads, DB calls, network requests) asynchronously in the background using libuv and a thread pool. So one thread runs your code, and heavy I/O work happens in the background without blocking it.

---

## 3. What is the event loop?

The event loop is what allows Node.js to perform non-blocking operations even though it's single-threaded. It continuously checks the call stack and the callback queues, and pushes callbacks onto the stack when the stack is empty.

It runs in phases:

1. **Timers** – runs `setTimeout` and `setInterval` callbacks
2. **Pending callbacks** – runs some system-level callbacks
3. **Poll** – retrieves new I/O events and runs their callbacks
4. **Check** – runs `setImmediate` callbacks
5. **Close callbacks** – runs close events like `socket.on('close')`

Between each phase, Node runs microtasks — first `process.nextTick`, then Promise callbacks.

---

## 4. What is the difference between `setTimeout`, `setImmediate`, and `process.nextTick`?

- **`process.nextTick`** – runs immediately after the current operation, before the event loop continues. Highest priority.
- **`setImmediate`** – runs in the check phase, after the poll phase.
- **`setTimeout(fn, 0)`** – runs in the timers phase, with a minimum delay of about 1ms.

Order inside an I/O callback: `process.nextTick` → Promises → `setImmediate` → `setTimeout`.

---

## 5. What is the difference between synchronous and asynchronous code?

Synchronous code runs line by line and blocks the next line until the current one finishes. Asynchronous code doesn't wait — it starts an operation and moves on, then handles the result later through a callback, promise, or async/await. Node prefers async so the single thread isn't blocked.

---

## 6. Explain callbacks, Promises, and async/await.

- **Callback** – a function passed to another function, called when the task is done. Leads to nested "callback hell" when overused.
- **Promise** – an object representing a future value, with `.then()` and `.catch()`. Cleaner chaining.
- **async/await** – syntax built on Promises that lets you write async code that reads like synchronous code, using `try/catch` for errors.

```js
async function getUser() {
  try {
    const user = await fetchUser();
    return user;
  } catch (err) {
    console.error(err);
  }
}
```

---

## 7. What is the difference between `Promise.all`, `Promise.allSettled`, `Promise.race`, and `Promise.any`?

- **`Promise.all`** – resolves when all promises resolve; rejects immediately if any one fails.
- **`Promise.allSettled`** – waits for all, never rejects; returns the status of each.
- **`Promise.race`** – settles as soon as the first promise settles (resolve or reject).
- **`Promise.any`** – resolves with the first successful one; rejects only if all fail.

---

## 8. What is the difference between CommonJS and ES Modules?

- **CommonJS** uses `require()` and `module.exports`, loads synchronously. It's the default in Node.
- **ES Modules** use `import` and `export`, load asynchronously, and support tree-shaking. Enable them with `"type": "module"` in package.json or a `.mjs` extension.

---

## 9. What is middleware in Express?

Middleware is a function that runs during the request-response cycle. It has access to `req`, `res`, and `next`. It can run code, modify the request/response, end the cycle, or pass control to the next middleware with `next()`. Used for logging, authentication, body parsing, error handling, etc.

```js
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});
```

---

## 10. How does error handling work in Express?

You use a special error-handling middleware that takes four arguments `(err, req, res, next)` and place it last. Any error passed to `next(err)` jumps straight to it.

```js
app.use((err, req, res, next) => {
  res.status(err.status || 500).json({ message: err.message });
});
```

For async route handlers, wrap them so rejected promises reach the error handler:

```js
const wrap = fn => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);
```

---

## 11. What are streams in Node.js?

Streams let you read or write data in chunks instead of loading everything into memory at once — great for large files or network data.

Four types:
- **Readable** – read data (e.g. `fs.createReadStream`)
- **Writable** – write data (e.g. `fs.createWriteStream`)
- **Duplex** – both read and write (e.g. sockets)
- **Transform** – modify data as it passes through (e.g. gzip compression)

```js
fs.createReadStream('big.txt').pipe(fs.createWriteStream('copy.txt'));
```

---

## 12. What is backpressure in streams?

Backpressure happens when the writable side can't keep up with the readable side. `.pipe()` handles it automatically by pausing the source when the destination's buffer is full and resuming when it drains. Using `pipeline()` is preferred because it also handles errors and cleanup.

---

## 13. What is a Buffer?

A Buffer is a chunk of raw binary memory used to handle binary data like file or network bytes, stored outside V8's heap. Use `Buffer.from()` or `Buffer.alloc()`.

---

## 14. What is the difference between `process.env` and how do you manage config?

`process.env` holds environment variables. You keep config like DB URLs, API keys, and ports in environment variables (often loaded from a `.env` file using the `dotenv` package) instead of hardcoding them, so the same code runs in dev, staging, and production.

---

## 15. How does Node.js handle child processes / worker threads?

Since Node is single-threaded, CPU-heavy work can block it. To handle that:
- **`child_process`** – spawn separate processes (`spawn`, `exec`, `fork`).
- **`worker_threads`** – run JavaScript in parallel threads sharing memory, best for CPU-intensive tasks.
- **`cluster`** – fork multiple copies of the app across CPU cores to handle more requests.

---

## 16. How do you scale a Node.js application?

- Use the **cluster module** or **PM2** to run one process per CPU core.
- Keep the app **stateless** and store session data in Redis so any instance can serve any request.
- Scale horizontally by running multiple instances behind a **load balancer**.
- Use **caching** (Redis) and a **CDN** to reduce load.

---

## 17. What is the difference between `dependencies` and `devDependencies`?

- **`dependencies`** – packages needed to run the app in production.
- **`devDependencies`** – packages needed only during development, like testing and linting tools.

---

## 18. What is package-lock.json?

It records the exact version of every installed package, including nested dependencies, so installs are identical across all machines and environments. Always commit it.

---

## 19. What is the difference between `exec` and `spawn`?

- **`exec`** – runs a command, buffers the whole output, and returns it at once. Good for small output.
- **`spawn`** – streams the output as it comes. Good for large or continuous output.

---

## 20. How do you handle authentication in Node.js?

Most commonly with **JWT (JSON Web Tokens)**. On login, the server verifies credentials and returns a signed token. The client sends that token in the `Authorization` header on each request, and the server verifies the signature to authenticate the user — no server-side session needed. Use short-lived access tokens plus refresh tokens for security.

---

## 21. How do you secure a Node.js application?

- Validate and sanitize all user input.
- Use **helmet** to set secure HTTP headers.
- Use parameterized queries to prevent SQL injection.
- Hash passwords with **bcrypt**.
- Add **rate limiting** to prevent brute-force attacks.
- Keep secrets in environment variables.
- Keep dependencies updated (`npm audit`).
- Enforce HTTPS.

---

## 22. What is the N+1 query problem?

It's when you run one query to get a list, then one extra query per item — so N items cause N+1 total queries, which is slow. Fix it by using joins, `IN` queries, or a batching tool like DataLoader.

---

## 23. What is caching and how do you implement it?

Caching stores frequently used data so you don't recompute or refetch it. Common approaches:
- **In-memory** (fast but per-process).
- **Redis** (shared across instances).
- **HTTP caching** with `Cache-Control` and `ETag` headers.

Always set a TTL (expiry) and have an invalidation strategy.

---

## 24. How do you handle uncaught exceptions and unhandled rejections?

```js
process.on('uncaughtException', (err) => {
  console.error(err);
  process.exit(1); // let the process restart
});

process.on('unhandledRejection', (reason) => {
  console.error(reason);
});
```

Best practice is to log the error and let the process restart cleanly rather than keep running in a bad state.

---

## 25. What is the difference between `res.send`, `res.json`, and `res.end`?

- **`res.send`** – sends a response of any type (string, object, buffer).
- **`res.json`** – sends a JSON response and sets the content-type header automatically.
- **`res.end`** – ends the response without sending data (or with raw data).

---

## 26. How do you connect Node.js to a database?

Use a driver or ORM. For SQL: `pg` (Postgres), `mysql2`, or an ORM like Sequelize/Prisma. For MongoDB: the `mongodb` driver or Mongoose. Use **connection pooling** so connections are reused instead of opened per request.

---

## 27. What is the difference between SQL and NoSQL databases?

- **SQL** – structured, fixed schema, supports relations and joins, ACID compliant. Example: PostgreSQL, MySQL.
- **NoSQL** – flexible schema, scales horizontally, good for large or evolving data. Example: MongoDB, Redis.

---

## 28. How do you do graceful shutdown in Node.js?

On `SIGTERM`, stop accepting new requests, finish the in-flight ones, close DB connections, then exit. This avoids dropping active requests during a deploy or restart.

```js
process.on('SIGTERM', () => {
  server.close(() => process.exit(0));
});
```

---

## 29. How do you improve the performance of a Node.js app?

- Use async/non-blocking code, never block the event loop.
- Add caching (Redis).
- Use clustering to use all CPU cores.
- Optimize database queries and add indexes.
- Use pagination for large datasets.
- Compress responses (gzip) and use a CDN for static files.
- Stream large files instead of loading them into memory.

---

## 30. What is the difference between `require` and `import`?

- **`require`** – CommonJS, synchronous, can be called anywhere in the code.
- **`import`** – ES Modules, loaded at the top, static, allows tree-shaking. Needs ESM enabled.

---

## Quick Recall Table

Cover the right side and test yourself.

| Question | Key point |
|---|---|
| Node's I/O library | libuv |
| Runs before event loop continues | `process.nextTick` |
| Runs in check phase | `setImmediate` |
| Parallelism for CPU work | Worker threads |
| One process per core | cluster / PM2 |
| Handles backpressure | `.pipe()` / `pipeline()` |
| Transforms stream data | Transform stream |
| Exact dependency versions | package-lock.json |
| Fails fast on first rejection | `Promise.all` |
| Never rejects | `Promise.allSettled` |
| Secure headers | helmet |
| Password hashing | bcrypt |
| Fixes N+1 | joins / DataLoader |
| Stateless auth token | JWT |
| Shared cache across instances | Redis |
| Stops new requests on deploy | Graceful shutdown (SIGTERM) |

---

**How to revise:** read all questions once, mark the ones you can't answer instantly, drill those, then just run the recall table before the interview.
