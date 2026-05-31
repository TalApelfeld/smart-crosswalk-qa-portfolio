# Test Strategy & Design Techniques

This document shows *how* the test cases were designed — demonstrating software-testing
fundamentals and manual-testing methodology. Each technique below is applied to a real part of
the AI Smart Crosswalk system, with example test cases referenced by ID.

---

## 1. Testing Levels & Types Used
| Level / Type | Applied to |
|--------------|------------|
| **Functional / black-box** | API behavior against documented requirements |
| **Negative testing** | Invalid IDs, missing required fields, bad enums, malformed query params |
| **Boundary testing** | Pagination edges, search-query length, confidence thresholds |
| **White-box (code analysis)** | Reading controllers/middleware to find error-handling defects |
| **UI / exploratory** | Dashboard, Alerts, Crosswalks pages — labels, filters, empty states |
| **Integration** | Find-or-create crosswalk from an alert; device link/unlink; realtime broadcast |

---

## 2. Equivalence Partitioning (EP)
Divide inputs into classes that should be treated the same; test one representative per class.

- **Camera `status`** → valid class `{active, inactive, error}`, invalid class `{anything else}`
  → TC-019 (valid), TC-020 (invalid).
- **`dangerLevel`** → valid `{LOW, MEDIUM, HIGH}`, invalid `{everything else}`
  → TC-005 (valid), TC-006 (invalid enum).
- **`:id` route param** → valid 24-hex ObjectId vs malformed string → TC-002, TC-003.

## 3. Boundary Value Analysis (BVA)
Test the edges where behavior changes.

- **Search query length** (`q` must be ≥ 2): length 0 → 400, length 1 → 400, length 2 → 200.
  → TC-014, TC-015, TC-016.
- **Pagination**: `page=1` (first), `page` beyond last (empty page), `page=-1` (negative edge),
  `limit=0` (falls back to default). → TC-009, TC-010, TC-011, TC-012.
- **Confidence thresholds** (0.4 and 0.7 boundaries): 0.39, 0.40, 0.69, 0.70, missing.
  → decision table in §4, TC-007.

## 4. Decision Table — danger level from confidence
Rule source: `backend/utils/dangerLevel.js`.

| # | Condition: `confidence` | Expected `dangerLevel` |
|---|--------------------------|------------------------|
| R1 | missing / 0 / falsy | MEDIUM |
| R2 | 0.39 | LOW |
| R3 | 0.40 | MEDIUM |
| R4 | 0.69 | MEDIUM |
| R5 | 0.70 | HIGH |
| R6 | 0.95 | HIGH |

Note interaction: an explicit `dangerLevel` in the body overrides the derived value (R-override),
covered by TC-008.

## 5. State Transition Testing
- **Camera status**: `active → inactive → error → active` via `PATCH /:id/status`. → TC-021.
- **Crosswalk ↔ device links**: `unlinked → linked (camera) → unlinked`; same for LED.
  → TC-026, TC-027, TC-028, TC-029.
- **Referential safety**: a linked camera/LED cannot transition to deleted. → TC-023, TC-033.

## 6. Error Guessing & Robustness
Experience-driven probing of likely weak spots (these surfaced real defects):
- Feeding regex metacharacters into the crosswalk search (`q=(`). → TC-017 / BUG-001.
- Sending an invalid enum to a create endpoint with no validation. → TC-020 / BUG-003.
- Sending a malformed date to a date-range filter. → TC-031 / BUG-005.
- Negative pagination index. → TC-011 / BUG-004.

## 7. UI / Exploratory Charter
For each page, verify: correct labels & headings, filter behavior, empty-state copy, "Load More"
pagination, dialog open/validate/submit/cancel, toast messages, and the realtime alert behavior
(HIGH/MEDIUM toast vs LOW silent). Exact strings are asserted from the source (see test cases
TC-034 – TC-040 and `docs/sut-overview.md §7`).

## 8. Prioritization (risk-based)
- **High**: data-integrity and availability paths — create/delete, validation, the 500-error
  defects, referential safety.
- **Medium**: filtering, pagination, search, stats accuracy.
- **Low**: cosmetic UI strings, documentation contract mismatches.

## 9. Reproducibility
Every API test case is runnable two ways: import `reproducibility/smart-crosswalk.postman_collection.json`
into Postman, or paste the matching command from `reproducibility/curl-commands.md`. Known IDs and
seeded values are listed in `reproducibility/test-data.md`.
