# Evidence

This folder holds execution artifacts (request/response captures, screenshots, log excerpts)
produced when the test suite is **run** against a live SUT.

This baseline is a **test-design + code-analysis** engagement (see
`../reports/test-summary-report.md`), so no execution artifacts are committed yet. When the suite
is executed, save one file per executed test/defect here, named by ID, e.g.:

```
TC-006-invalid-enum-response.txt
BUG-001-search-500-stacktrace.txt
BUG-006-dashboard-status-screenshot.png
```

Each capture should include the request, the full response (status line + body), and the date/
environment so the result is reproducible and auditable.
