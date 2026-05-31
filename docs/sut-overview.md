# System Under Test (SUT) Overview — AI Smart Crosswalk

This document maps the system being tested so that every requirement, test case, and bug
report can be traced back to a concrete, real part of the application.

The SUT is a **real, working full-stack project**: an AI-powered crosswalk safety system that
detects pedestrians with computer vision, assesses a danger level, raises alerts, and manages
LED crosswalk signals.

---

## 1. Architecture

| Layer | Technology | Role |
|-------|------------|------|
| Backend API | Node.js, Express 4, Mongoose 8 | REST API + business logic |
| Database | MongoDB (Atlas in prod) | Persists Alerts, Crosswalks, Cameras, LEDs |
| Real-time | Socket.IO 4 | Pushes new alerts to dashboards; sends LED commands |
| AI module | Python, YOLOv8 pose estimation | Detects pedestrians, POSTs alerts to the API |
| Media | Cloudinary | Stores/deletes detection images |
| Frontend | React 18, Vite 5, TanStack Query 5, Tailwind 4 | Operator dashboard |

**High-level data flow**

```
Camera frame ──► Python YOLOv8 ──► POST /api/alerts (imageUrl + confidence + location)
                                       │
                                       ├─► danger level derived from confidence
                                       ├─► find-or-create Crosswalk from location
                                       ├─► save Alert to MongoDB
                                       └─► Socket.IO "alert:new" ──► Dashboard toast + live list
```

---

## 2. Data Models (Mongoose schemas)

**Alert** (`backend/models/Alert.js`)
- `dangerLevel`: enum `LOW | MEDIUM | HIGH`, optional, default `MEDIUM`
- `crosswalkId`: ObjectId ref Crosswalk, optional, default `null` (YOLO-only alerts allowed)
- `imageUrl`: String, optional
- `timestamp`: Date, default `Date.now`, required
- `timestamps: true` (adds `createdAt`, `updatedAt`)

**Crosswalk** (`backend/models/Crosswalk.js`)
- `location.city`, `location.street`, `location.number`: all **required**, trimmed strings
- `cameraId`: ObjectId ref Camera, optional, default `null`
- `ledId`: ObjectId ref LED, optional, default `null`

**Camera** (`backend/models/Camera.js`)
- `status`: enum `active | inactive | error`, required, default `active`

**LED** (`backend/models/LED.js`)
- No fields beyond `timestamps` — represents a physical lighting unit.

---

## 3. API Surface

Base URL: `http://localhost:3000/api`

### Alerts — `/api/alerts`
| Method | Path | Notes |
|--------|------|-------|
| GET | `/` | Paginated, sorted by `timestamp` desc. Filters: `dangerLevel`, `crosswalkId`, `page`, `limit` (default 10) |
| GET | `/stats` | `{ total, low, medium, high }` |
| GET | `/:id` | `validateObjectId` → 400 if not a valid ObjectId; 404 if missing |
| POST | `/` | Create alert; derives danger level from `confidence` if not given; find-or-creates crosswalk from `location` |
| PATCH | `/:id` | Update; `runValidators: true` |
| DELETE | `/:id` | Deletes alert; also deletes Cloudinary asset if `imageUrl` is a Cloudinary URL |

### Crosswalks — `/api/crosswalks`
| Method | Path | Notes |
|--------|------|-------|
| GET | `/` | Paginated, sorted by `createdAt` desc, populates camera + LED |
| GET | `/search?q=` | `validateSearchQuery`: `q` required, min length 2. Builds a **RegExp from `q`** over city/street/number |
| GET | `/stats` | `{ total }` |
| GET | `/:id` | Single crosswalk with devices populated |
| GET | `/:id/alerts` | Alerts for a crosswalk. Filters: `startDate`, `endDate`, `dangerLevel`, `sortBy` (`oldest`/`danger`), pagination (default limit 50) |
| GET | `/:id/stats` | Aggregated alert stats: total, byDangerLevel, last24h/7d/30d |
| POST | `/` | `validateCreateCrosswalk`: requires location.city/street/number |
| PATCH | `/:id` | Update crosswalk |
| DELETE | `/:id` | Delete crosswalk |
| PATCH | `/:id/camera` | `validateLinkCamera`: link camera (404 if camera missing) |
| DELETE | `/:id/camera` | Unlink camera |
| PATCH | `/:id/led` | `validateLinkLED`: link LED |
| DELETE | `/:id/led` | Unlink LED |

### Cameras — `/api/cameras`
| Method | Path | Notes |
|--------|------|-------|
| GET | `/` | All cameras |
| GET | `/:id` | Single |
| POST | `/` | Create — **no status validation middleware applied** (see BUG-003) |
| PATCH | `/:id/status` | `validateCameraStatus` → 400 on invalid enum |
| DELETE | `/:id` | 400 if linked to a crosswalk; else delete |

### LEDs — `/api/leds`
| Method | Path | Notes |
|--------|------|-------|
| GET | `/` | All LEDs |
| GET | `/:id` | Single |
| POST | `/` | Create |
| POST | `/:id/command` | Requires `state ∈ ON|OFF|BLINK`; emits Socket.IO command, waits for ACK; 504 on timeout |
| DELETE | `/:id` | 400 if linked to a crosswalk; else delete |

### System — `/api/health`, `/`
- `GET /api/health` → reports `mongoose.connection.readyState` (1 = connected). Returns HTTP 200 in both connected and disconnected states (only the `success` field differs).
- `GET /` → API welcome + endpoint index.

---

## 4. Key Business Rules

**Danger level from YOLO confidence** (`backend/utils/dangerLevel.js`)
| Confidence | Danger level |
|------------|--------------|
| missing / falsy | `MEDIUM` |
| `>= 0.7` | `HIGH` |
| `>= 0.4` and `< 0.7` | `MEDIUM` |
| `< 0.4` | `LOW` |

On `POST /api/alerts`, explicit `dangerLevel` wins; otherwise it is derived from `confidence`;
otherwise it defaults to `MEDIUM`.

**Find-or-create crosswalk** (`alertControllers.findOrCreateCrosswalk`)
- If `crosswalkId` given → use it.
- Else if `location` given → find a crosswalk matching city+street+number, or create one.
- Errors here are swallowed (logged) so the alert is still created without a crosswalk.

**Referential safety on delete** — a Camera or LED that is linked to a crosswalk cannot be
deleted (returns HTTP 400 with a message).

**Pagination** (`controllerHelpers.parsePagination` / `buildPagination`)
- `page = parseInt(query.page) || 1`, `limit = parseInt(query.limit) || defaultLimit`
- `skip = (page - 1) * limit`
- Response includes `{ currentPage, totalPages, total, hasMore }`.

---

## 5. Error Handling

Global handler `backend/middleware/errorMiddleware.js`:
```js
const statusCode = res.statusCode === 200 ? 500 : res.statusCode;
```
The status code is whatever a controller/middleware set before throwing; if nothing set a
non-200 code, the response is **500**. This means Mongoose `ValidationError` / `CastError` that
bubble up without an explicit status become **500** (see BUG-002).

---

## 6. Real-time (Socket.IO) — `backend/socket/index.js`
- Namespace `/traffic`; rooms: `dashboard`, `crosswalk:<id>`, `crosswalk:<id>:leds`, `led:<id>`.
- `emitAlertRealtime(alert)` broadcasts `alert:new` globally and to the dashboard + crosswalk rooms.
- Frontend (`SocketBridge.jsx`): HIGH → error toast, MEDIUM → warning toast, **LOW → silent**;
  toast format `ALARM {LEVEL} | {location} | {timestamp}`, duration 7000 ms.

---

## 7. Frontend Pages (operator UI)
| Route | Page | Highlights |
|-------|------|------------|
| `/` | Dashboard | Total Alerts / Total Crosswalks stats; **System Status card (API/DB/YOLO)**; Quick Actions (Add Camera/LED, Manage Devices) |
| `/alerts` | Alerts | 4 stat cards; FilterBar (Danger Level, Crosswalk search, Date Range); Load More; Add/Edit/Delete alert |
| `/crosswalks` | Crosswalks | 4 stat cards; SearchBar; CrosswalkCard list; Add/Edit/Delete; link/unlink devices |
| `/crosswalks/:id` | Crosswalk Details | Detail card; Events History with filters + Load More; per-event AlertHistoryCard |

> Note: the Dashboard **System Status** values are hard-coded in `Dashboard.jsx:130-134`
> (`status="online"` / `status="connected"`), not bound to any live health check — see BUG-006.

---

## 8. Seeded Test Data (`backend/scripts/seedDatabase.js`)
- **3 Cameras**: 2 `active`, 1 `inactive`.
- **3 LEDs**.
- **3 Crosswalks**:
  1. תל אביב / דיזנגוף / 50 → camera1 + led1
  2. תל אביב / אבן גבירול / 123 → camera2 + led2
  3. ירושלים / יפו / 234 → camera3 + led3
- **70 Alerts**: ~40% LOW, ~40% MEDIUM, ~20% HIGH; ~10% unlinked; timestamps within the last 30 days.

`npm run seed` populates this; `cleanDatabase.js` clears all four collections.
