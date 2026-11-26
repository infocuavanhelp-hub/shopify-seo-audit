# Development plan

Status: **planning**. No application code exists yet. This document records the research done before development, the decisions it led to, and the milestones with acceptance criteria. Anything not confirmed against an official source is marked **Unverified**.

Research date: 26 November 2026.

## 1. Research findings

### 1.1 Shopify platform

| Topic | Finding | Source |
| --- | --- | --- |
| API version | Shopify releases a new API version each quarter; each stable version is supported for at least 12 months. Latest stable at research time: **2026-10**. Supported: 2026-01, 2026-04, 2026-07, 2026-10. 2027-01 is a release candidate. | [API versioning](https://shopify.dev/docs/api/usage/versioning) |
| Which API | Use the **GraphQL Admin API** (versioned). | same |
| Rate limits | Cost-based leaky bucket per app + store. Restore rate: 100 points/s (Standard), 200 (Advanced), 1,000 (Plus), 2,000 (Enterprise). Single queries over 1,000 requested points are rejected. Cost and throttle status are returned in `extensions.cost`. Bulk operations are recommended for large reads and are not subject to the per-query cost limit. | [GraphQL Admin API rate limits](https://shopify.dev/docs/apps/build/apis/graphql-admin/rate-limits) |
| App framework | Shopify's current app template uses **React Router v7+** (forked from the Remix template), Prisma for session storage (SQLite default; Postgres/MySQL for multi-instance), `shopify.authenticate.admin(request)` for authenticated Admin API access. MIT licensed. Created with `shopify app init --template=https://github.com/Shopify/shopify-app-template-react-router`. | [Template repo](https://github.com/Shopify/shopify-app-template-react-router), [shopify-app-react-router docs](https://shopify.dev/docs/api/shopify-app-react-router) |
| Scopes (read) | `read_products` covers Product, ProductVariant, Collection. `read_content` covers Page, Blog, Article (`read_online_store_pages` is an alternative). `read_files` covers MediaImage. `read_online_store_navigation` covers UrlRedirect. | [Access scopes](https://shopify.dev/docs/api/usage/access-scopes) |
| Scopes to avoid | `read_customers`, `read_companies`, `read_customer_*`. Write scopes also grant read, so request them only when a write feature ships. | same |

**Unverified, to check during Milestone 3:**
- Whether product image alt text and dimensions are fully readable with `read_products` alone, or whether `read_files` is also needed.
- Whether menu (navigation) structure is readable for internal-linking checks and with which scope.
- Current rules for merchants creating custom apps / admin API tokens for a standalone CLI (Shopify has been changing how custom apps are created). The CLI must use whatever method is currently documented.
- Node.js version required by the current Shopify CLI and template.

### 1.2 SEO standards

| Topic | Finding | Source |
| --- | --- | --- |
| Titles | Google sets **no length limit** for `<title>`; it truncates for display. Titles should be unique, descriptive, concise, not keyword-stuffed, not boilerplate. | [Title links](https://developers.google.com/search/docs/appearance/title-link) |
| Meta descriptions | Google sets **no length limit**; it truncates for display and may not use the description at all. Descriptions should be unique, accurate, specific, not keyword lists. | [Snippets](https://developers.google.com/search/docs/appearance/snippet) |
| Product structured data | For merchant listings, `Product` requires `name`, `image`, `offers` (an `Offer`). Recommended: `description`, `brand.name`, `gtin*`, `aggregateRating`, and others. | [Merchant listing structured data](https://developers.google.com/search/docs/appearance/structured-data/merchant-listing) |

**Decision:** because Google publishes no length limits, title and description length checks are **heuristics**, labelled as such in the UI and docs, with configurable thresholds (defaults: title 30–60 characters, meta description 70–160 characters). The tool will never present these as Google rules. The full `Offer` required-property list is still to be confirmed from the same Google page before the structured-data rule is written.

### 1.3 Comparable open-source tools

- [Outpost-Apps/shoplint](https://github.com/Outpost-Apps/shoplint) (MIT): CLI that audits a public storefront URL for speed, SEO, images, structured data and installed apps. No Admin API access, single commit, very early.
- Most other Shopify SEO auditors found are closed-source paid apps on the Shopify App Store.

**Gap this project targets:** catalogue-wide auditing from Admin API data (duplicate titles and descriptions across every product, alt-text coverage across all media, collection content), a transparent and documented scoring model, and suggestion features that never invent facts and never write to a store without approval. Storefront crawling is complementary and will be built too, but it is not the differentiator.

## 2. Architecture (proposed)

A TypeScript monorepo with a strict split between pure logic and I/O:

```
packages/
  core/        Audit rules, issue model, scoring. Pure functions, no network, no framework. Most tests live here.
  storefront/  Technical crawler: robots.txt, sitemap, status codes, redirects, HTML head/headings, JSON-LD. Polite, rate-limited.
  shopify/     Admin GraphQL client: pagination, cost-aware throttling, retries, bulk operations for large catalogues. Maps API data to core types.
  optimiser/   Suggestion generator. Rule/template-based first, using only fields present in product data. Optional LLM provider later, behind an interface, off by default.
  cli/         `shopify-seo-audit` command: audit a fixture file, a storefront URL, or a store via Admin API token; JSON/CSV/HTML export.
apps/
  shopify-app/ Embedded admin app built from Shopify's React Router template. Read-only scopes. Dashboard, history, exports.
```

Key rules:
- **Read-only first.** Initial scopes: `read_products`, `read_content` (plus `read_files` only if verification shows it is needed). No customer scopes.
- **No automatic writes.** Any write feature (applying a suggested title etc.) is a later milestone, needs a write scope added deliberately, a preview/diff, explicit per-change approval, and stores the previous value so it can be reverted.
- **Secrets** only in server-side environment variables; `.env.example` documents names; nothing in frontend bundles or the repo. Secret scanning in CI.
- **Testing target:** mock data and fixtures, plus a free Shopify development store. Never the owner's live CUAVAN store without written permission.

## 3. Scoring model (proposed)

- Each rule has a fixed severity: Critical, High, Medium, Low, with documented point weights (proposed 10 / 5 / 2 / 1).
- Category score (Products, Collections, Technical, Images) = 100 × (1 − penalty ÷ maximum possible penalty for the items audited in that category), so stores of different sizes are comparable.
- Overall score = weighted mean of category scores, weights documented in `docs/SCORING.md`.
- Every score in the UI links to the exact rules and counts that produced it.
- The UI and docs state plainly that a score does not predict rankings or sales.

## 4. Milestones and acceptance criteria

| # | Milestone | Acceptance criteria |
| --- | --- | --- |
| 1 | Research and plan | This document, with sources. ✅ |
| 2 | Repository foundation | MIT licence, README (honest status), CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, CHANGELOG, issue/PR templates, `.gitignore`, `.env.example`, Dependabot, CI running install, lint, typecheck, test, build with actions pinned to commit SHAs. CI green on `main`. |
| 3 | Core engine | Issue model, all product/collection/image rules in §3 of the brief, scoring. ≥90% line coverage on `core`. Every rule has passing and failing fixture tests. `docs/RULES.md` and `docs/SCORING.md` generated from or checked against the rule definitions. |
| 4 | CLI + fixture audits | `shopify-seo-audit audit --fixture <file>` produces JSON, CSV and HTML reports. Commands in README verified in CI. |
| 5 | Storefront crawler | robots.txt, sitemap, status/redirect chains, head tags, headings, canonicals, JSON-LD checks against a local test server. Respects robots.txt, concurrency and delay limits. Clearly separates automatic checks from manual-review items. Never claims Google index status. |
| 6 | Shopify Admin integration | Paginated and bulk reads, cost-aware throttling, retry with backoff, expired/revoked token handling. Integration tests against mocked GraphQL responses (including throttled and error responses). Verified once against a development store, with the run documented. |
| 7 | Embedded app + dashboard | React Router app from the official template, read-only scopes, dashboard (scores, counts, critical/high lists, per-product/collection pages, history, export). Accessibility checks (axe) and Playwright end-to-end tests of the main journeys against mocked data. |
| 8 | Optimiser | Suggestions for titles, meta descriptions, headings and description structure built only from existing product fields. Tests proving no attribute appears in output that is absent from input. No write-back in this milestone. |
| 9 | Security review | Dependency audit, licence audit, secret scan, review of auth/session handling, CSRF/XSS/redirect checks, log review. Findings and residual risks written to `docs/SECURITY-REVIEW.md`. |
| 10 | Release readiness | Release gate in the brief verified item by item with links to evidence. Tagged `v0.1.0` only if the gate passes. |

## 5. Known blockers

- **Package installs:** the current working environment cannot reach the npm registry, so Milestones 2+ (which need TypeScript tooling, test runners and Shopify packages) require network access for package installs to be enabled.
- **Development store:** Milestone 6 end-to-end verification needs a free Shopify Partner account and development store created by the owner.
- **Claude for Open Source Program:** see `docs/CLAUDE-OSS-ELIGIBILITY.md`.
