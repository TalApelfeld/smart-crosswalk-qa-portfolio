# Test Summary Report — AI Smart Crosswalk System

*Structured after IEEE 829. Author: Tal Apelfeld. Baseline v1.0.*

## 1. Summary
This report covers a **test-design and code-analysis** engagement against the AI Smart Crosswalk
system (a real full-stack project: Express/MongoDB API, React UI, Socket.IO, Python YOLOv8). The
engagement produced a full set of QA artifacts — requirements, a test plan, a 50-case manual test
suite, a requirements traceability matrix, and 7 defect reports — and verified the SUT's behavior
by reading its source code.

> **Honesty note.** No test execution metrics (pass/fail percentages) are reported in this
> baseline. The test cases are **authored and ready to run** but are marked *Not Executed*. The 7
> defects were found by **source-code analysis** and are labeled *Open / needs runtime
> confirmation*; each cites the exact file and code that produces the behavior. When the suite is
> executed (see §6), results and evidence will be recorded in `evidence/` and this report updated.

## 2. Coverage Metrics (design-time)
| Metric | Value |
|--------|-------|
| Requirements specified | 22 |
| Requirements with ≥1 test case | 22 (**100%** traceability) |
| Test cases designed | 50 |
| — API / functional (Alerts, Crosswalks, Cameras, LEDs) | 42 |
| — UI / real-time | 5 |
| — Non-functional / documentation (health, routing, contract) | 3 |
| Design techniques applied | EP, BVA, Decision Table, State Transition, Error Guessing |
| Defects identified (code analysis) | 7 |

## 3. Defects Found
| ID | Severity | Area | Summary |
|----|----------|------|---------|
| BUG-001 | High | Crosswalk search | Unsanitized `RegExp(q)` → 500 + ReDoS risk |
| BUG-002 | Medium | Error handling | Mongoose validation errors return 500 instead of 400 |
| BUG-003 | Medium | Cameras | `POST /api/cameras` skips status validation → 500 |
| BUG-004 | Medium | Pagination | Negative `page` → negative skip → 500 |
| BUG-005 | Medium | Crosswalk alerts | Invalid date filter → cast error → 500 |
| BUG-006 | Medium | Dashboard UI | System Status hard-coded to healthy |
| BUG-007 | Low | Documentation | README API contract mismatch |

**Theme:** the most impactful cluster (BUG-001/002/003/004/005) is **inconsistent input
validation and error handling** — client-side mistakes surface as HTTP 500 server errors instead
of 4xx, and one path (search) can be crashed by ordinary input. A single improvement to the global
error handler plus input clamping/escaping would resolve five of seven findings.

## 4. Risk Assessment
- **High:** `BUG-001` (availability — a trivial request crashes the search endpoint; DoS surface).
- **Medium:** the 500-vs-400 family — degrades API usability/observability and can mask real
  failures.
- **Cross-cutting (noted in the test plan):** the API ships **without authentication**; any client
  can read or delete all data. Recommended for triage before production, though out of scope here.

## 5. Evaluation vs Exit Criteria
| Exit criterion | Status |
|----------------|--------|
| All requirements traced to ≥1 test case | ✅ Met (100%) |
| Test suite authored & reproducible (Postman + curl) | ✅ Met |
| Defects documented with root cause + fix | ✅ Met (7/7) |
| All planned test cases **executed** | ⏳ Pending (execution phase) |
| Critical/High defects triaged | ⏳ Pending fix verification |

## 6. How to Execute (reproduce)
1. Start a database: MongoDB Atlas, a local `mongod`, or `mongodb-memory-server` (recommended so
   no real data is touched).
2. Configure `smart-crosswalk/backend/.env` with `MONGODB_URI`, `PORT=3000`, `NODE_ENV=development`,
   `CORS_ORIGIN=http://localhost:5173`.
3. `cd smart-crosswalk/backend && npm install && npm run seed && npm run dev`.
4. (UI cases) `cd smart-crosswalk/frontend && npm install && npm run dev`.
5. Capture real IDs (see `reproducibility/test-data.md`), then either import
   `reproducibility/smart-crosswalk.postman_collection.json` into Postman or run the matching
   commands in `reproducibility/curl-commands.md`.
6. Record each result in `test-cases/test-cases.csv` (Status column) and save artifacts to
   `evidence/`; update §2–§5 of this report with actual pass/fail counts.

## 7. Recommendations
1. Fix the global error handler to map `ValidationError`/`CastError` → 400 (resolves BUG-002 and
   improves BUG-003/005). 
2. Escape user input before building the search `RegExp`, or switch to a text index (BUG-001).
3. Clamp `page`/`limit` to positive bounds (BUG-004).
4. Validate dates before querying (BUG-005).
5. Bind the Dashboard status card to `/api/health` + a real AI heartbeat (BUG-006).
6. Reconcile the README endpoint table with the routes (BUG-007).
7. Add authentication/authorization before any production deployment.
