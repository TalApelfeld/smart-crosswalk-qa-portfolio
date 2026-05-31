# BUG-003 — POST /api/cameras skips status validation; invalid status returns 500

| Field | Value |
|-------|-------|
| **Bug ID** | BUG-003 |
| **Severity** | Medium |
| **Priority** | Medium |
| **Status** | Open |
| **Found By** | Code analysis (needs runtime confirmation) |
| **Component** | `backend/routes/cameraRoutes.js`, `cameraControllers.createCamera` |
| **Linked Requirement** | REQ-011, REQ-021 |
| **Linked Test Case** | TC-034 |
| **Environment** | Backend (Express 4 / Mongoose 8), Node 18+ |

## Summary
The `validateCameraStatus` middleware is applied only to the **update** route
(`PATCH /api/cameras/:id/status`), not to the **create** route (`POST /api/cameras`). As a
result, creating a camera with an invalid `status` is not caught by the same friendly 400
validator; it falls through to Mongoose enum validation and (because of BUG-002) returns **500**.
This is an inconsistent contract: the same field is validated on update but not on create.

## Preconditions
Backend running.

## Steps to Reproduce
1. Create a camera with an invalid status:
   ```bash
   curl -i -X POST http://localhost:3000/api/cameras \
     -H "Content-Type: application/json" \
     -d '{"status":"broken"}'
   ```
2. Compare against the update path:
   ```bash
   curl -i -X PATCH http://localhost:3000/api/cameras/<id>/status \
     -H "Content-Type: application/json" -d '{"status":"broken"}'
   ```

## Expected Result
Both endpoints reject `"broken"` the same way: **400** with
`Invalid camera status: "broken". Must be one of: active, inactive, error`.

## Actual Result (predicted from code)
- `PATCH /:id/status` → **400** (validated). ✔
- `POST /` → reaches the model, Mongoose enum validation throws, and the global handler returns
  **500** (per BUG-002). ✘ (inconsistent with the update path)

## Root Cause
Route wiring in `cameraRoutes.js`:
```js
router.post("/", createCamera);                                   // no validator
router.patch("/:id/status", validateObjectId(), validateCameraStatus, updateCameraStatus); // validated
```
`createCamera` simply calls `Camera.create(req.body)`.

## Suggested Fix
Apply the existing validator to the create route too:
```js
router.post("/", validateCameraStatus, createCamera);
```
(Combined with the BUG-002 fix, this yields a consistent 400 on both paths.) Note: if cameras
should be creatable with no body (defaulting to `active`), adjust `validateCameraStatus` to allow
an absent `status` on create.
