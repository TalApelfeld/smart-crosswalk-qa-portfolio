# BUG-005 — Invalid date in crosswalk alert filter causes a 500

| Field | Value |
|-------|-------|
| **Bug ID** | BUG-005 |
| **Severity** | Medium |
| **Priority** | Medium |
| **Status** | Open |
| **Found By** | Code analysis (needs runtime confirmation) |
| **Component** | `backend/controllers/crosswalkControllers.js` → `getCrosswalkAlerts` |
| **Linked Requirement** | REQ-021 |
| **Linked Test Case** | TC-027 |
| **Environment** | Backend (Express 4 / Mongoose 8), Node 18+ |

## Summary
The "alerts for a crosswalk" endpoint accepts `startDate`/`endDate` query params and passes them
straight into `new Date(...)`. A non-parseable value produces an **Invalid Date**, which Mongoose
cannot cast for the `timestamp` comparison, yielding an HTTP **500** instead of a clean 400 or an
ignored filter.

## Preconditions
Backend running; a valid crosswalk ID `{id}`.

## Steps to Reproduce
1. Request with a malformed date:
   ```bash
   curl -i "http://localhost:3000/api/crosswalks/{id}/alerts?startDate=not-a-date"
   ```
2. Observe the status code.

## Expected Result
The bad date is ignored (return **200** with unfiltered results) or rejected with **400**. No 5xx.

## Actual Result (predicted from code)
`new Date("not-a-date")` → `Invalid Date`; the query `{ timestamp: { $gte: Invalid Date } }`
fails to cast → **500**.

## Root Cause
```js
if (startDate || endDate) {
  query.timestamp = {};
  if (startDate) query.timestamp.$gte = new Date(startDate);   // no validity check
  if (endDate)   query.timestamp.$lte = new Date(endDate);
}
```

## Suggested Fix
Validate the parsed date before using it:
```js
const parseDate = (v) => {
  const d = new Date(v);
  return isNaN(d.getTime()) ? null : d;
};
const gte = parseDate(startDate);
const lte = parseDate(endDate);
if (gte || lte) {
  query.timestamp = {};
  if (gte) query.timestamp.$gte = gte;
  if (lte) query.timestamp.$lte = lte;
}
```
(Reject with 400 if a provided date is non-null-but-invalid, per the API's preferred contract.)
