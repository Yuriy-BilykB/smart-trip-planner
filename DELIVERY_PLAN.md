# MVP Delivery Plan — Learning-oriented

## Phase 0 — Validation and setup (30–60 min)
- Review PRD; confirm must-have scope.
- Choose and inspect a small Lviv POI dataset.
- Initialize Python environment, FastAPI app, pytest.
- Define request/response models and API error conventions.

## Phase 1 — First 2–3 hours: usable backend slice
- `GET /health` and `POST /trips/generate`.
- 10–20 seeded POIs with coordinates, categories, and estimated durations.
- Deterministic selection + simple scheduling.
- 5–8 pytest cases including overlap and infeasible time windows.
- Swagger/OpenAPI demo. No external APIs required yet.

**Definition of done:** a 1-day Lviv plan is generated from real seeded places, does not overlap, and passes tests.

## Phase 2 — Editing and routing
- Add `/trips/recalculate` for remove/add/reorder.
- Add routing provider adapter with mocked tests and failure fallback.
- Implement simple React planner and itinerary list.
- Show map markers and routes with correct attribution.

## Phase 3 — MVP release candidate
- Expand 1–3 day support and data quality checks.
- Add clear empty/error states and provider observability.
- Add budget/weather only if must-have release criteria are met.
- Run QA checklist and 5-user usability test.

## AI-first learning method
For each feature: define user outcome → draft acceptance criteria → implement with AI assistance → explain the code and Python internals → write tests → review trade-offs.

## Immediate first ticket
**STP-001: Generate a one-day Lviv itinerary from a local POI seed dataset.**

Acceptance criteria:
1. Accept valid interests and day time window.
2. Reject invalid input with FastAPI validation.
3. Return a list of stops with non-overlapping time ranges.
4. Return warnings when requested constraints cannot be satisfied.
5. Include at least 5 deterministic unit/API tests.
