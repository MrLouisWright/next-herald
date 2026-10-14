# 0005. Own curated schema.org types, structurally compatible with schema-dts

- Status: Accepted
- Date: 2026-10-14

## Context
[schema-dts](https://github.com/google/schema-dts) (~3.1M downloads/week) types the whole schema.org vocabulary; it is excellent but large and now depends on `schema-dts-lib`. ADR 0002 forbids dependencies. Most sites use a handful of types that Google turns into rich results.

## Decision
- Ship hand-written types and builders for a **curated set**: `Organization`, `WebSite` (+ `SearchAction`), `WebPage`, `BreadcrumbList`, `Article`/`NewsArticle`/`BlogPosting`, `Product` + `Offer` + `AggregateRating` + `Review`, `FAQPage`, `Person`, `LocalBusiness`, `Event`, `VideoObject`, `SoftwareApplication`, `Recipe`, `JobPosting` (phased per [ROADMAP](../../ROADMAP.md)).
- Builders (`jsonLd.article({...})`) mark Google-required properties as required in TypeScript; recommended ones are optional and reported by `herald check` (`jsonld-recommended`).
- `<JsonLd>` accepts any object with `@type` (escape hatch), so schema-dts `WithContext<T>` values type-check without importing schema-dts.

## Consequences
\+ No dependency, readable types, required-field guidance. − Vocabulary coverage is partial; schema.org changes need manual updates (checked quarterly).

## Alternatives considered
Generating types for the full vocabulary at build time: rejected for v1 (size, editor performance, maintenance). schema-dts as optional peer: rejected (ADR 0002).
