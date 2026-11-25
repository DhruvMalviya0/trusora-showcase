# Architecture

High-level diagrams for Trusora.

## Verification model

Four questions are answered separately, each with its own status, evidence and date:

```mermaid
%%{init: {'theme':'base','themeVariables':{'primaryColor':'#0B1F4B','primaryTextColor':'#FFFFFF','primaryBorderColor':'#2F6BFF','lineColor':'#2F6BFF'}}}%%
flowchart TD
    L[Listing] --> R[Scam-signal rules]
    L --> C1[Registration<br/>Is the company in the official register?]
    L --> C2[Website<br/>Is the site the company's own?]
    L --> C3[Recruiter<br/>Is the contact email on the company domain?]
    L --> C4[Hiring channel<br/>Does the listing come from the company's own channel?]
    R --> OUT[Evidence report]
    C1 --> OUT
    C2 --> OUT
    C3 --> OUT
    C4 --> OUT
    OUT --> S[Each result: verified, possible match,<br/>mismatch, unverified or unavailable]
```

More diagrams (data flow, deployment modes) will be added here later.
