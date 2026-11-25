# Security engineering

A high-level description of the security measures built into Trusora, written for reviewers and engineers. It
describes protections observed in the private repository on 2026-11-25. Implementation details that would help an
attacker (exact limits, patterns, endpoints, secrets handling internals) are intentionally omitted.

> **Status.** Trusora has had internal security review and remediation passes. At the end of the last recorded pass
> (2026-11-12) no open critical, high or medium findings were known in the code, but **infrastructure checks for a
> hosted deployment were not complete and the product was not approved for public launch**. Nothing on this page is a
> production-readiness or penetration-test claim.

## 1. Deployment modes and trust boundaries

| Mode | Boundary |
|---|---|
| **Local** | Runs on the user's own computer, listening only on the loopback address. No sign-in. The server accepts requests only for local host names, which also blocks DNS-rebinding attacks from other websites. |
| **Hosted (multi-user)** | Users sign in; every user's data is separated by account. Administrators sign in through a stronger flow. |

## 2. Security controls

| Area | Control |
|---|---|
| Authentication | Google sign-in (verified email required) or an emailed single-use sign-in link. Link tokens are random, stored only as a hash, expire quickly, work once, are kept out of server logs by travelling in the URL fragment, are rate-limited, and the reply is the same whether or not the email is allowed. |
| Sessions | Signed, `HttpOnly`, `SameSite` cookies (`Secure` in hosted mode) with a bounded lifetime; the session is cleared when a user signs in. |
| CSRF | State-changing requests must carry a custom header that a cross-site form cannot set. |
| Authorisation | Every user-data query is filtered by the signed-in user's id; shared-configuration writes and waitlist reads are limited to administrators. |
| Administrative security | Administrator rights come only from a session that has completed a multi-step sign-in: an emailed code plus an authenticator-app code (or a recovery code), with first-time enrolment protected by a separate enrolment key. Failures return uniform messages so the flow cannot be used to discover administrator addresses. |
| Browser hardening | A restrictive Content-Security-Policy, frame blocking, `nosniff`, no referrer, and no-store caching on API responses. Static files are served only from a fixed directory. |
| Startup safety | A hosted deployment refuses to start with a weak signing secret or without its production database. |
| Rate limiting | Request limits for sensitive actions are stored in the application database, so they are shared across server workers and survive restarts. |
| Outbound request protection (SSRF) | Every request the system makes to a third-party site goes through a guard that resolves the host itself, refuses the connection unless every resolved address is a public one on an allowed port, and then connects to that exact address. Because the check happens at connect time it also covers redirects, and because the checked address is the one used it defeats DNS rebinding. |
| Polite collection | `robots.txt` honoured, per-host rate limits, an identifying user agent, retries with back-off, and no sign-in bypassing or circumvention of bot checks. |
| Input limits | Upload size caps; resume parsing is bounded and nothing is written to disk. |
| Data access | Parameterised queries throughout; dynamic identifiers come only from fixed lists. |
| Secret handling | Users' optional AI provider keys are encrypted at rest with authenticated encryption; the encryption keys live outside the database and support rotation. A key that cannot be decrypted is treated as a distinct "unreadable" state and is never used or silently overwritten. Keys are not written to logs or returned by the API. Environment and profile files are excluded from version control. |
| AI output handling | Model replies are parsed strictly, validated against enumerations, and any quoted evidence must appear in the source text before it can influence a result. |
| Registry integrity | The company register is opened read-only. |
| Demo isolation | In demonstration mode, settings changes and bot runs are refused. |
| Account deletion | A user can delete their account, which removes their rows. |

## 3. Recovery and operational safety

- Database backups and a **restore drill** are part of the automated suite (see [TESTING.md](TESTING.md)).
- A key-rotation tool re-encrypts stored provider keys, reports counts without revealing values, and leaves any
  undecryptable value untouched.
- Cloud deployment and scheduled-run configuration exist, but the deployment itself was **not verified** in this review.

## 4. Security testing

| Layer | What exists | Verified on 2026-11-25 |
|---|---|---|
| Security regression tests | A dedicated module of 70 test functions covering sign-in, sessions, request guards, outbound-request protection, headers and related areas | Passed as part of the full run (395 tests, 0 failures) |
| Administrator-authentication tests | A dedicated module of 33 test functions | Passed as part of the full run |
| Key-handling tests | Encryption, rotation, unreadable-key handling and leak checks across logs and API responses | Passed as part of the full run |
| Continuous checks | A workflow that, on every push and pull request and weekly, audits Python and front-end production dependencies and runs the security regression tests | Workflow exists; **its runs were not inspected** |
| Dynamic checks | A script for running checks against a local server | Exists; **not run** in this review |
| Independent penetration test | None | **Not done** |

## 5. Known gaps (summarised)

- Hosted infrastructure checks (managed database, hosting, scheduled runs) are not complete.
- There is no independent penetration test.
- The company register cannot be hosted in the current hosted setup, which is a functional limit rather than a security one.
- Server-side auto-apply is partial and was reviewed during remediation; it is not presented as a finished feature.
