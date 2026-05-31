# BUG-007 — README API contract does not match the implemented routes

| Field | Value |
|-------|-------|
| **Bug ID** | BUG-007 |
| **Severity** | Low |
| **Priority** | Low |
| **Status** | Open |
| **Found By** | Code analysis |
| **Component** | `AI-Smart-Crosswalk/smart-crosswalk/README.md` vs `backend/routes/*` |
| **Linked Requirement** | REQ-021 (contract/documentation accuracy) |
| **Linked Test Case** | TC-050 |
| **Environment** | Documentation |

## Summary
The README's "API Endpoints" table does not match the routes the backend actually exposes. A
developer or integrator following the README will call a wrong path and/or miss real endpoints.

## Discrepancies
1. **Wrong path** — README lists crosswalk stats as `GET /:id/alert-stats`, but the implemented
   route is `GET /api/crosswalks/:id/stats` (`backend/routes/crosswalkRoutes.js`).
2. **Undocumented endpoints** — the README omits several real routes:
   - `POST /api/leds/:id/command` (send ON/OFF/BLINK command, awaits ACK)
   - `PATCH /api/crosswalks/:id/camera` and `DELETE /api/crosswalks/:id/camera` (link/unlink camera)
   - `PATCH /api/crosswalks/:id/led` and `DELETE /api/crosswalks/:id/led` (link/unlink LED)
3. **Camera update path** — README shows `PATCH /:id/status` (correct), but does not note that
   plain `PATCH /api/cameras/:id` is **not** implemented (only the `/status` sub-path exists).

## Steps to Reproduce
1. Open the project README "API Endpoints" section.
2. Compare each documented path with `backend/routes/alertRoutes.js`, `crosswalkRoutes.js`,
   `cameraRoutes.js`, `ledRoutes.js`.

## Expected Result
Documented endpoints exactly match implemented routes (path, method).

## Actual Result
The mismatches listed above. For example, `GET /api/crosswalks/<id>/alert-stats` returns
`404 Not Found` (the route is `/stats`).

## Suggested Fix
Update the README endpoint table to reflect the actual routes, add the missing endpoints, and
correct the crosswalk stats path to `/:id/stats`. Consider generating the endpoint list from the
route files to keep docs and code in sync.
