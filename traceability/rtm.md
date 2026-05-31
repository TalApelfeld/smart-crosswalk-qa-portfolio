# Requirements Traceability Matrix (RTM)

The RTM is the backbone of this portfolio: it proves **every requirement is covered by at least
one test case**, and links each discovered **defect** back to the test case and requirement that
exposed it. You can walk in either direction — requirement → tests → defects, or defect → test →
requirement.

## REQ → Test Cases → Defects

| Requirement | Test Case(s) | Defect(s) |
|-------------|--------------|-----------|
| REQ-001 — Alerts paginated & sorted | TC-001, TC-009, TC-015, TC-025 | — |
| REQ-002 — Alert filtering | TC-013, TC-014, TC-026 | — |
| REQ-003 — Danger level from confidence | TC-005, TC-007 | — |
| REQ-004 — Explicit danger level overrides | TC-008 | — |
| REQ-005 — Find-or-create crosswalk from location | TC-018 (also exercised via POST alert w/ location) | — |
| REQ-006 — dangerLevel enum constraint | TC-005, TC-006, TC-016 | BUG-002 |
| REQ-007 — ObjectId validation on `:id` | TC-003 | — |
| REQ-008 — 404 for non-existent IDs | TC-002, TC-004, TC-024, TC-044 | — |
| REQ-009 — Crosswalk required location fields | TC-018, TC-019 | — |
| REQ-010 — Search query rules & safety | TC-020, TC-021, TC-022, TC-023 | BUG-001 |
| REQ-011 — Camera status enum on all writes | TC-033, TC-034, TC-035, TC-036 | BUG-003 |
| REQ-012 — No delete of linked device | TC-037, TC-038, TC-042 | — |
| REQ-013 — Link/unlink devices | TC-024, TC-028, TC-029, TC-030, TC-031, TC-032, TC-039 | — |
| REQ-014 — Cloudinary asset cleanup on alert delete | TC-017 | — |
| REQ-015 — LED command validation & ACK | TC-040, TC-041 | — |
| REQ-016 — Realtime broadcast on new alert | TC-047 | — |
| REQ-017 — Toast severity behavior | TC-047, TC-048 | — |
| REQ-018 — Alerts page filters/empty/pagination | TC-045, TC-046 | — |
| REQ-019 — Dashboard real system status | TC-049 | BUG-006 |
| REQ-020 — Health reports DB state | TC-043 | — |
| REQ-021 — Correct 4xx vs 5xx status codes | TC-006, TC-023, TC-027, TC-034, TC-050 | BUG-001, BUG-002, BUG-003, BUG-005, BUG-007 |
| REQ-022 — Graceful pagination edges | TC-009, TC-010, TC-011, TC-012 | BUG-004 |

**Coverage:** 22 / 22 requirements have ≥ 1 test case → **100% requirement coverage**.

## Defect → Test Case → Requirement (reverse trace)

| Defect | Exposed by | Violates |
|--------|------------|----------|
| BUG-001 — Search regex 500 | TC-023 | REQ-010, REQ-021 |
| BUG-002 — Validation → 500 | TC-006 | REQ-006, REQ-021 |
| BUG-003 — Camera create no validation | TC-034 | REQ-011, REQ-021 |
| BUG-004 — Negative page → 500 | TC-011 | REQ-022 |
| BUG-005 — Invalid date → 500 | TC-027 | REQ-021 |
| BUG-006 — Hard-coded dashboard status | TC-049 | REQ-019 |
| BUG-007 — README contract mismatch | TC-050 | REQ-021 |
