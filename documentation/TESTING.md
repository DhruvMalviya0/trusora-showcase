# Testing

How Trusora is tested, with the numbers that were verified for this repository.

## 1. Verified results

The full Python suite was run on **2026-11-25** from a clean copy of the private repository's working tree (no company
registry file, no AI provider keys, no database content from the real installation):

| Result | Value |
|---|---|
| Tests executed | **395** |
| Failures and errors | **0** |
| Skipped | **11** (all require a PostgreSQL test database that was not configured) |
| Duration | about 45 seconds |

Notes on these numbers:

- Earlier documentation in the private repository recorded 278 tests (2026-11-12). The suite has grown since; the figure
  above is the one measured on 2026-11-25.
- The test functions *defined* across the 14 test modules number 377. The executed total is higher because the
  PostgreSQL module re-runs the multi-user and edit-preservation test classes against a second database engine.
- Eleven tests are skipped in this environment, so the PostgreSQL code path was **not exercised** in this run.

## 2. Testing strategy

| Principle | How it shows up |
|---|---|
| **Offline and deterministic** | Tests use fake HTTP responses and a scripted fake language model; no test calls a real job board or AI provider. |
| **Pure rules are tested directly** | Scam-signal checks are pure functions of text, so they are tested with hostile and obfuscated inputs without a database. |
| **Contracts are pinned** | Tests assert the shape of data that crosses layers (for example, the list fields the front end expects), after a real defect where one layer's output overwrote another's. |
| **Safety paths are tested as first-class features** | Rate limits, token single-use, user isolation, key encryption and "unavailable is not the same as not found" all have dedicated tests. |
| **Benchmarks guard behaviour, not accuracy** | A labelled corpus checks that rule changes do not silently change outcomes; it is not a measure of real-world accuracy (see below). |
| **Recovery is drilled** | A backup-and-restore drill and a key-rotation drill run inside the suite. |

## 3. Backend tests

| Module | Functions | Covers |
|---|---:|---|
| Core pipeline | 49 | Internship detection, remote and region rules, hours and pay, scoring, evaluation, feed parsers, manual import, edit preservation, user isolation, URL normalisation |
| Scam-signal rules | 36 | Fee, contact, pay and claim checks; scoring rules; hostile text; pipeline wiring; benchmark guard |
| AI fee reading | 35 | Output validation, quote verification, trigger gating, cache, limits, failures, hybrid scoring (fake model) |
| Company identity | 56 | The four checks, importer validation, match levels, common-name and legal-form rules, post-snapshot handling, separate report fields |
| Copied postings, company names, blocklist | 11 | Template detection, training-style names, blocklist and re-check status rules |
| Accounts and activity | 6 | Inactive-account pausing, instant matching from cache |
| Auto-apply | 23 | Login, CAPTCHA and external-form detection; helper parsing; flow with mocked HTTP; API routes |
| Browser-assisted apply | 24 | Fill plans, field values, login and success detection, database updates; no real browser |
| Web routes and launch features | 18 | Feed card, legacy trust score, resume parsing, route guards |
| Security regression | 70 | See [SECURITY.md](SECURITY.md) |
| Administrator authentication | 33 | Multi-step sign-in and enrolment |
| Database schema | 6 | Schema consistency and migrations |
| Tracker data contract | 8 | The shape of data the interface depends on |
| PostgreSQL parity | 2 (re-run as 20) | Multi-user and edit tests on PostgreSQL; **skipped** without a test database |

The module-level function counts above were counted from the test files on 2026-11-25; they are not a per-test
coverage measure.

## 4. Front-end tests

The web interface has **browser-level checks** (a headless browser against a demo server) covering the application
shell, shared components, feed, tracker, detail panel, dashboard, activity log, platforms and companies, settings, public
pages, a sweep of every route in both themes (console errors, failed requests, overflow, contrast), and an accessibility
scan at desktop and phone sizes in both themes.

**These checks were not run for this repository** (they need a browser engine and an accessibility library that were not
set up in the review environment). The screenshots in this repository were captured separately and are not test
output. The front end has **no unit-test suite**.

## 5. The scam-signal benchmark

Re-run on 2026-11-25 (matches the figures recorded on 2026-11-12):

| Set | Scams | Legitimate | Recall (scam marked risky) | Legitimate marked risky |
|---|---:|---:|---:|---:|
| Tuned corpus | 36 | 36 | 100% | 0% |
| Hold-out A | 15 | 15 | 100% | 0% |
| Hold-out B | 12 | 12 | 91.7% | 0% |
| **Hold-out C (never tuned against)** | 10 | 10 | **50.0%** | 0% |

Only hold-out C is an honest figure for unseen wording, and it is small (20 hand-written listings). The benchmark runs
rules only, with no AI reading. **No real-world accuracy figure exists.** A frozen, independent benchmark protocol has
been written but its dataset has not been built.

A separate matcher benchmark for company names is described in
[COMPANY_VERIFICATION.md](COMPANY_VERIFICATION.md#10-measured-behaviour-recorded-2026-11-12-not-re-run-on-2026-11-25). It was
**not re-run** for this repository.

## 6. Security harness

See [SECURITY.md](SECURITY.md#4-security-testing). In short: a dedicated regression module and an administrator-
authentication module run as part of the suite, a workflow runs dependency audits and the security module on every push
and weekly, and a dynamic-check script exists. The workflow's history and the dynamic script were not inspected or run
for this repository, and no independent penetration test has been done.

## 7. Test coverage boundaries

- **No coverage percentage is measured or claimed.**
- No test uses a real job board, AI provider, mail server or Google sign-in.
- Registry-backed checks are tested against small built registries, not the 3.8-million-record file. Tests that read
  the real register in the author's environment were not part of the clean-copy run.
- The hosted deployment, scheduled cloud run and PostgreSQL path are not exercised by this run.
- Fetchers for several job boards have no test that parses their real responses; they are verified only by live runs.
- Browser-assisted apply is unit-tested without a real browser.
- Prompt-injection robustness of the AI reading is not separately tested against adversarial adverts.
