# 0003. Next.js 16+ App Router only; supported versions and canary CI

- Status: Accepted
- Date: 2026-10-14

## Context
Metadata behaviour changed across 13–16 (streaming metadata, `htmlLimitedBots`, async `params`, `generateSitemaps` `id` as a promise, Cache Components). Supporting the Pages Router means a second head mechanism.

## Decision
- Peer range `next@>=16.0.0 <17`, `react@>=19`. App Router only; no Pages Router entry point.
- CI: unit and e2e on the **latest Next 16 minor** and the oldest supported minor; a **nightly `next@canary` job** opens an issue on failure ([#4](https://github.com/MrLouisWright/next-herald/issues/4)).
- Known Next regressions that affect us are tracked with a `upstream` label and documented in [Troubleshooting](../troubleshooting.md).
- Node: `>=20.9` (Next 16's minimum) for runtime; the CLI's TypeScript config needs Node with type stripping, see [ADR 0009](0009-cli-config-and-route-scan.md).

## Consequences
\+ One mental model, small surface. − Projects on Next ≤ 15 or Pages Router should keep next-seo / next-sitemap (see [migration](../migration/next-seo.md)).

## Alternatives considered
Supporting Next 15: rejected (async API and Cache Components differences double the test matrix).
