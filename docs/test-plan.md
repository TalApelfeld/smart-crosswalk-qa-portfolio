# Test Plan — AI Smart Crosswalk System

*Structured after IEEE 829. Author: Tal Apelfeld. Status: Baseline v1.0.*

---

## 1. Introduction
The AI Smart Crosswalk System detects pedestrians via computer vision, raises danger-graded
alerts, and controls LED crosswalk signals. This plan defines **what** will be tested, **how**,
and the **criteria** for considering testing complete. It targets the backend REST API, the
operator web UI, the real-time layer, and the documented business rules.

## 2. Test Items
- Backend REST API (`smart-crosswalk/backend`) — Alerts, Crosswalks, Cameras, LEDs, Health.
- Validation middleware and the global error handler.
- Business rules: danger-level derivation, find-or-create crosswalk, referential delete safety.
- Real-time alert broadcasting (Socket.IO).
- Frontend operator UI (Dashboard, Alerts, Crosswalks, Crosswalk Details).

## 3. Features To Be Tested
| Area | Examples |
|------|----------|
| Alerts CRUD + stats | create/list/get/update/delete, pagination, filtering, stats |
| Crosswalks CRUD + search + device linking | required fields, search query rules, link/unlink camera & LED |
| Cameras | create, status enum, status update, delete with referential check |
| LEDs | create, command (ON/OFF/BLINK), delete with referential check |
| Input validation | ObjectId format, required fields, enums, query params |
| Business rules | confidence→danger mapping, find-or-create crosswalk |
| Error handling | correct HTTP status codes for client vs server errors |
| Real-time | new-alert broadcast, toast severity behavior |
| UI | filters, search, empty states, pagination, dialogs, system status card |

## 4. Features Not To Be Tested (Out of Scope)
- The YOLOv8 model's detection accuracy / ML quality (separate concern from software QA).
- Cloudinary's own upload/delete service reliability (third-party).
- MongoDB Atlas infrastructure and Socket.IO library internals.
- Load/performance and penetration testing beyond the noted robustness checks.
- Authentication/authorization — the API currently ships without auth (noted as a risk in §10).

## 5. Approach (Test Strategy summary)
Black-box, requirement-driven manual testing, complemented by white-box code analysis (source
is available). Design techniques: Equivalence Partitioning, Boundary Value Analysis, Decision
Tables, State Transition, and Error Guessing. API tests are reproducible via a **Postman
collection** and **curl** commands. See `docs/test-strategy.md` for technique-by-technique detail.

## 6. Item Pass/Fail Criteria
- A **test case passes** when the actual result matches the expected result exactly (status code,
  response shape, and/or UI state/strings).
- A **test case fails** when behavior deviates; a linked `BUG-NNN` is raised.
- A requirement is **covered** when at least one test case traces to it in the RTM.

## 7. Test Deliverables
- `requirements/` — requirements (md + csv)
- `test-cases/` — test cases (md + csv)
- `traceability/` — RTM (md + csv)
- `bug-reports/` — defect reports (md) + register (csv)
- `reports/test-summary-report.md` — summary
- `reproducibility/` — Postman collection, curl commands, known test data

## 8. Environment Needs
- **Backend**: Node.js 18+, the project's `.env` with a MongoDB connection string, `npm install`, `npm run dev`.
- **Database**: a MongoDB instance (Atlas, local `mongod`, or `mongodb-memory-server`). Seed with `npm run seed`.
- **Frontend**: `npm install && npm run dev` (Vite at `http://localhost:5173`).
- **Tools**: Postman (or Newman), curl, a modern browser, browser dev tools.

## 9. Entry & Exit Criteria
**Entry:** SUT builds and starts; health check returns connected; seed data loaded; test data documented.
**Exit:** all planned test cases executed; 100% of requirements traced to ≥1 test case; all
Critical/High defects triaged; test summary report published.

## 10. Risks & Assumptions
| Risk | Mitigation |
|------|------------|
| No authentication on the API — any client can read/delete all data | Documented as a security risk; recommend auth before production |
| Unsanitized user input flows into a `RegExp` (search) | Covered by TC + BUG-001 |
| Inconsistent HTTP status codes (validation → 500) | Covered by TC + BUG-002 |
| Real DB testing could pollute production data | Use a disposable/seeded DB, never prod |

**Assumption:** This baseline is a **test-design + code-analysis** engagement. Test cases are
authored and ready to execute; defects listed were found by source-code review and are labeled
*Needs runtime confirmation*. No execution metrics are reported until the suite is run (see the
test summary report).

## 11. Schedule (logical phases)
1. Understand SUT → 2. Author requirements → 3. Design test cases → 4. Build RTM →
5. Code-analysis defect discovery → 6. (Execution phase — pending) → 7. Summary report.
