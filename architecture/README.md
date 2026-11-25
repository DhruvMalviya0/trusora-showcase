# Architecture

Sanitised diagrams of the architecture that is **implemented today** in the private repository (inspected 2026-11-25).
They show components and data flow, not code. Diagrams render on GitHub.

| Diagram | Section |
|---|---|
| Overall system architecture | [1](#1-overall-system-architecture) |
| Internship risk-analysis workflow | [2](#2-internship-risk-analysis-workflow) |
| Company verification workflow | [3](#3-company-verification-workflow) |
| Registry data processing workflow | [4](#4-registry-data-processing-workflow) |

## 1. Overall system architecture

One codebase runs in two modes. **Local mode** serves a single user on their own computer with a local database and no
sign-in. **Hosted mode** serves many users, each with their own profile and tracker, on a managed database. The company
registry is available only in local mode.

```mermaid
flowchart LR
    subgraph Client
        B[Web app<br/>React and TypeScript]
    end
    subgraph Server["Web server (Python, Flask)"]
        API[JSON API]
        AUTH[Sign-in routes<br/>hosted mode only]
        STATIC[Built front-end files]
    end
    subgraph Bot["Collection and analysis package"]
        PIPE[Pipeline run]
        FETCH[Source fetchers]
        EVAL[Filters, scoring, flags]
        SHIELD[Scam-signal engine]
        APPLY[Application helpers]
    end
    subgraph Data
        SDB[(Application database<br/>SQLite local, PostgreSQL hosted)]
        REG[(Company registry<br/>read-only, local mode only)]
    end
    subgraph External
        JOBS[Job boards and feeds]
        RDAP[Public domain-age service]
        LLM[Optional AI provider]
        GOOG[Google sign-in]
        SMTP[Email service]
    end
    B --> API
    B --> STATIC
    B --> AUTH
    AUTH --> GOOG
    AUTH --> SMTP
    API --> SDB
    API -->|Run now| PIPE
    PIPE --> FETCH --> JOBS
    PIPE --> EVAL --> SHIELD
    SHIELD --> LLM
    SHIELD --> REG
    EVAL --> RDAP
    PIPE --> SDB
    API --> APPLY
```

Notes:

- Every request from the server to a third-party site passes through a guard that validates the destination address
  (see the [security overview](../documentation/SECURITY.md)).
- A scheduled run is started by the operating system's task scheduler locally, or by a scheduled workflow in hosted mode.
- The browser-assisted applier is a separate local command that drives the user's own browser; it is not part of the
  web server.

## 2. Internship risk-analysis workflow

What happens to one listing between being read from a source and being shown in the feed.

```mermaid
flowchart TD
    S[Source listing] --> N[Normalise into one listing shape]
    N --> D{Already tracked?}
    D -- yes --> U[Update machine fields only<br/>never the user's edits]
    D -- no --> F[Filters: internship, remote, region,<br/>eligibility phrases, minimum score]
    F -- rejected --> SK[Skipped listings,<br/>with the exact reason]
    F -- kept --> SC[Match score from skills and title]
    SC --> RU[Rule engine<br/>fee, contact, data, pay, schemes, names, domain age, copies]
    RU --> ID[Company identity checks]
    ID --> Q{Money wording and<br/>no settled fee?}
    Q -- yes --> AI[Optional AI fee reading<br/>quote must appear in the advert]
    Q -- no --> R
    AI --> R[Report: score, label, reasons,<br/>what could not be checked, next steps]
    R --> H{Hard red flag?}
    H -- yes --> RF[Stored as Skipped, red flag]
    H -- no --> TR[Stored as To Review]
    RF --> FEED[Feed and tracker]
    TR --> FEED
```

## 3. Company verification workflow

Four checks; the company status combines registration and website, and the recruiter status is kept separate.

```mermaid
flowchart TD
    L[Listing] --> REG[Registration check]
    L --> WEB[Website check]
    L --> REC[Recruiter check]
    L --> HIR[Hiring-channel check]
    REG --> R1{Register available?}
    R1 -- no --> RU[Unavailable]
    R1 -- yes --> R2{Identifier quoted?}
    R2 -- yes --> R3[Look up by identifier<br/>verified, mismatch or unverified]
    R2 -- no --> R4[Match by name:<br/>exact, alias, variant, possible]
    R4 --> R5[Verified only for one strong match;<br/>otherwise unverified with candidates]
    WEB --> W1[Domain is on the student's vetted list<br/>or a confirmed domain]
    REC --> C1[Email domain is a confirmed company domain]
    HIR --> H1[Fetched from the company's own system<br/>or applies on its vetted site]
    R3 & R5 & RU & W1 --> CO[Company status<br/>registration plus website]
    C1 --> RS[Recruiter status]
    H1 --> HS[Hiring-channel status]
    CO & RS & HS --> OUT[Stored with evidence, source,<br/>timestamp and limitations]
```

## 4. Registry data processing workflow

How public company and LLP records become a read-only lookup file.

```mermaid
flowchart TD
    P[Published master-data CSV files<br/>5 company files, 1 LLP file] --> V[Check header and column mapping]
    V -- missing id, name or status --> X[Refuse the file]
    V --> DR[Dry run: duplicates, malformed ids,<br/>placeholder dates, statuses]
    DR --> IM[Import in batches]
    IM --> NM[Normalise names, derive match keys,<br/>map status to a status group]
    NM --> FL[Attach quality flags<br/>no row is dropped]
    FL --> DB[(Registry database<br/>stamped with snapshot date)]
    DB --> CK[Integrity check and import report]
    DB --> LK[Read-only lookups by identifier or name key]
```

Scale (re-verified 2026-11-25): 3,824,308 records, of which 3,236,968 are companies and 587,340 are LLPs; the file is
about 2.33 GB. The snapshot date stamped on every record is 4 October 2026.
