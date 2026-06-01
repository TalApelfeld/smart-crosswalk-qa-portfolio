# AI Smart Crosswalk System - QA

**Manual QA engagement on a real, full-stack project.** This repository documents the
requirements, test plan, manual test cases, traceability, and defects I produced while testing
the **AI Smart Crosswalk System** — an AI-powered pedestrian-safety platform (computer-vision
detection → danger grading → alerts → LED signal control).

## The 7 defects at a glance
| ID | Sev | Summary |
|----|-----|---------|
| [BUG-001](bug-reports/BUG-001-crosswalk-search-regex-500.md) | High | Crosswalk search builds a `RegExp` from raw input → `(` crashes it (500) + ReDoS risk |
| [BUG-002](bug-reports/BUG-002-validation-errors-return-500.md) | Medium | Mongoose validation errors return **500** instead of **400** |
| [BUG-003](bug-reports/BUG-003-camera-create-skips-status-validation.md) | Medium | `POST /api/cameras` skips status validation (only the update path validates) |
| [BUG-004](bug-reports/BUG-004-negative-page-param-500.md) | Medium | Negative `?page=-1` → negative skip → 500 |
| [BUG-005](bug-reports/BUG-005-invalid-date-filter-500.md) | Medium | Invalid date in alert filter → cast error → 500 |
| [BUG-006](bug-reports/BUG-006-dashboard-hardcoded-system-status.md) | Medium | Dashboard "System Status" is hard-coded to healthy |
| [BUG-007](bug-reports/BUG-007-readme-endpoint-contract-mismatch.md) | Low | README API contract doesn't match the implemented routes |

---

## Repository structure
```
qa-portfolio/
├── docs/
│   ├── sut-overview.md        # System-under-test map: architecture, models, endpoints, rules
│   ├── test-plan.md           # IEEE 829 test plan (scope, strategy, criteria, risks)
│   └── test-strategy.md       # Test-design techniques, applied to real features
├── requirements/              # 22 testable requirements (md + csv)
├── test-cases/                # 50 manual test cases (md + csv)
├── traceability/              # RTM: REQ → TC → BUG (md + csv)
├── bug-reports/               # 7 defect reports (md) + register (csv)
├── reports/
│   └── test-summary-report.md # IEEE 829 summary + execution plan
├── reproducibility/
│   ├── smart-crosswalk.postman_collection.json
│   ├── curl-commands.md
│   └── test-data.md
├── evidence/                  # execution artifacts (added when the suite is run)
└── templates/                 # reusable test-case + bug-report templates
```

## How to read it
1. Start with [`docs/sut-overview.md`](docs/sut-overview.md) to understand the system.
2. Read the [`docs/test-plan.md`](docs/test-plan.md) and [`docs/test-strategy.md`](docs/test-strategy.md).
3. Browse [`requirements/`](requirements/requirements.md) → [`test-cases/`](test-cases/test-cases.md) → [`traceability/rtm.md`](traceability/rtm.md).
4. Review the [`bug-reports/`](bug-reports/) and the [`reports/test-summary-report.md`](reports/test-summary-report.md).

## How to reproduce
Start the SUT (backend + a MongoDB instance; UI optional), capture the seeded IDs, then run the
Postman collection or the curl commands. Full steps are in
[`reports/test-summary-report.md` §6](reports/test-summary-report.md) and
[`reproducibility/test-data.md`](reproducibility/test-data.md).

---

## About the System Under Test
The AI Smart Crosswalk System is a real project: **Node.js/Express + Mongoose + MongoDB** backend,
**React + Vite + TanStack Query** frontend, **Socket.IO** real-time layer, and a **Python/YOLOv8**
pose-estimation module that detects pedestrians and posts danger-graded alerts. This QA portfolio
treats it as the system under test; it does not modify the application code.

— *Tal Apelfeld*
