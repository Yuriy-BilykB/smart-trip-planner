# Product Requirements Document (PRD)

**Product:** Smart Trip Planner — City Explorer  
**Version:** 0.1 — draft  
**Owner:** Product team  
**Status:** Ready for MVP scoping and validation

## 1. Problem and opportunity
Planning a short city visit requires manually combining points of interest, estimated visit times, travel distances, and opening hours. Travelers can waste time zigzagging across the city or plan more stops than fit in a day. This is a **hypothesis**, not yet validated through user research.

## 2. Target users and jobs to be done
**Primary persona (hypothesis):** Independent traveler planning a 1–3 day visit to Lviv, primarily on foot.

**Job to be done:** “When I visit a city for a few days, help me quickly create and adjust a realistic sightseeing plan so I spend less time organizing and more time exploring.”

## 3. Value proposition
Generate an editable day-by-day itinerary with real places, approximate walking time, visit durations, and clear explanations when constraints cannot be met.

## 4. Goals and non-goals
**Goals:** (1) generate a feasible plan; (2) support fast manual edits; (3) make assumptions transparent; (4) provide a useful map view.

**Non-goals for MVP:** multi-city trips, cross-border transport, booking hotels/flights, account system, AI chat, real-time public transit routing, guaranteed venue availability.

## 5. MVP scope
| ID | Feature | Priority | Description |
|---|---|---|---|
| F-01 | Trip preferences | Must | Lviv, 1–3 days, interests, preferred daily start/end time, optional budget |
| F-02 | Places discovery | Must | Select POIs from a curated and/or externally sourced Lviv dataset |
| F-03 | Itinerary generation | Must | Allocate places across days and avoid overlapping visits |
| F-04 | Route ordering | Must | Estimate walking travel time and optimize visit order heuristically |
| F-05 | Interactive editing | Must | Add, remove, reorder stops; recalculate times and route |
| F-06 | Map view | Must | Show numbered stops and route geometry where available |
| F-07 | Clear failure states | Must | Explain unavailable data, infeasible constraints, and provider failures |
| F-08 | Budget estimate | Should | Show estimated known/unknown costs, never fabricated exact prices |
| F-09 | Weather awareness | Should | Display forecast within available forecast horizon; suggest indoor options |

**Release gate:** Must-have items complete. Should-have items may follow in v1.1.

## 6. Core user journey
1. Open planner; destination defaults to Lviv.
2. Select 1–3 days, interests, start/end hours, and optional budget.
3. Submit generation request.
4. Review itinerary: stops, approximate durations, walking legs, and map.
5. Remove a stop, add a suggested place, or reorder stops.
6. Recalculate; see updated time estimates and warnings.

**Screens:** Planner form; itinerary results with day tabs and map; place picker for editing. Loading, empty, partial-data, and error states are mandatory.

## 7. User stories and acceptance criteria
### US-01 — Generate a trip
As a traveler, I want a personalized itinerary so I can explore Lviv efficiently.
- AC-01: The request supports 1, 2, or 3 days; invalid values return a validation error.
- AC-02: Returned stops are drawn from available POI records, not invented locations.
- AC-03: Each stop has a start time, end time, and estimated duration.
- AC-04: Stops do not overlap, including travel time between them.
- AC-05: When there are too few feasible places, the response returns a partial plan with warnings rather than inventing stops.
- AC-06: Unavailable routing data yields a labeled fallback estimate or a clear error.

### US-02 — Edit an itinerary
As a traveler, I want to modify stops so the itinerary matches my preferences.
- AC-07: A traveler can remove a stop and receive a recalculated plan.
- AC-08: A traveler can add a valid place if constraints allow; otherwise a clear warning is returned.
- AC-09: A traveler can reorder stops; recalculation preserves the requested order when explicitly locked.
- AC-10: The plan recalculates travel legs, arrival times, and estimated costs after an edit.

### US-03 — View the route
As a traveler, I want a map so I can understand the itinerary geographically.
- AC-11: Each itinerary stop has coordinates and a matching map marker.
- AC-12: The route line is shown only if valid geometry is available; straight-line approximations are visibly labeled.

## 8. Business rules
- A trip has exactly one destination in v1; destination is represented as a separate entity for future multi-city expansion.
- A day has a start/end window; activities and travel must fit within it.
- Route optimization uses a simple deterministic heuristic in v1, not an optimality guarantee.
- Opening hours are enforced only when trustworthy structured hours are available; unknown hours are flagged.
- Costs are estimates and can be unknown; the app must not imply that a place is free when price data is missing.
- Weather forecast is not available for arbitrary future dates; show a limitation when out of range.

## 9. Success metrics (initial hypotheses)
- ≥95% of valid requests on our supported demo dataset produce a usable itinerary.
- No overlapping stops in automated acceptance tests.
- Median itinerary generation time <10 seconds under agreed test conditions.
- Users can complete at least one edit without regenerating from scratch.
- Qualitative test: at least 5 target users can understand and modify a generated itinerary.

## 10. External dependencies and risks
- **POIs:** OpenStreetMap / Overpass. Coverage and data freshness vary; public Overpass endpoints are rate-limited and unsuitable as guaranteed production infrastructure.
- **Routing:** openrouteservice or another OSM-based routing provider; key, quotas, geographic coverage, and attribution must be verified before integration.
- **Weather (optional):** Open-Meteo; review licensing and commercial usage conditions.
- **Map tiles:** Choose a compliant tile provider and respect usage policy and attribution; do not assume public OSM tile servers support arbitrary app traffic.
- **Budget (optional):** POI prices are frequently unavailable; estimated categories need transparent assumptions.

## 11. Product risks and mitigations
| Risk | Mitigation |
|---|---|
| Sparse or inaccurate place metadata | Start with a small verified Lviv POI seed dataset; record data provenance |
| Route provider outages | Adapter layer, timeout, bounded retry, graceful fallback |
| Unrealistic itineraries | Time-window constraints, travel buffers, acceptance tests |
| Scope creep | Separate must-have release gate from future features |
| Future multi-city expansion | Model Trip → DestinationSegment → Day → Stop, but implement one segment now |

## 12. Future roadmap
- v1.1: Better opening hours, weather, budget estimates, persistence/sharing.
- v2: More cities, saved trips, richer discovery.
- v3: Multiple destination segments and intercity connections; potentially AI-assisted recommendations.

## 13. Discovery tasks before release
- Interview 5 potential users about planning pain points and current tools.
- Validate 20–30 Lviv POIs for coordinates, categories, and opening hours.
- Verify provider API keys, quotas, licensing, and attribution in current official docs.
- Test walking-route coverage and response time for central Lviv.
