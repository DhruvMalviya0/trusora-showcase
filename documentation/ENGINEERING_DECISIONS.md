# Engineering decisions

The main technical decisions behind Trusora, the trade-offs they involved, and what was learned. Each entry is based on
the private repository and its internal records as inspected on 2026-11-25. These are decisions, not guarantees of
outcomes: where a decision's effect has not been measured, that is said.

## 1. Rules first, with the language model in a narrow supporting role

**Decision.** Scam signals are found by deterministic rules that return the exact text they matched. A language model
answers exactly one question (does the applicant have to pay?) and only when money wording is present and no rule has
already settled the matter.

**Why.** Rules are explainable, cheap, fast and testable offline. A student can be shown *why* a listing was flagged. A
model's free-form verdict cannot be audited, and an advert is untrusted input that may try to instruct the model.

**Trade-offs.**
- Rules miss wording they were not written for. On the one hold-out set that was never tuned against, the rules alone
  missed half of the scams (see [SCAM_DETECTION.md](SCAM_DETECTION.md#9-measured-behaviour-and-what-it-does-not-show)).
- The model's help is limited by design: its answer must include a quote that really appears in the advert, its
  findings carry less weight, and it cannot move a listing below the "check first" level alone. Whether it closes the
  gap left by the rules has **not been measured**.

## 2. Keep fraud risk, company identity, recruiter identity and evidence confidence separate

**Decision.** The report keeps four fields apart instead of producing one "trust score".

**Why.** They answer different questions. A real company can be impersonated, so a verified company must not make a
suspicious recruiter look safe. A registered company must not offset a fee demand. Merging them would hide exactly the
cases that matter.

**Trade-off.** More to explain to the user, and the interface does not yet show all four fields (see section 9).

## 3. "Unavailable" is not "not found", and "unverified" is not "fake"

**Decision.** Five statuses (verified, mismatch, unverified, conflicting, unavailable) with distinct wording. When no
registry is loaded, the report says the check *could not run*, never that the company does not exist.

**Why.** Hosted deployments have no registry, and most real listings come from foreign employers, brands and young
start-ups that will not be in an Indian register. Treating absence as evidence would produce false accusations.

## 4. Conservative company-name matching

**Decision.** Exact, alias and variant matches can verify a company; looser "possible" matches never do. Legal forms must
agree (a listing that says LLP is not verified against a private limited company of the same name). Common names with no
Indian legal form, and names whose only match is an inactive namesake, are downgraded. Records dated after the snapshot
cannot produce a verified match.

**Why.** Reviewing real matches found cases where a name-only match was wrong, such as an Indian arm of a foreign parent
or unrelated namesakes. Each rule was added after such a case.

**Trade-off.** Recall is deliberately sacrificed. In the recorded matcher benchmark, a one-letter typo or a parent brand is
never matched (it stays *unverified*) in exchange for near-zero wrong matches.

## 5. The registry is a separate, read-only, locally stored database

**Decision.** The 3.8-million-record register (about 2.33 GB) lives in its own SQLite file, opened read-only, with the
snapshot date stamped on every record and name keys versioned.

**Why.** It keeps the register out of the application database and its backups, makes accidental writes impossible, and
gives millisecond lookups on a laptop (the recorded lookup time was about 0.25 ms; not re-measured).

**Trade-off.** It cannot be loaded into the small hosted database, so hosted mode reports registration as *unavailable*
until the register is hosted somewhere. This is the main functional gap between local and hosted modes.

## 6. Import validation: never drop a row, flag it

**Challenge.** The source files contained malformed identifiers, placeholder dates, names with no usable words, mis-encoded
characters, test records, and records dated after the stated snapshot.

**Decision.** Import every row, attach quality flags, report every category, exclude placeholder and test records from
matching, and stop a file that lacks an id, name or status. The snapshot date was taken from the portal's own statement
after the inspection found records dated a day later.

**Lesson.** Real public data needs an explicit data-quality layer; deciding what to *flag* and what to *refuse* is a
product decision as much as an engineering one.

## 7. One codebase, two storage back ends

**Decision.** The same application runs locally on SQLite and hosted on PostgreSQL behind a single data-access wrapper,
with per-user profiles in hosted mode.

**Trade-off.** Dual-engine support doubles the test matrix. The PostgreSQL tests exist but were skipped in the 2026-11-25
run, so that path is the least recently verified.

## 8. Polite collection over coverage

**Decision.** Sources are read through public APIs and pages with `robots.txt` honoured, per-host rate limits and no
sign-in or bot-check circumvention. Login-walled sites are listed for manual browsing, and a pasted link is read as a
single page.

**Trade-off.** Coverage is lower than an aggressive scraper would achieve; the recorded runs found few remote,
India-open internships on company boards. The decision is a deliberate one.

## 9. Showing only part of the report

**Decision (and debt).** The back end computes the full four-field report and stores it, but the interface renders only the
score, label and reasons. The remaining fields are in the data before the interface was ready for them.

**Why it matters.** The product's central promise, an honest account of what was and was not verified, is only partly visible
today. This is listed as in progress rather than complete.

## 10. A real defect, and what it taught

A change that added the AI reading reused a variable name that already held a list of AI-screening reasons. The listing
record then stored a dictionary where the interface expected a list, which blanked a page. The fix was a **data
contract**, not a front-end patch: the new value gets its own name, the API layer guarantees the field's type for every
response including old rows, and a dedicated test module pins the contract. **Lesson:** enforce types at the boundary
between layers, and test the contract itself.

## 11. Evidence discipline for accuracy claims

**Decision.** The benchmark labels which sets were tuned against and which were not, and the repository records that only
the untuned hold-out is an honest figure. A frozen independent benchmark protocol (separate labelling, two reviewers,
one run per engine version) was written before any result existed.

**Why.** Tuning rules against a corpus and then reporting accuracy on it proves nothing. Public pages therefore quote the
untuned figure, and no real-world accuracy is claimed.

## 12. Security as architecture, not a checklist

**Decisions.** Outbound requests go through one guard that validates every resolved address at connect time; secrets are
encrypted at rest with rotation and a distinct "unreadable" state; administrators use a multi-step sign-in; rate limits
persist in the database. Details are in [SECURITY.md](SECURITY.md).

**Lesson from review.** An independent review of the encryption work found that a stored key that could not be decrypted
was read as "empty", so an unrelated settings save would have overwritten it. Introducing an explicit third state,
*unreadable*, fixed the class of problem rather than the single case.

## 13. Known follow-ups

These are open items recorded in the project's own notes, not completed work:

- Build the independent benchmark dataset (the protocol exists; the data does not).
- Render the full verification report in the interface.
- Retire the older trust score once every stored listing has been re-evaluated by the current engine, so two scales do
  not coexist.
- Host the register, or an equivalent lookup service, so hosted mode can verify companies.
