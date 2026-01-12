# CMP-1443 — Trade-specific questions adapt to the chosen job type

As a customer, I want to pick what kind of job I need (moving,
roofing, electrical) and answer a few questions specific to that
trade, so that the contractor gets the details they need for this
kind of work and I am not asked about things that do not apply.

Acceptance criteria:
- The job types offered are the ones the contractor has configured;
  a contractor with one trade skips the choice.
- Choosing a trade shows its question set (three to five questions)
  and hides the others; changing the trade swaps the questions and
  keeps contact details.
- Every answer appears in the contractor's inbox labelled with its
  question, in the order asked.
- A required question left blank shows an inline message and does
  not clear other answers.

## Implementation notes

Implementation: question sets live in `trade_question_sets`
(trade, version, questions JSON), seeded for moving, roofing and
electrical from Customer Success's signed-off copy.
GET /api/intake/config/{contractor} returns the contractor's enabled
trades and their current question sets; a contractor with one trade
skips the choice in the UI. The form renders the chosen trade's
three to five questions and swaps them on change while keeping
contact details. POST /api/intake/requests validates answers
against the question-set version the form was served, stores them
keyed by question id, and de-duplicates by phone number within ten
minutes. The inbox renders answers in question order with the
address on a map and the submission time in the contractor's
timezone.

Acceptance (execution-specific):
- Contract tests for both endpoints against the OpenAPI spec; an
  unknown question id or stale set version is a 422 naming the
  field.
- Duplicate submission within ten minutes from the same phone
  returns the original request id and creates no second row.
- The inbox shows a new request within 30 seconds of submission
  (end-to-end through the embed).
- A request from an EU contractor is written to the EU database
  only (integration test on the connection used).
