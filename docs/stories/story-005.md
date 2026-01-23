# CMP-1450 — AI identifies job type, size band and observed elements from the photos

As office staff reading a new request, I want an AI assessment
beside the photos that says what kind of job this is, how big it
is and what it saw, so that I can triage it in under two minutes
without being a tradesperson.

Acceptance criteria:
- Within 60 seconds (p95) of submission, a request with readable
  photos shows job type (from the trade's catalogue), size band
  (small / medium / large), observed elements and a 0-100
  confidence, labelled "AI assessment -- review before quoting".
- I can mark the assessment right or wrong in one tap; the
  judgement is stored with the prompt and model versions.
- On the labelled pilot set the shipped prompt scores at least 80%
  agreement on (job type, size band) per trade.
- A request with no photos produces no assessment and no model call.

## Implementation notes

Implementation: an `assess_request` job enqueued on submission when
the request has photos (zero photos: no job, no model call). It
loads the trade's catalogue and few-shot examples from
`prompts/assessment/{trade}/v{N}.md`, sends the photos to the
hosted vision model's regional endpoint in JSON mode with a schema
(job_type, size_band, observed_elements[], confidence 0-100,
per-photo readable flag, missing_shots[]), validates with pydantic,
and stores the result in `request_assessments` with prompt_version
and model_version NOT NULL. Retries with backoff on 429/5xx, max
three, then "assessment_failed" visible to ops only. The inbox
panel beside the gallery is labelled "AI assessment -- review
before quoting" and carries right/wrong buttons writing
`assessment_feedback` (user, verdict, prompt and model versions);
it is absent from the DOM while the contractor's visibility flag is
shadow.

Acceptance (execution-specific):
- p95 enqueue-to-stored under 60 s on the staging load test (100
  requests, 3 photos each).
- The no-training flag is set on every provider call (assert on the
  mock's request body).
- One tap on right/wrong writes a feedback row and disables the
  buttons; a second tap does not write.
- `scripts/eval_assess.py` on the labelled set reports >= 80%
  agreement on (job_type, size_band) per trade for the shipped
  prompt versions; the report is attached to the release PR.
