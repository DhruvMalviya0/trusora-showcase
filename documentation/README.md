# Documentation

Technical documentation for Trusora. Every statement was checked against the private repository on 2026-11-25; figures
that were recorded earlier and not re-run are marked with their date.

| Document | Contents |
|---|---|
| [PRODUCT_OVERVIEW.md](PRODUCT_OVERVIEW.md) | What Trusora does, the problem, target users, capabilities, workflow, current status |
| [SCAM_DETECTION.md](SCAM_DETECTION.md) | Risk-analysis pipeline, rule-based evidence, fee detection, AI-assisted reading, how evidence is presented, benchmark, limitations |
| [COMPANY_VERIFICATION.md](COMPANY_VERIFICATION.md) | MCA company and LLP registry, CIN and name matching, legal-form validation, statuses, ambiguous records, human review |
| [SECURITY.md](SECURITY.md) | Authentication, sessions, SSRF protection, secret handling, rate limiting, administrator security, security testing |
| [TESTING.md](TESTING.md) | Testing strategy, backend and front-end tests, security harness, verified counts, coverage boundaries |
| [ENGINEERING_DECISIONS.md](ENGINEERING_DECISIONS.md) | Major decisions, trade-offs, challenges, lessons |

Diagrams are in [`../architecture/`](../architecture/README.md) and screenshots in [`../screenshots/`](../screenshots/README.md).

**Status vocabulary.** *Verified* means observed working or covered by a test run on 2026-11-25. *Implemented, not
independently tested* means the code exists and is in use without an automated test. *Partial*, *Blocked* and *Not built*
are used literally.
