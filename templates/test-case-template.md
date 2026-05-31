# Test Case Template

> Copy this block for every new test case. One test case = one verifiable behavior.

| Field | Value |
|-------|-------|
| **Test Case ID** | TC-NNN |
| **Title** | Short, action-oriented title |
| **Linked Requirement(s)** | REQ-NNN |
| **Module / Area** | Alerts / Crosswalks / Cameras / LEDs / UI / Non-functional |
| **Type** | Positive / Negative / Boundary / Functional / UI / Non-functional |
| **Design Technique** | Equivalence Partitioning / Boundary Value Analysis / Decision Table / State Transition / Error Guessing |
| **Priority** | High / Medium / Low |
| **Preconditions** | What must be true before running (server up, seeded data, a known ID, etc.) |

**Test Data**

```
(any request body, query params, or known IDs used)
```

**Steps**

1. Step one
2. Step two
3. ...

**Expected Result**

- HTTP status code and/or UI state
- Response body shape / exact UI strings

**Actual Result**

- _(filled during execution)_

**Status:** Not Executed / Pass / Fail / Blocked

**Linked Defect(s):** BUG-NNN (if the test fails)

**Notes:** _(optional)_
