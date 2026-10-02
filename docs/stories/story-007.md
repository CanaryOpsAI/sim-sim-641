# CMP-1442 — Contractor maintains a rate card and sees a proposed range per request

As a contractor, I want to enter my prices per job type and size
band once, and see a proposed price range with line items on every
assessed request, so that I can respond with a number in minutes
instead of working it out from scratch.

Acceptance criteria:
- The rate card holds a low and high price per (job type, size
  band) plus the trade's modifiers (per square, per room, per
  flight of stairs, per circuit); it is private to the contractor.
- A request with an assessment shows a proposed range, the line
  items behind it, and the rate card's last-edited date.
- A job type missing from the rate card shows the assessment, asks
  for a manual price, and notes the gap on the rate card.
- A high bound over 2x the low is flagged for the contractor to
  narrow.

## Implementation notes

Implementation: `rate_cards` (contractor, trade, job_type,
size_band, low, high, updated_at, updated_by) and
`rate_card_modifiers` (contractor, trade, modifier key, unit
price), tenant-scoped; a settings grid that saves on blur.
`estimate_for(request)` reads the assessment, looks up (job_type,
size_band) on the contractor's card, applies modifiers from
observed elements and answers (squares, rooms, flights of stairs,
circuits), and returns a range with line items and the card's
updated_at. A missing job type returns "price manually" and
records the gap on the card; a high bound over 2x the low returns a
`wide_range` flag. The inbox shows the range, line items, card date
and the flag.

Acceptance (execution-specific):
- Property tests: line items always sum to the bounds; bounds are
  non-negative; the result is independent of answer order.
- Golden tests: the pilot roofer's card and 20 labelled requests
  reproduce the ranges Customer Success agreed with them.
- A query for contractor A cannot read B's card (two-tenant
  integration test).
- Editing a price updates updated_at and updated_by; the grid shows
  the saved state (Playwright).
