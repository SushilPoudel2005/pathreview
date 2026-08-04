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

**Cohort ledger:** [x] Issue added to cohort ledger
_(Added name, GitHub username `SushilPoudel2005`, and issue #146 to the
section tab of the cohort ledger.)_

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

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/SushilPoudel2005/pathreview/commit/caed273688d9828b39d46f2af50307f9a94adcb6

**Reproduction summary:**
Ran `pytest tests/unit/test_pii_scrubber.py -v` and the 4 phone tests for
issue #146 fail (`test_us_phone_number_redaction`, `test_us_phone_formats`,
`test_detect_phone_pii`, `test_phone_at_start_of_text`). A direct REPL check
confirms `scrub("(555) 123-4567")` and `scrub("+1 555 123 4567")` return the
number unchanged and `detect()` returns `[]`, because the `phone_us` regex's
`[-.]?` separators never match the space after the `)` / between groups.

**PLAN.md link:** https://github.com/SushilPoudel2005/pathreview/blob/fix/146-pii-parenthesized-phone/PLAN.md

**Walkthrough video (recommended):**

**Blockers or open questions:**
None blocking. Noting for Week 9: `test_mixed_pii_and_text` also fails, but from
the unrelated over-greedy `street_address` regex (it redacts "Python"), not from
issue #146 — I'll keep that out of scope for this fix.

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
Recorded the pre-change baseline (`make test-unit`: 53 failed / 375 passed;
`make check`: ~178 pre-existing ruff findings across unrelated modules).
Implemented the core fix from PLAN.md sub-task 1: widened each `phone_us`
separator from `[-.]?` to `[-.\s]?` in `safety/pii_scrubber.py`. Verified at the
REPL that all four formats (`555-123-4567`, `(555) 123-4567`, `555.123.4567`,
`+1 555 123 4567`) now redact and that a bare SSN / version string is not
misclassified (sub-tasks 2 and 4).

**Next steps:**
Finish PLAN.md sub-task 3 (add regression tests + full-suite regression check)
and sub-task 5 (remove the Week-8 BUG marker, run `make check`/`make test-unit`,
open the PR).

**Blockers:**
None.

---

### Check-in 2 (end of week)

**PR link:** https://github.com/ascherj/pathreview/pull/717

**Branch:** `fix/146-pii-parenthesized-phone`

**What you built:**
Widened the `phone_us` regex separators to also accept a single space
(`[-.]?` → `[-.\s]?`), so `(555) 123-4567` and `+1 555 123 4567` are now redacted
by `scrub()` and found by `detect()` alongside the dash/dot formats. The digit
group anchoring (`{3}{3}{4}`) is unchanged, so SSNs and version numbers are not
misclassified as phone numbers.

**Tests added or updated:**
`tests/unit/test_pii_scrubber.py` — added three regression tests
(`test_parenthesized_phone_digits_fully_removed`,
`test_space_separated_plus_one_phone_detected`,
`test_phone_fix_does_not_misclassify_ssn`) and gave two pre-existing
assertion-less tests real assertions so the file is ruff-clean.

**Self-review confirmation:** [x] make check passes  [x] make test-unit passes
_"Passes" per the pre-existing-failures rule: my changes introduce **no new
failures**. Baseline `make test-unit` was 53 failed / 375 passed; after my
change it is 49 failed / 382 passed — a set-diff confirms the only difference is
the 4 #146 phone tests now passing. The two changed files are ruff/black-clean
and `mypy safety/` passes; the remaining repo-wide `make check`/`make test-unit`
failures (including `test_mixed_pii_and_text`, an unrelated greedy
`street_address` regex) are pre-existing and documented in the PR._

**Draft PR feedback received from:** none (opened ready for review; happy to
incorporate peer/mentor feedback in Slack before merge)
