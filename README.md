# Smart Trip Planner — City Explorer

**Status:** MVP specification v0.1 (draft)  
**Initial destination:** Lviv, Ukraine  
**Approach:** Algorithm-first; extensible to multi-city trips later.

## Product goal
Help independent travelers create a feasible, personalized 1–3 day walking itinerary in Lviv without switching between maps, attraction lists, and spreadsheets.

## Documentation
- [Product requirements (PRD)](PRD.md)
- [Technical design](TECHNICAL_DESIGN.md)
- [MVP delivery plan](DELIVERY_PLAN.md)

## Decisions
- MVP supports one destination per trip, initially Lviv.
- Travelers can add, remove, and reorder stops, then recalculate the itinerary.
- No accounts, hotel booking, intercity travel, or LLM dependency in v1.
- The system must distinguish estimated from verified prices and opening hours.

## Open questions
- Exact source and licensing for Lviv POI dataset and map tiles.
- Whether budget estimation and weather are included in v1 or delivered immediately after core itinerary.
- How much itinerary state should persist without accounts.
