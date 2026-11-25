# Screenshots

Genuine screenshots captured on 2026-11-25 from the running application at a 1440 pixel desktop width, in the
application's default dark theme. Violet is the application's own accent colour; nothing has been restyled.

## How they were made

- The application was run from an isolated copy of the private project's working tree, using its normal local
  workflow in **demo mode** (`serve.py --demo`), so the private repository and its real data were never opened by the
  screenshot session.
- The interface's built-in sample data is labelled **Demo data** in the page header. Company names carry "(sample)".
- Two extra listings were added through the interface's own **Add internship** form with synthetic text, so the
  scam-signal engine could be shown working on a flagged and on an unverified listing. Their company names are fictional
  (BrightFuture Careers and Example Labs). No real person, company, email address or key appears.
- Screenshots were not retouched. Two drawer screenshots were taken in a shorter window so only the header and flags
  show.
- The product name in the interface is currently "Internship Assistant".

| File | What it shows |
|---|---|
| [01-feed-to-apply.png](01-feed-to-apply.png) | The feed, best match first, with a verification mark on each listing |
| [02-feed-all-with-verification-marks.png](02-feed-all-with-verification-marks.png) | All listings sorted by newest: an amber *Not enough evidence* mark, two red *Likely scam* marks with reasons, and green checks |
| [03-tracker.png](03-tracker.png) | The tracker with match score, flags and status, including *Skipped (red flag)* rows |
| [04-risk-analysis-flagged-listing.png](04-risk-analysis-flagged-listing.png) | The detail panel for a synthetic fee-demanding listing: each flag in plain language |
| [05-risk-analysis-unverified-company.png](05-risk-analysis-unverified-company.png) | The detail panel for a listing whose company is not on the student's vetted list |
| [06-add-internship-form.png](06-add-internship-form.png) | Adding a listing by link or by form |
| [07-dashboard.png](07-dashboard.png) | Weekly goal, pipeline counts and applications per week (sample data) |
| [08-bot-activity.png](08-bot-activity.png) | One run: what was scanned, added and skipped, and why |
| [09-skipped-listings.png](09-skipped-listings.png) | Listings the filters rejected, with the exact triggering text and a *Track anyway* button |
| [10-platforms.png](10-platforms.png) | The sources the product reads, and how each is covered |

## What could not be screenshotted

| Screen | Why it is missing |
|---|---|
| Company registry verification results | The interface does not yet display the registry-based company, recruiter and confidence fields. They are computed and stored, but not rendered. Nothing was mocked up in their place. |
| Hosted-mode sign-in and administrator sign-in | They require a hosted deployment with email and Google credentials, which were not used. |
| Real-listing results | Real listings and the real company register were deliberately not used, to avoid exposing personal data and private data. |
| Settings and profile pages | They are blank in demo mode, so they were not included. |
