# Claude for Open Source Program: eligibility check

Checked: 26 November 2026, against the official page [claude.com/contact-sales/claude-for-oss](https://claude.com/contact-sales/claude-for-oss) and its [terms](https://www.anthropic.com/claude-for-oss-terms). Re-check before applying; criteria can change.

## Official criteria (any one qualifies)

| Route | Threshold |
| --- | --- |
| Maintainer / library author | 500+ dependent repos, **or** 100+ dependent packages, **or** 200,000+ combined monthly downloads (npm, PyPI, crates.io, RubyGems etc.) |
| Core contributor | Listed committer/maintainer on a recognised foundation or language project (e.g. CPython, Rust, Node.js TSC, Apache PMC, CNCF, Kubernetes, Linux kernel, Django, Rails) |
| Active contributor | 100+ merged pull requests to repos you don't own in the last 12 months |
| Community builder | One of your repos has 20+ unique external contributors with merged PRs in the last 12 months |
| Critical infrastructure | A repo you maintain has an OpenSSF criticality score ≥ 0.4 |

Stars are explicitly not a criterion. The page also invites projects that don't clearly fit to apply and explain their ecosystem value. The page does not state whether applications are currently open or give a deadline.

## Current position (honest)

| Status | Item |
| --- | --- |
| **Definitely not met today** | All five thresholds. This repository was created on 26 November 2026, has no published package, no dependents, no downloads and no external contributors. |
| **Unverified** | `infocuavanhelp-hub`'s merged PRs to other people's repositories in the last 12 months (could not be checked from this session). The owner can check at `github.com/pulls?q=is:pr+author:infocuavanhelp-hub+is:merged`. |
| **Possible later** | *Community builder* (20+ external contributors) or *Active contributor* (100+ merged PRs elsewhere) are the realistic routes for an individual. Download thresholds require a widely used library, which this app is unlikely to become on its own; the `core` package published to npm is the only plausible path there. |

## Ethical plan to build genuine eligibility

1. Ship a genuinely useful v0.1 with real documentation and labelled `good first issue` tasks that are small and well-specified.
2. Publish `@shopify-seo-audit/core` to npm only when it is stable and useful on its own.
3. Contribute real fixes upstream (Shopify CLI, the app template, SEO libraries the project depends on). Merged PRs to other repos count toward the *Active contributor* route.
4. Respond quickly to issues and review PRs well. That is what brings repeat external contributors.
5. Do **not** create fake accounts, fake PRs, split trivial changes to inflate counts, or buy stars/downloads. That would violate GitHub's terms and the spirit of the programme.
