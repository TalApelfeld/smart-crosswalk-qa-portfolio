# Test Cases — AI Smart Crosswalk System

40 manual test cases covering the backend API, business rules, error handling, real-time, and
the operator UI. Each case lists its linked requirement, design technique, steps, and expected
result. `Status` is **Not Executed** for this test-design baseline (see the test summary report
for the honesty note). Cases that traced to a defect during code analysis link a `BUG-NNN`.

**Conventions** — Base URL `http://localhost:3000/api`. `{validId}` / `{missingId}` etc. refer to
IDs documented in `reproducibility/test-data.md`. Curl equivalents are in
`reproducibility/curl-commands.md`.

Legend — Type: P=Positive, N=Negative, B=Boundary, F=Functional, UI, NF=Non-functional.
Technique: EP, BVA, DT (decision table), ST (state transition), EG (error guessing).

---

## Alerts

### TC-001 — List alerts with default pagination
- **Req:** REQ-001 | **Type:** P/F | **Technique:** — | **Priority:** High
- **Preconditions:** Seed data loaded (70 alerts).
- **Steps:** `GET /api/alerts`
- **Expected:** 200. Body has `success:true`, `count` ≤ 10, `total` = 70, `data` array sorted by `timestamp` desc, `pagination.currentPage=1`, `pagination.totalPages=7`, `pagination.hasMore=true`.
- **Status:** Not Executed

### TC-002 — Get a single alert by valid existing ID
- **Req:** REQ-008 | **Type:** P/F | **Priority:** High
- **Preconditions:** A known alert ID exists.
- **Steps:** `GET /api/alerts/{validAlertId}`
- **Expected:** 200, `data._id` equals the requested ID, `crosswalkId` populated when linked.
- **Status:** Not Executed

### TC-003 — Get alert with a malformed ID is rejected as 400
- **Req:** REQ-007 | **Type:** N | **Technique:** EP | **Priority:** High
- **Steps:** `GET /api/alerts/123`
- **Expected:** 400, message `Invalid id: 123`. DB is not queried.
- **Status:** Not Executed

### TC-004 — Get alert with a valid but non-existent ID returns 404
- **Req:** REQ-008 | **Type:** N | **Priority:** High
- **Steps:** `GET /api/alerts/64b0000000000000000000ff`
- **Expected:** 404, `success:false`, message `Alert not found`.
- **Status:** Not Executed

### TC-005 — Create alert with explicit valid danger level
- **Req:** REQ-003, REQ-006 | **Type:** P | **Technique:** EP | **Priority:** High
- **Steps:** `POST /api/alerts` body `{ "dangerLevel": "HIGH", "imageUrl": "https://x/y.jpg" }`
- **Expected:** 201, `data.dangerLevel="HIGH"`, `data._id` returned, `id` echoed.
- **Status:** Not Executed

### TC-006 — Create alert with an invalid danger level enum
- **Req:** REQ-006, REQ-021 | **Type:** N | **Technique:** EP | **Priority:** High
- **Steps:** `POST /api/alerts` body `{ "dangerLevel": "EXTREME" }`
- **Expected:** **400** with a validation message (client error).
- **Status:** Not Executed | **Linked Defect:** BUG-002 (returns 500 instead of 400)

### TC-007 — Danger level derived from confidence (decision table)
- **Req:** REQ-003 | **Type:** P/B | **Technique:** DT/BVA | **Priority:** High
- **Steps:** `POST /api/alerts` once per row, body `{ "confidence": <value> }`:
  | confidence | expected dangerLevel |
  |-----------|----------------------|
  | (omitted) | MEDIUM |
  | 0.39 | LOW |
  | 0.40 | MEDIUM |
  | 0.69 | MEDIUM |
  | 0.70 | HIGH |
- **Expected:** 201 each; `data.dangerLevel` matches the table.
- **Status:** Not Executed

### TC-008 — Explicit danger level overrides confidence
- **Req:** REQ-004 | **Type:** P | **Priority:** Medium
- **Steps:** `POST /api/alerts` body `{ "dangerLevel": "LOW", "confidence": 0.95 }`
- **Expected:** 201, `data.dangerLevel="LOW"` (not HIGH).
- **Status:** Not Executed

### TC-009 — Pagination: explicit page and limit
- **Req:** REQ-001, REQ-022 | **Type:** B | **Technique:** BVA | **Priority:** Medium
- **Steps:** `GET /api/alerts?page=2&limit=5`
- **Expected:** 200, `count`=5, `pagination.currentPage=2`, `data` is the 6th–10th newest alerts.
- **Status:** Not Executed

### TC-010 — Pagination: page beyond last returns empty data
- **Req:** REQ-022 | **Type:** B | **Technique:** BVA | **Priority:** Medium
- **Steps:** `GET /api/alerts?page=999&limit=10`
- **Expected:** 200, `count=0`, `data=[]`, `hasMore=false`, `total` still correct.
- **Status:** Not Executed

### TC-011 — Pagination: negative page is handled gracefully
- **Req:** REQ-022 | **Type:** N/B | **Technique:** BVA/EG | **Priority:** Medium
- **Steps:** `GET /api/alerts?page=-1&limit=10`
- **Expected:** A graceful response (200 with first page, or 400) — **no server error**.
- **Status:** Not Executed | **Linked Defect:** BUG-004 (negative skip → 500)

### TC-012 — Pagination: limit=0 falls back to default
- **Req:** REQ-022 | **Type:** B | **Technique:** BVA | **Priority:** Low
- **Steps:** `GET /api/alerts?limit=0`
- **Expected:** 200, behaves as default limit 10 (`0 || 10` ⇒ 10).
- **Status:** Not Executed

### TC-013 — Filter alerts by danger level
- **Req:** REQ-002 | **Type:** P/F | **Priority:** Medium
- **Steps:** `GET /api/alerts?dangerLevel=HIGH`
- **Expected:** 200, every item in `data` has `dangerLevel="HIGH"`.
- **Status:** Not Executed

### TC-014 — Filter alerts by crosswalkId
- **Req:** REQ-002 | **Type:** P/F | **Priority:** Medium
- **Steps:** `GET /api/alerts?crosswalkId={validCrosswalkId}`
- **Expected:** 200, every item's `crosswalkId._id` equals the filter value.
- **Status:** Not Executed

### TC-015 — Alert statistics totals
- **Req:** REQ-001 | **Type:** P/F | **Priority:** Medium
- **Steps:** `GET /api/alerts/stats`
- **Expected:** 200, `data` = `{ total, low, medium, high }`, and `low+medium+high = total`.
- **Status:** Not Executed

### TC-016 — Update an alert's danger level
- **Req:** REQ-006 | **Type:** P | **Priority:** Medium
- **Steps:** `PATCH /api/alerts/{validAlertId}` body `{ "dangerLevel": "LOW" }`
- **Expected:** 200, `data.dangerLevel="LOW"`, returns populated alert.
- **Status:** Not Executed

### TC-017 — Delete an alert
- **Req:** REQ-014 | **Type:** P | **Priority:** Medium
- **Preconditions:** An alert exists (ideally with a Cloudinary `imageUrl`).
- **Steps:** `DELETE /api/alerts/{validAlertId}` then `GET /api/alerts/{sameId}`
- **Expected:** Delete → 200 `Alert deleted successfully`; subsequent GET → 404. If image was a Cloudinary URL, the asset is removed (verify in Cloudinary console).
- **Status:** Not Executed

---

## Crosswalks

### TC-018 — Create a crosswalk with valid location
- **Req:** REQ-009 | **Type:** P | **Priority:** High
- **Steps:** `POST /api/crosswalks` body `{ "location": { "city": "Haifa", "street": "Herzl", "number": "10" } }`
- **Expected:** 201, `data.location` matches input, `cameraId`/`ledId` null.
- **Status:** Not Executed

### TC-019 — Create a crosswalk with missing location fields is rejected
- **Req:** REQ-009 | **Type:** N | **Technique:** EP | **Priority:** High
- **Steps:** `POST /api/crosswalks` body `{ "location": { "city": "Haifa" } }`
- **Expected:** 400, message lists missing required fields.
- **Status:** Not Executed

### TC-020 — Search: empty query rejected
- **Req:** REQ-010 | **Type:** N/B | **Technique:** BVA | **Priority:** Medium
- **Steps:** `GET /api/crosswalks/search` (no `q`)
- **Expected:** 400, message `Query parameter "q" is required`.
- **Status:** Not Executed

### TC-021 — Search: single-character query rejected (lower boundary)
- **Req:** REQ-010 | **Type:** B | **Technique:** BVA | **Priority:** Medium
- **Steps:** `GET /api/crosswalks/search?q=a`
- **Expected:** 400, message `Query parameter "q" must be at least 2 characters`.
- **Status:** Not Executed

### TC-022 — Search: valid two-character query (on boundary)
- **Req:** REQ-010 | **Type:** P/B | **Technique:** BVA | **Priority:** Medium
- **Steps:** `GET /api/crosswalks/search?q=Di`
- **Expected:** 200, `data` contains crosswalks whose city/street/number matches "Di" (e.g., Dizengoff), case-insensitive.
- **Status:** Not Executed

### TC-023 — Search: regex metacharacter query must not crash the server
- **Req:** REQ-010, REQ-021 | **Type:** N | **Technique:** EG | **Priority:** High
- **Steps:** `GET /api/crosswalks/search?q=%28`  (the character `(` )
- **Expected:** A graceful 200 (treating it as a literal search) or 400 — **never a 500/crash**.
- **Status:** Not Executed | **Linked Defect:** BUG-001 (unsanitized RegExp → 500)

### TC-024 — Get crosswalk by ID with devices populated
- **Req:** REQ-008, REQ-013 | **Type:** P | **Priority:** Medium
- **Steps:** `GET /api/crosswalks/{validCrosswalkId}`
- **Expected:** 200, `data.cameraId` and `data.ledId` are populated objects (or null), `data.location` present.
- **Status:** Not Executed

### TC-025 — Crosswalk alert stats aggregation
- **Req:** REQ-001 | **Type:** P/F | **Priority:** Medium
- **Steps:** `GET /api/crosswalks/{validCrosswalkId}/stats`
- **Expected:** 200, `data` has `total`, `byDangerLevel:{HIGH,MEDIUM,LOW}`, `last24Hours`, `last7Days`, `last30Days` (numbers).
- **Status:** Not Executed

### TC-026 — Get crosswalk alerts filtered by danger level
- **Req:** REQ-002 | **Type:** P/F | **Priority:** Medium
- **Steps:** `GET /api/crosswalks/{validCrosswalkId}/alerts?dangerLevel=high`
- **Expected:** 200, all returned alerts are HIGH (controller upper-cases the value), default page limit 50.
- **Status:** Not Executed

### TC-027 — Get crosswalk alerts with an invalid date filter must not crash
- **Req:** REQ-021 | **Type:** N | **Technique:** EG | **Priority:** Medium
- **Steps:** `GET /api/crosswalks/{validCrosswalkId}/alerts?startDate=not-a-date`
- **Expected:** A graceful 200 (ignoring the bad filter) or 400 — **never a 500**.
- **Status:** Not Executed | **Linked Defect:** BUG-005 (Invalid Date → cast error → 500)

### TC-028 — Link a camera to a crosswalk
- **Req:** REQ-013 | **Type:** P | **Technique:** ST | **Priority:** Medium
- **Steps:** `PATCH /api/crosswalks/{validCrosswalkId}/camera` body `{ "cameraId": "{validCameraId}" }`
- **Expected:** 200, `Camera linked successfully`, `data.cameraId` populated.
- **Status:** Not Executed

### TC-029 — Link a non-existent camera returns 404
- **Req:** REQ-013 | **Type:** N | **Priority:** Medium
- **Steps:** `PATCH /api/crosswalks/{validCrosswalkId}/camera` body `{ "cameraId": "64b0000000000000000000ff" }`
- **Expected:** 404, `Camera not found`.
- **Status:** Not Executed

### TC-030 — Link without cameraId is rejected
- **Req:** REQ-013 | **Type:** N | **Priority:** Medium
- **Steps:** `PATCH /api/crosswalks/{validCrosswalkId}/camera` body `{}`
- **Expected:** 400, `cameraId is required`.
- **Status:** Not Executed

### TC-031 — Unlink a camera from a crosswalk
- **Req:** REQ-013 | **Type:** P | **Technique:** ST | **Priority:** Medium
- **Steps:** `DELETE /api/crosswalks/{validCrosswalkId}/camera`
- **Expected:** 200, `Camera unlinked successfully`, `data.cameraId` null/absent.
- **Status:** Not Executed

### TC-032 — Link and unlink an LED
- **Req:** REQ-013 | **Type:** P | **Technique:** ST | **Priority:** Medium
- **Steps:** `PATCH /api/crosswalks/{id}/led` `{ "ledId": "{validLedId}" }`, then `DELETE /api/crosswalks/{id}/led`
- **Expected:** Link → 200 `LED linked successfully`; Unlink → 200 `LED unlinked successfully`.
- **Status:** Not Executed

---

## Cameras

### TC-033 — Create a camera with valid status
- **Req:** REQ-011 | **Type:** P | **Technique:** EP | **Priority:** Medium
- **Steps:** `POST /api/cameras` body `{ "status": "active" }`
- **Expected:** 201, `data.status="active"`.
- **Status:** Not Executed

### TC-034 — Create a camera with an invalid status enum
- **Req:** REQ-011, REQ-021 | **Type:** N | **Technique:** EP | **Priority:** High
- **Steps:** `POST /api/cameras` body `{ "status": "broken" }`
- **Expected:** **400** validation error (consistent with the PATCH-status path).
- **Status:** Not Executed | **Linked Defect:** BUG-003 (no validation on create → 500)

### TC-035 — Update camera status through valid transitions
- **Req:** REQ-011 | **Type:** P | **Technique:** ST | **Priority:** Medium
- **Steps:** `PATCH /api/cameras/{id}/status` with `active` → `inactive` → `error` in sequence.
- **Expected:** 200 each; `data.status` reflects each new value.
- **Status:** Not Executed

### TC-036 — Update camera status with invalid value rejected
- **Req:** REQ-011 | **Type:** N | **Technique:** EP | **Priority:** Medium
- **Steps:** `PATCH /api/cameras/{id}/status` body `{ "status": "on" }`
- **Expected:** 400, message lists valid statuses.
- **Status:** Not Executed

### TC-037 — Delete a camera that is linked to a crosswalk is blocked
- **Req:** REQ-012 | **Type:** N | **Priority:** High
- **Preconditions:** Camera is linked to a crosswalk (seed: camera1).
- **Steps:** `DELETE /api/cameras/{linkedCameraId}`
- **Expected:** 400, `Cannot delete camera linked to crosswalk`; camera still exists.
- **Status:** Not Executed

### TC-038 — Delete an unlinked camera succeeds
- **Req:** REQ-012 | **Type:** P | **Priority:** Medium
- **Preconditions:** Create a fresh, unlinked camera.
- **Steps:** `DELETE /api/cameras/{unlinkedCameraId}`
- **Expected:** 200, `Camera deleted successfully`.
- **Status:** Not Executed

---

## LEDs

### TC-039 — Create an LED
- **Req:** REQ-013 | **Type:** P | **Priority:** Low
- **Steps:** `POST /api/leds` body `{}`
- **Expected:** 201, `data._id` present, timestamps set.
- **Status:** Not Executed

### TC-040 — LED command with missing/invalid state is rejected
- **Req:** REQ-015 | **Type:** N | **Technique:** EP | **Priority:** Medium
- **Steps:** `POST /api/leds/{validLedId}/command` body `{ "state": "PURPLE" }`
- **Expected:** 400, `state is required and must be one of: ON, OFF, BLINK`.
- **Status:** Not Executed

### TC-041 — LED command with valid state but no subscribed device
- **Req:** REQ-015 | **Type:** N | **Priority:** Medium
- **Preconditions:** No LED device subscribed over Socket.IO.
- **Steps:** `POST /api/leds/{validLedId}/command` body `{ "state": "ON" }`
- **Expected:** Error response (400 `No LED clients subscribed...` or 504 ACK timeout) — not a 2xx.
- **Status:** Not Executed

### TC-042 — Delete an LED linked to a crosswalk is blocked
- **Req:** REQ-012 | **Type:** N | **Priority:** High
- **Preconditions:** LED linked to a crosswalk (seed: led1).
- **Steps:** `DELETE /api/leds/{linkedLedId}`
- **Expected:** 400, `Cannot delete LED linked to crosswalk`.
- **Status:** Not Executed

---

## System / Non-functional

### TC-043 — Health check reports database state
- **Req:** REQ-020 | **Type:** NF | **Priority:** Medium
- **Steps:** `GET /api/health`
- **Expected:** 200; when DB connected `success:true`, `database.connected:true`, `database.state:1`, `timestamp` present.
- **Status:** Not Executed

### TC-044 — Unknown route returns 404 JSON
- **Req:** REQ-008 | **Type:** N | **Priority:** Low
- **Steps:** `GET /api/does-not-exist`
- **Expected:** 404, `success:false`, message `Not Found - /api/does-not-exist`.
- **Status:** Not Executed

---

## Frontend / UI

### TC-045 — Alerts page filters and empty states
- **Req:** REQ-018 | **Type:** UI | **Priority:** Medium
- **Steps:** Open `/alerts`; expand FilterBar; set Danger Level = "HIGH"; set a Crosswalk search with no matches; clear filters.
- **Expected:** Danger filter shows only HIGH alerts; an "Active" badge appears while filtering; no-match state shows icon 🔍, title "No Matching Alerts", message "Try adjusting your filters to see more results."; "Clear All" resets to the full list.
- **Status:** Not Executed

### TC-046 — Alerts page "Load More" pagination
- **Req:** REQ-018 | **Type:** UI | **Priority:** Medium
- **Steps:** With >10 alerts, scroll to the list bottom and click "Load More".
- **Expected:** The next page of alerts appends to the list; the button hides when `hasMore` is false.
- **Status:** Not Executed

### TC-047 — Realtime HIGH alert raises an error toast
- **Req:** REQ-017, REQ-016 | **Type:** UI/Real-time | **Priority:** Medium
- **Steps:** With `/alerts` open, `POST /api/alerts` `{ "dangerLevel": "HIGH", "crosswalkId": "{id}" }`.
- **Expected:** An error toast `ALARM HIGH | {location} | {timestamp}` (≈7s); the new alert prepends to the list and stat counters increment without a manual refresh.
- **Status:** Not Executed

### TC-048 — Realtime LOW alert is silent
- **Req:** REQ-017 | **Type:** UI/Real-time | **Priority:** Low
- **Steps:** `POST /api/alerts` `{ "dangerLevel": "LOW", ... }` with the UI open.
- **Expected:** No toast appears; list/stats still update.
- **Status:** Not Executed

### TC-049 — Dashboard System Status reflects real service health
- **Req:** REQ-019 | **Type:** UI/NF | **Priority:** Medium
- **Steps:** Stop the backend (or disconnect the DB), then load `/`. Observe the System Status card.
- **Expected:** The card shows API/Database/YOLO as down/disconnected when they actually are.
- **Status:** Not Executed | **Linked Defect:** BUG-006 (statuses hard-coded "online"/"connected")

### TC-050 — README API contract matches the implemented routes
- **Req:** REQ-021 (contract) | **Type:** Documentation | **Priority:** Low
- **Steps:** Compare the README "API Endpoints" table to the actual routes in `backend/routes/*`.
- **Expected:** Documented paths exist as written (e.g., crosswalk alert-stats path).
- **Status:** Not Executed | **Linked Defect:** BUG-007 (README says `/:id/alert-stats`, route is `/:id/stats`; command & link endpoints undocumented)
