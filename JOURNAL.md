# Module 3 Journal — PathReview

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/146

**Issue title:** PII scrubber fails to redact parenthesized US phone numbers

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The PII scrubber in the `safety` module is supposed to find and hide personal
information in text before it is stored or shown. Its US phone-number regular
expression only matches dash-separated numbers like `555-123-4567`, so the very
common parenthesized area-code format `(555) 123-4567` slips through completely.
As a result `scrub()` leaves that number visible in the output and `detect()`
reports that no PII was present, which is a real privacy leak for any resume or
profile that lists a phone number that way. A successful fix updates the
`phone_us` pattern in `safety/pii_scrubber.py` so both formats (and the space
after the parentheses) are matched, making the four related tests in
`tests/unit/test_pii_scrubber.py` pass without breaking the other formats.

**Branch name:** fix/146-pii-parenthesized-phone

**Setup confirmation:** [x] App runs locally at localhost:5173
_(Verified: `make` Python env installed, `pytest tests/unit/test_pii_scrubber.py`
runs and reproduces the 4 failing phone tests; frontend Vite dev server started
and `http://localhost:5173/` returned HTTP 200.)_

**Cohort ledger:** [ ] Issue added to cohort ledger
_(To do: add name, GitHub username `SushilPoudel2005`, and issue #146 to the
section tab, then check this box.)_

### "Is this issue right for me?" — checklist reasoning

- **Scope is small and well-bounded.** The fix is contained to a single regex in
  one file (`safety/pii_scrubber.py`). No architecture or cross-module changes.
- **I can reproduce it.** The issue's repro snippet runs locally and I confirmed
  `(555) 123-4567` passes through `scrub()` unredacted while `555-123-4567` is
  redacted.
- **Tests already exist.** Four failing unit tests
  (`test_us_phone_number_redaction`, `test_us_phone_formats`,
  `test_detect_phone_pii`, `test_phone_at_start_of_text`) define "done," so I can
  verify the fix objectively with `make test-unit`.
- **I understand the domain.** Basic regex knowledge is enough; no ML, infra, or
  async concerns.
- **Effort matches the estimate.** Labeled 2–4 hours / "good first issue,"
  realistic for a first contribution to a large codebase.
- **Risk of scope creep is low.** The main thing to watch is not over-broadening
  the pattern so it starts matching non-phone digit sequences; the existing tests
  guard against that.
