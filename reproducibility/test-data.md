# Known Test Data

The backend ships a seed script (`backend/scripts/seedDatabase.js`) that creates a deterministic
data set. Run it before executing the test cases so the placeholders in this portfolio resolve to
real records.

## How to load
```bash
cd smart-crosswalk/backend
npm install
npm run seed       # populates cameras, leds, crosswalks, alerts
# to reset:
node scripts/cleanDatabase.js
```

## What gets created
| Entity | Count | Details |
|--------|-------|---------|
| Cameras | 3 | 2 `active`, 1 `inactive` |
| LEDs | 3 | no extra fields |
| Crosswalks | 3 | see below |
| Alerts | 70 | ~40% LOW, ~40% MEDIUM, ~20% HIGH; ~10% unlinked; timestamps within last 30 days |

### Crosswalks (seeded)
| # | City | Street | Number | Camera | LED |
|---|------|--------|--------|--------|-----|
| 1 | תל אביב (Tel Aviv) | דיזנגוף (Dizengoff) | 50 | camera1 (active) | led1 |
| 2 | תל אביב (Tel Aviv) | אבן גבירול (Eben Gabirol) | 123 | camera2 | led2 |
| 3 | ירושלים (Jerusalem) | יפו (Jaffo) | 234 | camera3 | led3 |

## Placeholder → how to obtain the real value
Because Mongo ObjectIds are generated at seed time, capture them at runtime:

| Placeholder | How to get it |
|-------------|---------------|
| `{validCrosswalkId}` | `GET /api/crosswalks` → take `data[0]._id` |
| `{validAlertId}` | `GET /api/alerts` → take `data[0]._id` |
| `{validCameraId}` | `GET /api/cameras` → take `data[0]._id` |
| `{validLedId}` | `GET /api/leds` → take `data[0]._id` |
| `{linkedCameraId}` | A camera that appears as a crosswalk's `cameraId` (seed camera1) |
| `{linkedLedId}` | An LED that appears as a crosswalk's `ledId` (seed led1) |
| `{missingId}` | Use a syntactically valid but unused ObjectId: `64b0000000000000000000ff` |
| `123` (malformed) | Any non-ObjectId string to test `validateObjectId` |

> Note on search test (TC-022): the seeded streets are in Hebrew. For an ASCII search match, first
> create a crosswalk with a Latin street (e.g. TC-018 creates "Herzl, Haifa") and search `q=He`.
