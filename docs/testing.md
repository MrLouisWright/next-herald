# Testing

> **Status: planned / pre-release.** This page documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](../ROADMAP.md).

## Your app
- **Unit**: builders are pure; snapshot `pageMetadata(...)` output or assert fields.
- **Page rules in unit tests**: `checkHtml(html, { url })` from `next-herald/testing` runs page-level rules (titles, canonical, JSON-LD, OG tags) on any HTML string, e.g. from a Playwright test.
- **Site rules in CI**: `herald check --build --format github` (site-wide rules need the whole crawl: duplicates, reciprocity, sitemap consistency).
- Run against previews with `--env preview` (expects `noindex`) and production builds with `--env production`.

## next-herald itself (planned)
- Vitest unit tests; type tests with `expect-type` for builders (required properties, `params` enumerator coverage).
- Corpora: JSON-LD XSS payloads, malformed HTML for the head scanner (plus fuzzing), sitemap/robots edge cases.
- E2E: Playwright + `next build`/`next start` on each example, on the latest and oldest supported Next 16 minor, Turbopack (default) and `--webpack`; nightly `next@canary` ([ADR 0003](adr/0003-nextjs-16-only.md)).
- Dependency budget and request-API guard in CI ([#3](https://github.com/MrLouisWright/next-herald/issues/3), [#7](https://github.com/MrLouisWright/next-herald/issues/7)).
- Every code sample in `docs/` is extracted and type-checked ([#13](https://github.com/MrLouisWright/next-herald/issues/13)).
