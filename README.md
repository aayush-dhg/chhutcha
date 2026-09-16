# ChhutCha

**ChhutCha** is a Nepal-focused coupon and deals marketplace that helps customers discover trustworthy discounts from local stores, restaurants, service providers, and online businesses operating in Nepal.

Businesses will be able to publish and manage their own offers, while ChhutCha provides discovery, validation, redemption, moderation, and performance insights through one scalable platform.

> Domain: [chhutcha.com](https://chhutcha.com)  
> Status: Planning and architecture  
> Initial product: Responsive web application  
> Future clients: Android and iOS applications

## Vision

Make saving money simple for customers in Nepal while giving businesses an accessible, measurable way to attract and retain customers.

## Who ChhutCha serves

### Customers

Customers can:

- Discover nearby, popular, newest, and expiring-soon deals.
- Search by business, keyword, category, location, or online availability.
- View discount value, validity, branches, exclusions, and terms.
- Save favorite businesses and coupons.
- claim coupons using unique codes or QR codes.
- Redeem offers online or at a participating branch.
- Review redemption history and receive expiry notifications.
- Report inaccurate, expired, or misleading offers.

### Businesses

Businesses can:

- Register a business profile and complete verification.
- Add PAN/VAT, contact, website, social, and branch information.
- Create percentage, fixed-amount, buy-one-get-one, free-item, and promo-code offers.
- Configure validity dates, schedules, branches, audience, minimum spend, inventory, and per-user limits.
- Upload images, terms, exclusions, and redemption instructions.
- Pause, resume, edit, duplicate, or expire a campaign.
- Validate unique codes or scan QR codes.
- Track views, saves, claims, redemptions, conversion, and campaign performance.
- Manage team members and future subscription plans.

### Administrators

Administrators can:

- Review business verification and coupon submissions.
- Moderate content and detect duplicate, fraudulent, or misleading offers.
- Manage users, businesses, categories, locations, reports, and disputes.
- Suspend accounts or offers and maintain an audit trail.
- Feature selected deals and review platform analytics.

## Coupon information model

Each coupon should support:

- Title and short description
- Business and participating branches
- Category and tags
- Offer type and discount value
- Original price and offer price, when applicable
- Start date, expiry date, and usable days/hours
- In-store, online, or hybrid redemption
- Coupon code, promo code, or QR redemption
- Total redemption inventory
- Per-customer usage limit
- Minimum purchase and maximum discount
- Eligibility and geographic restrictions
- Terms, exclusions, cancellation, and refund conditions
- Images and business contact details
- Approval, publishing, pause, expiry, and rejection status

## MVP scope

The first release focuses on the smallest dependable marketplace:

1. Customer and business authentication
2. Business profiles, verification, and branches
3. Coupon creation and admin approval
4. Browse, search, category, and location filters
5. Coupon detail pages with complete terms
6. Save and claim actions
7. Unique-code and QR-based redemption
8. Redemption limits and basic fraud protection
9. Customer, business, and admin dashboards
10. Email/SMS notifications and basic analytics

## Product roadmap

### Phase 0 — Foundation

- Confirm customer and business workflows
- Define brand system and responsive UX
- Establish repository, environments, CI/CD, logging, and analytics
- Define privacy, content, verification, and coupon policies

### Phase 1 — Web MVP

- Build the responsive customer marketplace
- Add business onboarding and coupon management
- Add admin review and moderation
- Launch secure claim and redemption flows
- Pilot with a small group of verified Nepal-based businesses

### Phase 2 — Marketplace growth

- Reviews, ratings, referrals, collections, and personalized alerts
- Map-based and proximity discovery
- Business campaign analytics
- Featured placement and subscription plans
- eSewa and Khalti integration where payment is required
- Nepali-language localization and expanded regional coverage

### Phase 3 — Mobile and scale

- Release Android and iOS applications
- Use the same versioned API as the web client
- Add push notifications, wallet passes, and deeper personalization
- Improve fraud detection, recommendations, and search
- Scale services independently when real traffic requires it

## Architecture approach

ChhutCha starts as a **modular monolith** with clear domain boundaries. This keeps the first release fast to build and simple to operate while preserving a path to extract high-load modules into services later.

Core domains:

- Identity and access
- Business and branch management
- Coupon lifecycle
- Discovery and search
- Claims and redemptions
- Notifications
- Moderation and administration
- Analytics and billing

The responsive web app, future mobile apps, and admin portal communicate through a versioned API. PostgreSQL is the system of record. Redis supports caching, rate limits, and short-lived redemption state. Object storage holds images and documents. A search engine can be introduced when database-backed search is no longer sufficient.

See:

- [Technical architecture](docs/architecture.md)
- [Interactive architecture view](architecture.html)

## Suggested technology stack

| Layer | Initial choice | Purpose |
|---|---|---|
| Web | Next.js + TypeScript | Customer, business, and admin experiences |
| API | NestJS + TypeScript | Versioned REST API and domain modules |
| Database | PostgreSQL | Transactional system of record |
| Cache | Redis | Caching, OTPs, rate limits, and redemption locks |
| Files | S3-compatible object storage | Coupon images and verification documents |
| Search | PostgreSQL initially; OpenSearch later | Keyword, category, business, and location discovery |
| Jobs | Queue-backed workers | Notifications, expiry, analytics, and media processing |
| Auth | Email/phone OTP plus secure sessions/JWT | Customer and business identity |
| Observability | Structured logs, metrics, traces, alerts | Reliability and incident diagnosis |
| Deployment | Containers + managed cloud services | Repeatable environments and controlled scaling |

Technology choices remain proposals until implementation begins.

## Key workflows

### Business publishes a coupon

1. Business signs in and completes its profile.
2. Verification documents and branch details are submitted.
3. Business creates a coupon with rules, inventory, dates, and terms.
4. Validation catches incomplete or conflicting settings.
5. An administrator approves, rejects, or requests changes.
6. The approved coupon is indexed and published.
7. Scheduled jobs automatically activate and expire it.

### Customer redeems a coupon

1. Customer discovers an active coupon.
2. The API confirms eligibility, inventory, dates, and usage limits.
3. ChhutCha issues a short-lived unique code or QR token.
4. The business validates the token at checkout.
5. Redemption is recorded atomically to prevent reuse.
6. Both parties receive confirmation and analytics are updated.

## Security and trust

- Role-based access for customers, business staff, and administrators
- OTP throttling, rate limiting, secure cookies/tokens, and session revocation
- Encryption in transit and at rest
- Atomic redemption and idempotency controls
- Signed, expiring QR tokens rather than reusable static codes
- Business verification and manual review
- Audit logs for approvals, edits, redemption, and administrative actions
- Document access controls and minimal retention
- Abuse reporting, suspension, and dispute workflows
- Backups, recovery testing, monitoring, and alerting

## Proposed repository structure

```text
chhutcha/
├── apps/
│   ├── web/                 # Customer and business web application
│   ├── admin/               # Administration portal
│   └── api/                 # Versioned backend API
├── packages/
│   ├── ui/                  # Shared components and design tokens
│   ├── contracts/           # Schemas, DTOs, and generated API types
│   └── config/              # Shared linting and TypeScript config
├── docs/
│   └── architecture.md
├── infrastructure/          # Deployment and environment definitions
├── architecture.html
└── README.md
```

This structure is a target for implementation; the repository currently begins with product and architecture documentation.

## Initial success measures

- Verified businesses onboarded
- Active, approved coupons
- Search-to-detail and detail-to-claim conversion
- Successful redemption rate
- Repeat customers and saved businesses
- Expired/invalid coupon report rate
- Business retention and campaign reuse
- API availability and redemption latency

## Important product decisions still to validate

- Guest browsing versus account-required claiming
- Verification requirements for different business types
- Whether ChhutCha processes payments or only enables discounts
- Monetization through subscriptions, featured placement, commission, or a hybrid model
- Nepali/English localization priority for MVP
- Redemption fallback when a merchant has weak connectivity
- Data residency, privacy, tax, and consumer-protection requirements

## Contributing

The project is in its planning stage. Use issues for product decisions, technical proposals, bugs, and scoped implementation work. Keep credentials and customer/business data out of the repository.

## License

A license has not yet been selected. Until one is added, all rights are reserved.
