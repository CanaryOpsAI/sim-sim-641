# CMP-1446 — Contractor reviews, adjusts and sends the estimate to the customer

As a contractor, I want to adjust the proposed range if I disagree
and send it to the customer by text and email, so that the customer
hears a number from me within the hour and nothing goes out I have
not looked at.

Acceptance criteria:
- Either bound or any line item can be edited; the range updates
  immediately and the edit is recorded with my user and the time.
- Sending delivers SMS and email within 60 seconds with the range,
  my name, the submitted photos and the Legal-approved "initial
  estimate, not a quote" wording; the request shows "estimate sent".
- A send during the customer's quiet hours (9pm-8am their time) is
  queued for 8am and I am told so; an opted-out customer gets email
  only.
- Editing after sending produces a new message that supersedes the
  first; nothing changes silently.

## Implementation notes

Implementation: inbox editing of either bound or any line item,
recalculated client-side and confirmed server-side on send; the
send button is disabled while the contractor's `send_enabled` flag
is off (first pilot week). POST /api/estimates/{id}/send writes an
`estimate_sends` row (user, time, original range, sent range,
diff), renders the Legal-approved SMS and email templates with the
range, the contractor's name and -- email only -- the submitted
photos, and enqueues delivery. Customer local time 9pm-8am
schedules the send for 8am and says so in the response. STOP
opt-outs come from the provider's list; opted-out customers get
email only. A second send creates a new message with "supersedes"
wording and references the first.

Acceptance (execution-specific):
- Delivery within 60 s of send in the staging run (provider
  sandbox timestamps in the PR).
- Time-travel tests: 10pm customer time schedules for 8am and the
  response says so; 8:01am is immediate.
- An opted-out fixture number gets no SMS and the email is sent.
- Every send has an `estimate_sends` row with a non-null diff; a
  second send references the first; templates match Legal's copy
  byte-for-byte (snapshot test).
