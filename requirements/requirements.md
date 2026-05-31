# Requirements — AI Smart Crosswalk System

Each requirement is **testable** and derived directly from the real source code (file references
in the *Source* column). Requirements are the anchor of traceability: see `traceability/rtm.md`
for the REQ → TC → BUG mapping.

| ID | Requirement | Category | Priority | Source |
|----|-------------|----------|----------|--------|
| REQ-001 | `GET /api/alerts` SHALL return alerts paginated and sorted by `timestamp` descending, with a `pagination` object (`currentPage`, `totalPages`, `total`, `hasMore`). Default `limit` is 10. | Functional | High | `alertControllers.getAllAlerts`, `controllerHelpers` |
| REQ-002 | `GET /api/alerts` SHALL support filtering by `dangerLevel` and `crosswalkId` query params. | Functional | Medium | `alertControllers.getAllAlerts` |
| REQ-003 | `POST /api/alerts` SHALL create an alert; when `dangerLevel` is omitted it SHALL derive it from `confidence` (≥0.7→HIGH, ≥0.4→MEDIUM, else LOW), defaulting to MEDIUM when confidence is absent. | Business rule | High | `alertControllers.createAlert`, `utils/dangerLevel.js` |
| REQ-004 | An explicitly provided `dangerLevel` SHALL override any confidence-derived value. | Business rule | Medium | `alertControllers.createAlert` |
| REQ-005 | `POST /api/alerts` with a `location` (city+street+number) and no `crosswalkId` SHALL find an existing matching crosswalk or create a new one and link the alert to it. | Integration | Medium | `alertControllers.findOrCreateCrosswalk` |
| REQ-006 | `dangerLevel` SHALL be constrained to the enum `LOW | MEDIUM | HIGH`; invalid values SHALL be rejected as a client error (HTTP 400). | Validation | High | `models/Alert.js` |
| REQ-007 | Any `:id` route parameter SHALL be a valid Mongo ObjectId; otherwise the API SHALL respond 400 without hitting the database. | Validation | High | `middleware/common/validateObjectId.js` |
| REQ-008 | Requesting a non-existent but valid resource ID SHALL respond 404 with a descriptive message. | Functional | High | `controllerHelpers.notFound` |
| REQ-009 | `POST /api/crosswalks` SHALL require `location.city`, `location.street`, and `location.number`; missing fields SHALL respond 400. | Validation | High | `middleware/crosswalks/validateCrosswalk.js` |
| REQ-010 | `GET /api/crosswalks/search` SHALL require query param `q` of length ≥ 2 and SHALL match against city/street/number; invalid `q` SHALL respond 400 and SHALL NOT crash the server. | Functional | High | `validateSearchQuery`, `crosswalkControllers.searchCrosswalks` |
| REQ-011 | Camera `status` SHALL be one of `active | inactive | error` on **all** write paths (create and update); invalid values SHALL respond 400. | Validation | High | `models/Camera.js`, `validateCameraStatus` |
| REQ-012 | A Camera or LED that is currently linked to a crosswalk SHALL NOT be deletable; the API SHALL respond 400 with an explanatory message. | Business rule | High | `cameraControllers.deleteCamera`, `ledControllers.deleteLED` |
| REQ-013 | A crosswalk SHALL support linking and unlinking a Camera and an LED; linking a non-existent device SHALL respond 404. | Functional | Medium | `crosswalkControllers.linkCamera/unlinkCamera/linkLED/unlinkLED` |
| REQ-014 | Deleting an alert whose `imageUrl` is a Cloudinary asset SHALL also remove that asset from Cloudinary. | Integration | Medium | `alertControllers.deleteAlertById`, `utils/cloudinary.js` |
| REQ-015 | `POST /api/leds/:id/command` SHALL require `state ∈ {ON, OFF, BLINK}`, deliver the command to a subscribed device over Socket.IO, and respond 504 if no ACK is received before timeout. | Functional | Medium | `ledControllers.sendLEDCommand`, `socket/index.js` |
| REQ-016 | Creating a new alert SHALL broadcast an `alert:new` event to dashboard and crosswalk subscribers in real time. | Real-time | Medium | `socket/emitAlertRealtime` |
| REQ-017 | The frontend SHALL raise a toast for HIGH (error) and MEDIUM (warning) realtime alerts and SHALL stay silent for LOW alerts. | UI / Real-time | Medium | `frontend/.../SocketBridge.jsx` |
| REQ-018 | The Alerts page SHALL provide filters for danger level, crosswalk search, and date range, show correct empty-state messages, and paginate via "Load More". | UI | Medium | `frontend/.../pages/Alerts.jsx`, `FilterBar.jsx` |
| REQ-019 | The Dashboard System Status card SHALL reflect the **actual** health of the API, database, and YOLO detection services. | UI / Non-functional | Medium | `frontend/.../pages/Dashboard.jsx` |
| REQ-020 | `GET /api/health` SHALL report the live database connection state. | Non-functional | Medium | `backend/server.js` |
| REQ-021 | The API SHALL return HTTP status codes that correctly distinguish client errors (4xx) from server errors (5xx). | Non-functional | High | `middleware/errorMiddleware.js` |
| REQ-022 | Pagination SHALL handle out-of-range and non-positive `page`/`limit` values gracefully (no server error). | Robustness | Medium | `controllerHelpers.parsePagination` |
