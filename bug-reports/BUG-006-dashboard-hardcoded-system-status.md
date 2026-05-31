# BUG-006 — Dashboard System Status is hard-coded to healthy and never reflects real state

| Field | Value |
|-------|-------|
| **Bug ID** | BUG-006 |
| **Severity** | Medium |
| **Priority** | Medium |
| **Status** | Open |
| **Found By** | Code analysis (needs runtime confirmation) |
| **Component** | `frontend/src/pages/Dashboard.jsx` (lines 130–134) |
| **Linked Requirement** | REQ-019 |
| **Linked Test Case** | TC-049 |
| **Environment** | Frontend (React 18 / Vite), any browser |

## Summary
The Dashboard's "System Status" card shows **API Server: Online**, **Database: Connected**, and
**YOLO Detection: Connected**, but these values are **hard-coded static props** — they are not
bound to any health check or live signal. The card therefore reports the system as fully healthy
even when the backend is down, the database is disconnected, or the AI module is not running.
This is misleading for an operator-facing monitoring dashboard.

## Preconditions
Frontend running.

## Steps to Reproduce
1. Stop the backend server (or disconnect MongoDB / stop the YOLO module).
2. Load the Dashboard at `/`.
3. Inspect the "System Status" card.

## Expected Result
Each row reflects the **actual** state: e.g. "API Server: Offline" when the backend is
unreachable, "Database: Disconnected" when `GET /api/health` reports `database.connected:false`,
and a real indicator for the AI module.

## Actual Result (predicted from code)
All three rows always render as Online/Connected, regardless of true state.

## Root Cause
```jsx
fields={[
  { label: "API Server",     component: <StatusIndicator status="online"    label="Online" /> },
  { label: "Database",       component: <StatusIndicator status="connected" label="Connected" /> },
  { label: "YOLO Detection", component: <StatusIndicator status="connected" label="Connected" /> },
]}
```
The status strings are literals; there is no call to `/api/health` and no socket/health state
driving them.

## Suggested Fix
Drive the indicators from real signals:
- **Database / API:** poll `GET /api/health` (already returns `database.connected` and overall
  `success`) and map the result to the indicator. Show "Offline" if the request fails.
- **YOLO Detection:** derive from a real signal (e.g. a heartbeat/last-detection timestamp, or a
  Socket.IO connection event from the AI module) rather than a constant.

## Note
This finding is included to demonstrate UI/observability testing. It is lower-risk than the API
defects but directly contradicts the dashboard's purpose (real-time monitoring).
