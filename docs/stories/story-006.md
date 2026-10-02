# CMP-1447 — Low-confidence assessment asks for specific additional photos

As office staff, when the AI cannot tell what the job is from the
photos, I want it to say so and say which photos would help, so
that I can ask the customer for exactly those instead of guessing
or scheduling a visit.

Acceptance criteria:
- Photos that are all dark, blurred or of the wrong thing produce
  a "needs more photos" state naming the missing shots, not a job
  type.
- Below the configured confidence threshold the assessment is
  shown as "needs review" and is never presented as a job type
  alone.
- The requested shots are worded in the trade's guidance
  vocabulary so they can be sent to the customer as-is.
- On the labelled set, at least 70% of "needs more photos" flags are
  ones the contractor agrees with.

## Implementation notes

Implementation: the assessment job maps the model's output to a
state: no readable photo -> "needs_more_photos" with missing_shots;
confidence below the trade's threshold (config, starting 65) ->
"needs_review"; otherwise "assessed". Photos the model reports as
not matching the chosen job type set "photos_do_not_match" with the
job type it did see. missing_shots are rendered through the trade's
guidance vocabulary table so the inbox shows the same phrases the
customer saw ("the whole roof from the street"), copyable as a
message. The eval harness reports the needs-more-photos agreement
rate alongside the job-type agreement.

Acceptance (execution-specific):
- Three all-black fixture photos store "needs_more_photos" with at
  least one missing shot and no job type, after exactly one model
  call.
- A fixture at confidence 60 stores "needs_review" and the panel
  never shows a bare job type for it (Playwright).
- Every rendered missing shot matches a guidance-vocabulary entry
  (unit test over the mapping table).
- Harness report shows >= 70% agreement on needs-more-photos flags
  for the labelled set.
