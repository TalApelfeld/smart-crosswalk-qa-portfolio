# BUG-002 — Mongoose validation errors return HTTP 500 instead of 400

| Field | Value |
|-------|-------|
| **Bug ID** | BUG-002 |
| **Severity** | Medium |
| **Priority** | High |
| **Status** | Open |
| **Found By** | Code analysis (needs runtime confirmation) |
| **Component** | `backend/middleware/errorMiddleware.js` → `errorHandler` |
| **Linked Requirement** | REQ-021, REQ-006 |
| **Linked Test Case** | TC-006 |
| **Environment** | Backend (Express 4 / Mongoose 8), Node 18+ |

## Summary
When a request fails Mongoose schema validation (e.g., an invalid `dangerLevel` enum on an
alert), the resulting `ValidationError`/`CastError` reaches the global error handler with no HTTP
status set. The handler then defaults the response to **500 Internal Server Error**, even though
this is a **client** error that should be **400 Bad Request**. Clients cannot distinguish "you
sent bad data" from "the server broke".

## Preconditions
Backend running.

## Steps to Reproduce
1. Send an alert with an invalid enum value:
   ```bash
   curl -i -X POST http://localhost:3000/api/alerts \
     -H "Content-Type: application/json" \
     -d '{"dangerLevel":"EXTREME"}'
   ```
2. Observe the status code.

## Expected Result
**400 Bad Request** with a message identifying the invalid field/value.

## Actual Result (predicted from code)
**500 Internal Server Error** with the raw Mongoose validation message (and a stack trace when
`NODE_ENV` is not `production`).

## Root Cause
The error handler only preserves a status that a controller set earlier; otherwise it falls back
to 500:
```js
const statusCode = res.statusCode === 200 ? 500 : res.statusCode;
```
Controllers create documents directly (`Alert.create(...)`, `Crosswalk.create(...)`, etc.) and
pass any thrown error to `next(error)` without setting `res.status(400)`. Mongoose
`ValidationError` and `CastError` are therefore treated as server errors.

## Impact / Scope
Affects every write path that relies on schema validation without an explicit guard — e.g.
`POST/PATCH /api/alerts` with a bad enum, and `POST /api/cameras` with a bad status (see BUG-003).

## Suggested Fix
Detect Mongoose error types in the global handler and map them to 4xx:
```js
let status = res.statusCode === 200 ? 500 : res.statusCode;
if (err.name === "ValidationError") status = 400;
if (err.name === "CastError") status = 400;          // malformed value/id
if (err.code === 11000) status = 409;                // duplicate key
res.status(status).json({ success: false, message: err.message, ... });
```
