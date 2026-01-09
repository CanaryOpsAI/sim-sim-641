# CMP-1449 — Customer enters contact details and service address on a phone

As a homeowner requesting a quote on my phone, I want to enter my
name, phone, email and the job's address with the address completed
for me as I type, so that the contractor can reach me and knows
where the job is without a call.

Acceptance criteria:
- Name, phone, email and address are on one screen with no page
  loads; phone and email are validated inline within one second.
- Typing three characters of the address offers matching addresses;
  choosing one fills street, city, region and postal code.
- If autocomplete is unavailable, every address field can be typed
  by hand and the form still submits.
- Leaving and returning within 24 hours on the same device restores
  every entered value.

## Implementation notes

Implementation: the intake bundle is a Preact app that mounts from
a script tag on the contractor's site and as a hosted page at
intake.{tenant}.example, styles isolated in a shadow root. Contact
step with phone validation via libphonenumber and email syntax
check, both on blur, messages inline within one second. Address
step calls the Platform autocomplete proxy
(GET /api/intake/address/suggest) after three characters, 300 ms
debounce; choosing a suggestion fills street, city, region and
postal code. Proxy error or an 800 ms timeout switches the step to
manual entry with a notice, and the form still submits. Draft
state persisted to localStorage keyed by contractor, restored for
24 hours, then discarded.

Acceptance (execution-specific):
- Bundle under 120 KB gzipped on the critical path; CI fails the
  build over it.
- Playwright on iOS Safari and Android Chrome completes the step
  with autocomplete on, and again with the proxy stubbed to 503.
- Reload mid-step restores every field; after 24 h the draft is
  gone (unit test on the TTL).
- axe run on the step reports zero WCAG 2.1 AA violations.
