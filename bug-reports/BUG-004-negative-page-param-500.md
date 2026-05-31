# BUG-004 — Negative `page` query parameter causes a 500 (negative skip)

| Field | Value |
|-------|-------|
| **Bug ID** | BUG-004 |
| **Severity** | Medium |
| **Priority** | Medium |
| **Status** | Open |
| **Found By** | Code analysis (needs runtime confirmation) |
| **Component** | `backend/controllers/controllerHelpers.js` → `parsePagination` |
| **Linked Requirement** | REQ-022 |
| **Linked Test Case** | TC-011 |
| **Environment** | Backend (Express 4 / Mongoose 8), Node 18+ |

## Summary
Pagination computes `skip = (page - 1) * limit` without clamping `page` to a positive value. A
negative `page` (e.g. `-1`) yields a **negative skip**, which MongoDB rejects, producing an
HTTP **500**. Any paginated list endpoint is affected.

## Preconditions
Backend running with at least one record in the collection.

## Steps to Reproduce
1. Request a negative page:
   ```bash
   curl -i "http://localhost:3000/api/alerts?page=-1&limit=10"
   ```
2. Observe the status code (repeat against `/api/crosswalks?page=-2`).

## Expected Result
Graceful handling: either clamp to page 1 and return **200**, or reject with **400**. No 5xx.

## Actual Result (predicted from code)
`parseInt("-1") = -1` (truthy), so `skip = (-1 - 1) * 10 = -20`. `Model.find().skip(-20)` triggers
a MongoServerError (negative skip is invalid) → global handler → **500**.

## Root Cause
```js
export function parsePagination(query, defaultLimit = 10) {
  const parsedPage = parseInt(query.page) || 1;   // negative passes the `||` guard
  const parsedLimit = parseInt(query.limit) || defaultLimit;
  const skip = (parsedPage - 1) * parsedLimit;     // can be negative
  return { parsedPage, parsedLimit, skip };
}
```
The `|| 1` fallback only protects against `0`/`NaN`, not negatives. `limit` has the same gap
(a negative limit would also misbehave).

## Suggested Fix
Clamp both values to sane lower bounds:
```js
const parsedPage  = Math.max(1, parseInt(query.page)  || 1);
const parsedLimit = Math.max(1, parseInt(query.limit) || defaultLimit);
const skip = (parsedPage - 1) * parsedLimit;       // now always >= 0
```
Optionally also cap `parsedLimit` to a maximum (e.g. 100) to prevent oversized scans.
