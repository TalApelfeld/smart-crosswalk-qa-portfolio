# BUG-001 — Crosswalk search with regex metacharacters crashes the server (500)

| Field | Value |
|-------|-------|
| **Bug ID** | BUG-001 |
| **Severity** | High |
| **Priority** | High |
| **Status** | Open |
| **Found By** | Code analysis (needs runtime confirmation) |
| **Component** | `backend/controllers/crosswalkControllers.js` → `searchCrosswalks` |
| **Linked Requirement** | REQ-010, REQ-021 |
| **Linked Test Case** | TC-023 |
| **Environment** | Backend (Express 4 / Mongoose 8), any Node 18+, any OS |

## Summary
The crosswalk search endpoint builds a regular expression directly from the user-supplied query
string without escaping it. A query containing a regex metacharacter that forms an invalid
pattern (e.g. `(`, `[`, `*?`) throws a `SyntaxError` while compiling the `RegExp`. The error is
unhandled at that point and surfaces as an HTTP **500 Internal Server Error**.

## Preconditions
- Backend running and reachable.
- The request passes `validateSearchQuery` (so `q` is present and length ≥ 2).

## Steps to Reproduce
1. Send: `GET /api/crosswalks/search?q=((`
   ```bash
   curl -i "http://localhost:3000/api/crosswalks/search?q=(("
   ```
2. Observe the response status and body.

## Expected Result
The server treats `q` as a literal search term (escaping metacharacters) and returns **200** with
matching/empty results — or rejects it with a **400**. It must **not** return a 5xx or crash.

## Actual Result (predicted from code)
`new RegExp("((", "i")` throws `SyntaxError: Invalid regular expression`, which propagates to the
global error handler and returns **500 Internal Server Error**.

## Root Cause
`searchCrosswalks` constructs the regex from raw input:
```js
const searchRegex = new RegExp(q, "i");   // q is unescaped user input
```
`validateSearchQuery` only checks presence and length — not regex validity — so malformed
patterns reach `new RegExp`.

## Additional Risk
Even *valid* but pathological patterns (e.g. nested quantifiers) enable **ReDoS** (Regular
Expression Denial of Service) and unindexed full-collection scans, since the user controls the
matching engine. This raises the severity beyond a single 500.

## Suggested Fix
Escape the input before building the regex (treat it as a literal substring), e.g.:
```js
const escaped = q.replace(/[.*+?^${}()|[\]\\]/g, "\\$&");
const searchRegex = new RegExp(escaped, "i");
```
Optionally wrap regex construction in try/catch and respond 400 on failure, and/or use a MongoDB
text index instead of a regex scan.
