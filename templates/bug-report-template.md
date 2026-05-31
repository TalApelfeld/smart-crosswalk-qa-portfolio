# Bug Report Template

> Copy this block for every defect. A good bug report is reproducible by someone who has never seen the system.

| Field | Value |
|-------|-------|
| **Bug ID** | BUG-NNN |
| **Title** | Concise summary: [where] [what happens] [under what condition] |
| **Severity** | Critical / High / Medium / Low — _impact on the system_ |
| **Priority** | High / Medium / Low — _how soon it should be fixed_ |
| **Status** | Open / In Progress / Fixed / Closed / Won't Fix |
| **Found By** | Manual testing / Code analysis / Exploratory |
| **Component** | File / endpoint / page where the defect lives |
| **Linked Requirement(s)** | REQ-NNN |
| **Linked Test Case(s)** | TC-NNN |
| **Environment** | Backend commit / Node version / DB / OS |

## Preconditions
What state the system must be in before reproducing.

## Steps to Reproduce
1. ...
2. ...
3. ...

## Expected Result
What a correct system should do.

## Actual Result
What the system actually does (include status codes, error text, screenshots/log lines).

## Evidence
Request/response, stack trace, or screenshot reference.

## Root Cause (analysis)
The specific code path and file:line that causes the behavior.

## Suggested Fix
A concrete, minimal change that resolves it.
