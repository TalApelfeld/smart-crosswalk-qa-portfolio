# curl Commands — Reproducible API Tests

Copy-paste commands for every API test case and defect repro. They use `-i` so you can see the
**HTTP status line** (the key assertion for most cases). Replace `{...}` placeholders using
`reproducibility/test-data.md`.

> Base URL assumed `http://localhost:3000`. On Windows PowerShell, prefer `curl.exe` (not the
> `curl` alias) or use the Postman collection instead.

## Setup
```bash
# health check first
curl -i http://localhost:3000/api/health
```

## Alerts
```bash
# TC-001 list (default pagination)
curl -i "http://localhost:3000/api/alerts"

# TC-002 get by valid id
curl -i "http://localhost:3000/api/alerts/{validAlertId}"

# TC-003 malformed id -> expect 400
curl -i "http://localhost:3000/api/alerts/123"

# TC-004 valid but missing id -> expect 404
curl -i "http://localhost:3000/api/alerts/64b0000000000000000000ff"

# TC-005 create with explicit danger level -> expect 201
curl -i -X POST http://localhost:3000/api/alerts \
  -H "Content-Type: application/json" \
  -d '{"dangerLevel":"HIGH","imageUrl":"https://example.com/p.jpg"}'

# TC-006 invalid enum -> EXPECT 400 (BUG-002: actually 500)
curl -i -X POST http://localhost:3000/api/alerts \
  -H "Content-Type: application/json" -d '{"dangerLevel":"EXTREME"}'

# TC-007 derive from confidence (run per row)
curl -i -X POST http://localhost:3000/api/alerts -H "Content-Type: application/json" -d '{"confidence":0.39}'  # LOW
curl -i -X POST http://localhost:3000/api/alerts -H "Content-Type: application/json" -d '{"confidence":0.40}'  # MEDIUM
curl -i -X POST http://localhost:3000/api/alerts -H "Content-Type: application/json" -d '{"confidence":0.70}'  # HIGH
curl -i -X POST http://localhost:3000/api/alerts -H "Content-Type: application/json" -d '{}'                   # MEDIUM (default)

# TC-008 explicit overrides confidence -> LOW
curl -i -X POST http://localhost:3000/api/alerts -H "Content-Type: application/json" -d '{"dangerLevel":"LOW","confidence":0.95}'

# TC-009..TC-012 pagination
curl -i "http://localhost:3000/api/alerts?page=2&limit=5"     # TC-009
curl -i "http://localhost:3000/api/alerts?page=999&limit=10"  # TC-010 empty
curl -i "http://localhost:3000/api/alerts?page=-1&limit=10"   # TC-011 EXPECT graceful (BUG-004: 500)
curl -i "http://localhost:3000/api/alerts?limit=0"            # TC-012 default 10

# TC-013/14 filters
curl -i "http://localhost:3000/api/alerts?dangerLevel=HIGH"
curl -i "http://localhost:3000/api/alerts?crosswalkId={validCrosswalkId}"

# TC-015 stats
curl -i "http://localhost:3000/api/alerts/stats"

# TC-016 update
curl -i -X PATCH "http://localhost:3000/api/alerts/{validAlertId}" -H "Content-Type: application/json" -d '{"dangerLevel":"LOW"}'

# TC-017 delete
curl -i -X DELETE "http://localhost:3000/api/alerts/{validAlertId}"
```

## Crosswalks
```bash
# TC-018 create valid
curl -i -X POST http://localhost:3000/api/crosswalks -H "Content-Type: application/json" \
  -d '{"location":{"city":"Haifa","street":"Herzl","number":"10"}}'

# TC-019 missing fields -> 400
curl -i -X POST http://localhost:3000/api/crosswalks -H "Content-Type: application/json" \
  -d '{"location":{"city":"Haifa"}}'

# TC-020 search empty -> 400
curl -i "http://localhost:3000/api/crosswalks/search"

# TC-021 search 1 char -> 400
curl -i "http://localhost:3000/api/crosswalks/search?q=a"

# TC-022 search 2 chars -> 200
curl -i "http://localhost:3000/api/crosswalks/search?q=He"

# TC-023 regex metachar -> EXPECT 200/400, NOT 500 (BUG-001)
curl -i "http://localhost:3000/api/crosswalks/search?q=%28%28"

# TC-024 get by id
curl -i "http://localhost:3000/api/crosswalks/{validCrosswalkId}"

# TC-025 crosswalk alert stats
curl -i "http://localhost:3000/api/crosswalks/{validCrosswalkId}/stats"

# TC-026 crosswalk alerts filtered
curl -i "http://localhost:3000/api/crosswalks/{validCrosswalkId}/alerts?dangerLevel=high"

# TC-027 invalid date -> EXPECT graceful, NOT 500 (BUG-005)
curl -i "http://localhost:3000/api/crosswalks/{validCrosswalkId}/alerts?startDate=not-a-date"

# TC-028 link camera
curl -i -X PATCH "http://localhost:3000/api/crosswalks/{validCrosswalkId}/camera" -H "Content-Type: application/json" -d '{"cameraId":"{validCameraId}"}'

# TC-029 link missing camera -> 404
curl -i -X PATCH "http://localhost:3000/api/crosswalks/{validCrosswalkId}/camera" -H "Content-Type: application/json" -d '{"cameraId":"64b0000000000000000000ff"}'

# TC-030 link without cameraId -> 400
curl -i -X PATCH "http://localhost:3000/api/crosswalks/{validCrosswalkId}/camera" -H "Content-Type: application/json" -d '{}'

# TC-031 unlink camera
curl -i -X DELETE "http://localhost:3000/api/crosswalks/{validCrosswalkId}/camera"

# TC-032 link + unlink LED
curl -i -X PATCH "http://localhost:3000/api/crosswalks/{validCrosswalkId}/led" -H "Content-Type: application/json" -d '{"ledId":"{validLedId}"}'
curl -i -X DELETE "http://localhost:3000/api/crosswalks/{validCrosswalkId}/led"
```

## Cameras
```bash
# TC-033 create valid
curl -i -X POST http://localhost:3000/api/cameras -H "Content-Type: application/json" -d '{"status":"active"}'

# TC-034 invalid status -> EXPECT 400 (BUG-003: 500)
curl -i -X POST http://localhost:3000/api/cameras -H "Content-Type: application/json" -d '{"status":"broken"}'

# TC-035 status transitions
curl -i -X PATCH "http://localhost:3000/api/cameras/{validCameraId}/status" -H "Content-Type: application/json" -d '{"status":"inactive"}'
curl -i -X PATCH "http://localhost:3000/api/cameras/{validCameraId}/status" -H "Content-Type: application/json" -d '{"status":"error"}'

# TC-036 invalid status update -> 400
curl -i -X PATCH "http://localhost:3000/api/cameras/{validCameraId}/status" -H "Content-Type: application/json" -d '{"status":"on"}'

# TC-037 delete linked camera -> 400
curl -i -X DELETE "http://localhost:3000/api/cameras/{linkedCameraId}"

# TC-038 delete unlinked camera -> 200
curl -i -X DELETE "http://localhost:3000/api/cameras/{unlinkedCameraId}"
```

## LEDs
```bash
# TC-039 create
curl -i -X POST http://localhost:3000/api/leds -H "Content-Type: application/json" -d '{}'

# TC-040 invalid state -> 400
curl -i -X POST "http://localhost:3000/api/leds/{validLedId}/command" -H "Content-Type: application/json" -d '{"state":"PURPLE"}'

# TC-041 valid state, no subscriber -> 400/504 (not 2xx)
curl -i -X POST "http://localhost:3000/api/leds/{validLedId}/command" -H "Content-Type: application/json" -d '{"state":"ON"}'

# TC-042 delete linked LED -> 400
curl -i -X DELETE "http://localhost:3000/api/leds/{linkedLedId}"
```

## System
```bash
# TC-043 health
curl -i "http://localhost:3000/api/health"

# TC-044 unknown route -> 404
curl -i "http://localhost:3000/api/does-not-exist"
```
