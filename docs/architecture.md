# ChhutCha Technical Architecture

This document describes the proposed starting architecture for ChhutCha. The system begins as a modular monolith and is designed to support a responsive web product first, followed by Android and iOS clients through the same versioned API.

## System context

```mermaid
flowchart TB
    C[Customer]
    B[Business staff]
    A[ChhutCha administrator]
    P[ChhutCha platform]
    X[External services]

    C -->|Discover, claim, redeem| P
    B -->|Publish, validate, analyze| P
    A -->|Verify, approve, moderate| P
    P -->|OTP, messages, maps, payments| X
```

## Container architecture

```mermaid
flowchart TB
    subgraph Clients
        WEB[Responsive web app]
        ADMIN[Admin portal]
        MOBILE[Future Android and iOS]
    end

    EDGE[CDN, WAF and load balancer]
    API[Versioned API - modular monolith]
    WORKER[Background workers]
    DB[(PostgreSQL)]
    CACHE[(Redis)]
    FILES[(Object storage)]
    SEARCH[(Search index - later)]
    EXT[SMS, email, maps, eSewa and Khalti]

    WEB --> EDGE
    ADMIN --> EDGE
    MOBILE --> EDGE
    EDGE --> API
    API --> DB
    API --> CACHE
    API --> FILES
    API --> SEARCH
    API -->|Queue jobs| WORKER
    WORKER --> DB
    WORKER --> CACHE
    WORKER --> EXT
```

## Application modules

```mermaid
flowchart LR
    IAM[Identity and access]
    BIZ[Businesses and branches]
    COUPON[Coupon lifecycle]
    DISC[Discovery and search]
    REDEEM[Claims and redemption]
    ADMIN[Moderation and disputes]
    NOTIFY[Notifications]
    INSIGHT[Analytics and billing]

    IAM --> BIZ
    BIZ --> COUPON
    COUPON --> DISC
    COUPON --> REDEEM
    ADMIN --> BIZ
    ADMIN --> COUPON
    REDEEM --> NOTIFY
    DISC --> INSIGHT
    REDEEM --> INSIGHT
```

Modules share one deployment and database initially, but communicate through defined application interfaces and domain events. A module must not directly own or mutate another module's tables.

## Core responsibilities

| Module | Responsibilities |
|---|---|
| Identity and access | Registration, phone/email OTP, sessions, roles, permissions, account recovery |
| Businesses and branches | Business profiles, PAN/VAT/KYC verification, staff, branches, hours, online presence |
| Coupon lifecycle | Offer types, rules, terms, inventory, scheduling, approval, publishing, expiry |
| Discovery and search | Categories, keywords, location, online/local filters, ranking, featured deals |
| Claims and redemption | Eligibility, unique codes, QR tokens, atomic redemption, limits, fraud signals |
| Moderation and disputes | Reviews, reports, approvals, audit trail, suspension, customer/business disputes |
| Notifications | OTP, approval status, saved-deal alerts, expiry reminders, redemption receipts |
| Analytics and billing | Views, claims, conversions, merchant insights, plans, featured placement |

## Coupon lifecycle

```mermaid
stateDiagram-v2
    [*] --> Draft
    Draft --> PendingReview: Submit
    PendingReview --> ChangesRequested: Needs changes
    ChangesRequested --> Draft: Edit
    PendingReview --> Approved: Approve
    PendingReview --> Rejected: Reject
    Approved --> Scheduled: Starts later
    Approved --> Active: Starts now
    Scheduled --> Active: Start time
    Active --> Paused: Business or admin pauses
    Paused --> Active: Resume
    Active --> Expired: End time or inventory exhausted
    Active --> Suspended: Policy or fraud action
    Rejected --> [*]
    Expired --> [*]
    Suspended --> [*]
```

## Redemption sequence

```mermaid
sequenceDiagram
    actor Customer
    participant Client
    participant API
    participant Redis
    participant DB
    actor Merchant

    Customer->>Client: Claim coupon
    Client->>API: POST /v1/coupons/{id}/claims
    API->>DB: Check coupon, customer and limits
    API->>Redis: Create expiring claim lock
    API->>DB: Store claim and signed token
    API-->>Client: Display code or QR
    Merchant->>API: Validate token
    API->>DB: Atomic redeem if still eligible
    API->>Redis: Invalidate claim lock
    API-->>Merchant: Redemption confirmed
    API-->>Client: Receipt and updated status
```

The final write must use a database transaction, row-level constraint or equivalent atomic operation, and an idempotency key. A screenshot of a QR code must not be sufficient to redeem repeatedly.

## Initial data model

```mermaid
erDiagram
    USER ||--o{ BUSINESS_MEMBER : joins
    BUSINESS ||--o{ BUSINESS_MEMBER : employs
    BUSINESS ||--o{ BRANCH : operates
    BUSINESS ||--o{ COUPON : publishes
    COUPON }o--o{ BRANCH : valid_at
    USER ||--o{ SAVED_COUPON : saves
    COUPON ||--o{ SAVED_COUPON : is_saved
    USER ||--o{ CLAIM : creates
    COUPON ||--o{ CLAIM : generates
    CLAIM ||--o| REDEMPTION : becomes
    USER ||--o{ REPORT : submits
    COUPON ||--o{ REPORT : receives
    BUSINESS ||--o{ AUDIT_EVENT : produces
```

Additional supporting entities will include categories, locations, media, verification documents, notification preferences, plans, subscriptions, and analytics events.

## API outline

All public clients use a versioned interface such as `/api/v1`.

| Area | Example endpoints |
|---|---|
| Authentication | `POST /auth/otp/request`, `POST /auth/otp/verify`, `POST /auth/logout` |
| Discovery | `GET /coupons`, `GET /coupons/{slug}`, `GET /businesses/{slug}` |
| Customer | `POST /coupons/{id}/save`, `POST /coupons/{id}/claims`, `GET /me/claims` |
| Business | `POST /businesses`, `POST /businesses/{id}/coupons`, `GET /businesses/{id}/analytics` |
| Redemption | `POST /redemptions/validate`, `POST /redemptions/confirm` |
| Admin | `GET /admin/review-queue`, `POST /admin/coupons/{id}/decision` |

OpenAPI should be the source of truth for validation, documentation, and generated mobile/web client types.

## Deployment environments

```mermaid
flowchart LR
    DEV[Local development] -->|Pull request| CI[CI checks]
    CI -->|Merge| STAGE[Staging]
    STAGE -->|Approval| PROD[Production]
    PROD --> OBS[Logs, metrics, traces and alerts]
```

CI should run formatting, linting, type checks, unit tests, API contract tests, migration checks, dependency scanning, and builds. Production deployments should use backward-compatible migrations and a tested rollback procedure.

## Reliability and security controls

- CDN/WAF, TLS, secure headers, request-size limits, and API rate limits
- Passwordless OTP or securely hashed credentials with multi-factor support for privileged roles
- Role- and business-scoped authorization on every protected action
- Encrypted secrets and restricted verification-document access
- Signed short-lived redemption tokens and replay protection
- Transactional inventory and per-user usage enforcement
- Immutable audit events for sensitive business and admin actions
- Automated backups with recovery drills
- Structured logging without OTPs, tokens, or personal documents
- Health checks, service-level indicators, tracing, and actionable alerts
- Dependency, container, and application security scanning

## Scale path

Do not split services prematurely. Scale the stateless API and workers horizontally first. Introduce read replicas, a dedicated search index, CDN image transformations, and queue partitions based on measured load.

Likely future extraction candidates are:

1. Search and recommendations
2. Notifications
3. Analytics ingestion
4. Redemption and fraud controls
5. Billing

Extraction should follow observed bottlenecks, independent release needs, or security boundaries—not projected traffic alone.

## Nepal-specific integration considerations

- Normalize Nepali phone numbers and support reliable OTP providers.
- Store Nepal time explicitly and test coupon boundaries in `Asia/Kathmandu`.
- Support province, district, municipality, and branch-level discovery.
- Design for Nepali and English text, including Unicode search.
- Allow merchant-side redemption fallback for intermittent connectivity with strict reconciliation controls.
- Treat eSewa and Khalti as optional payment integrations; the coupon MVP need not hold customer funds.
- Validate PAN/VAT/KYC, privacy, tax, advertising, and consumer-protection requirements before launch.

## Architecture decisions to record next

- ADR-001: Modular monolith for MVP
- ADR-002: Authentication and OTP provider
- ADR-003: Claim and redemption token design
- ADR-004: Search implementation and ranking
- ADR-005: Hosting region and deployment platform
- ADR-006: Offline merchant redemption policy
- ADR-007: Payment boundary and monetization
