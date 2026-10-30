# Technical Design — Smart Trip Planner MVP

**Status:** Proposal, subject to implementation review.

## 1. Stack
- Python 3.13+, FastAPI, Pydantic v2, httpx, pytest.
- React frontend with a map library (e.g. Leaflet) and a compliant tile provider.
- Initially in-memory or JSON seed POI data; add PostgreSQL and SQLAlchemy only when persistence is needed.
- Docker optional for local consistency. Avoid queues, Redis, and Celery in the first iteration.

## 2. System context
```text
React UI
   |
   v
FastAPI HTTP API
   |-- Trip planning service
   |    |-- POI selector
   |    |-- Route ordering / scheduler
   |    `-- Budget estimator (optional)
   |-- POI provider adapter -> Seed dataset / OSM Overpass
   `-- Routing provider adapter -> openrouteservice (or alternative)
```

## 3. Domain model
- `Trip`: id, preferences, destination_segments[] (one in MVP).
- `DestinationSegment`: destination_id, start_date, days[].
- `DayPlan`: date/day_number, time_window, stops[], legs[], warnings[].
- `Stop`: place_id, planned_start, planned_end, duration_minutes, locked_order.
- `Place`: id, name, lat, lon, category, opening_hours?, cost_estimate?, source.
- `TravelLeg`: origin_id, destination_id, distance_m, duration_minutes, geometry?, estimation_method.

This model keeps multi-city support possible without implementing it in v1.

## 4. Proposed API contract
| Method | Path | Purpose |
|---|---|---|
| `GET` | `/api/v1/health` | Health check |
| `GET` | `/api/v1/places?city=lviv&category=...` | Discover supported POIs |
| `POST` | `/api/v1/trips/generate` | Generate a day-by-day itinerary |
| `POST` | `/api/v1/trips/recalculate` | Recalculate after add/remove/reorder edits |

**Why POST for recalculation?** The MVP can be stateless: frontend sends the current itinerary and desired changes. No database or trip ID persistence is required initially.

### Example generation request
```json
{
  "destination_id": "lviv",
  "days": 2,
  "interests": ["history", "food"],
  "day_start": "10:00",
  "day_end": "18:00",
  "budget": {"amount": 3000, "currency": "UAH"}
}
```

### Example response shape (illustrative)
```json
{
  "trip": {
    "destination_id": "lviv",
    "days": [
      {
        "day_number": 1,
        "stops": [
          {"place_id": "poi_001", "start": "10:00", "end": "11:00"}
        ],
        "legs": [],
        "warnings": ["Opening hours not verified for poi_001"]
      }
    ]
  },
  "estimated_cost": {"known_total": 0, "unknown_items": 1, "currency": "UAH"}
}
```

## 5. Algorithm (first iteration)
1. Validate trip constraints with Pydantic.
2. Filter supported POIs by interests and valid coordinates.
3. Score and select candidates using deterministic relevance and diversity rules.
4. Order candidates using nearest-neighbor walking-time heuristic.
5. Allocate visit + travel durations into daily windows; omit infeasible stops with warnings.
6. Return itinerary, route geometry if available, estimate provenance, and warnings.
7. Recalculate when the user edits stops; preserve explicitly locked order.

**Explicit limitation:** Nearest-neighbor is not globally optimal; revisit algorithm quality after the MVP.

## 6. Integration principles
- Put each external API behind a provider interface.
- Set connect/read timeouts and limited retries for transient failures.
- Cache where provider terms permit; never assume unlimited requests.
- Keep keys in environment variables, not source control.
- Log failures without sensitive user data; provide meaningful API error responses.
- Mock external APIs in deterministic tests.

## 7. QA strategy
- **Unit:** preference validation, scoring, route ordering, schedule overlap, edit rules.
- **Integration:** API request/response contracts, provider adapter failure handling.
- **E2E:** create 2-day trip, remove stop, add stop, reorder, verify updated map markers and schedule.
- **Edge cases:** zero places, invalid coordinates, provider timeout, closed venue, unknown price, insufficient daily hours, duplicate stop, out-of-range days.

## 8. Non-functional requirements
- No invented POIs; distinguish missing vs known data.
- Deterministic outputs for the same seed dataset and preferences.
- Clear validation errors (`422`) and meaningful provider error responses.
- Accessibility basics: keyboard-usable form and readable itinerary independent of map.
- Baseline performance to be measured, not assumed.
