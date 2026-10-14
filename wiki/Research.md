# Research: competitors and Next.js metadata issues (October 2026)

Checked 2026-10-14. Weekly downloads from `https://api.npmjs.org/downloads/point/last-week/<pkg>` for 2026-10-06 → 2026-10-12. Release dates from the npm registry. Every link was opened during this review.

## Download numbers

| Package | Downloads/week | Latest | Notes |
|---|---|---|---|
| [schema-dts](https://www.npmjs.com/package/schema-dts) | 3,138,833 | 2.1.0 (2026-10-02) | Types only; depends on `schema-dts-lib` |
| [@vercel/og](https://www.npmjs.com/package/@vercel/og) | 2,142,440 | — | Same engine as `next/og` |
| [next-sitemap](https://www.npmjs.com/package/next-sitemap) | 638,748 | 4.2.3 (2023-09-06) | 4 runtime deps |
| [next-seo](https://www.npmjs.com/package/next-seo) | 472,622 | 7.3.0 (2026-07-29) | No runtime deps; peers `next >=13.4`, `react >=18.2` |
| [react-schemaorg](https://www.npmjs.com/package/react-schemaorg) | 138,712 | — | JSON-LD component typed with schema-dts |

## Competitors

### next-seo
- Active: 7.1.0 (Jan 2026), 7.2.0 (Feb 2026), 7.3.0 (Jul 2026); ~8.5k stars; 0 open issues at time of check.
- v7 README: "For standard meta tags (`<title>`, `<meta>`), use Next.js's built-in `generateMetadata`". The App Router value is JSON-LD components; Pages Router lives under `next-seo/pages` ([README](https://github.com/garmeeh/next-seo)).
- No sitemap, robots, hreflang checks or CI tool. → next-herald complements rather than replaces it for JSON-LD; migration guide covers both.

### next-sitemap
- Last release 4.2.3 on 2023-09-06; repo last pushed 2026-05; 2 open issues.
- Postbuild script writing to `public/`. With Next 16 on Vercel those files are not deployed: [iamvishnusankar/next-sitemap#1048](https://github.com/iamvishnusankar/next-sitemap/issues/1048). Also [#1050](https://github.com/iamvishnusankar/next-sitemap/issues/1050) (duplicate `Content-Type`).
- Runtime deps: `@corex/deepmerge`, `@next/env@^13.4.3`, `fast-glob`, `minimist`.
- → Strongest gap: a maintained App Router-native sitemap index + robots solution. ADR 0001/0007.

### Native Next.js 16 (latest 16.4.0, canary 16.5.0-canary.10)
Supports `metadata`/`generateMetadata`, `sitemap.ts` with `images`, `videos`, `alternates.languages` and `generateSitemaps` ([docs](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/sitemap), [generateSitemaps](https://nextjs.org/docs/app/api-reference/functions/generate-sitemaps)), `robots.ts`, `opengraph-image`, `ImageResponse`. Gaps: no sitemap index, no news extension, shallow metadata merging, no JSON-LD helper beyond a [guide](https://nextjs.org/docs/app/guides/json-ld) that warns `JSON.stringify` is not XSS-safe, no checker.

Open upstream issues (all open on 2026-10-14):

| Issue | Summary | Impact on next-herald |
|---|---|---|
| [vercel/next.js#98684](https://github.com/vercel/next.js/issues/98684) | Clicking a `<Link>` while its prefetch is in flight commits with no `<title>` (pending segment-cache entry) | Client-side only; documented; not detectable by `herald check` |
| [#97472](https://github.com/vercel/next.js/issues/97472) | Soft navigation leaves previous route's OG/Twitter tags mounted (16.3, Vercel) | Documented |
| [#95836](https://github.com/vercel/next.js/issues/95836) | `not-found.tsx` metadata replaced after hydration | Documented |
| [#83510](https://github.com/vercel/next.js/issues/83510) | Duplicate metadata after server-action revalidation | Documented |
| [#80583](https://github.com/vercel/next.js/issues/80583) | Link prefetch calls `generateMetadata` when streaming | Docs: cache metadata data (`"use cache"`) |
| [#95406](https://github.com/vercel/next.js/issues/95406), [#96594](https://github.com/vercel/next.js/issues/96594) | `htmlLimitedBots` customisation breaks PPR resume / caching and drops metadata | `herald check --ua` compares bot and browser HTML |
| [#98126](https://github.com/vercel/next.js/issues/98126) | Per-param sitemaps under a dynamic segment collapsed (16.3 regression) | ADR 0007 avoids that layout |
| [#87314](https://github.com/vercel/next.js/issues/87314) | `sitemap.ts` and `sitemap.xml/route.ts` conflict | Large-site mode has no `sitemap.ts` |
| [#99055](https://github.com/vercel/next.js/issues/99055) | `revalidate` sitemap emitted as static in adapter outputs | Documented |
| [#99715](https://github.com/vercel/next.js/issues/99715) | Metadata routes send `max-age=0` instead of `s-maxage` | Documented |
| [#98007](https://github.com/vercel/next.js/issues/98007) | Route handlers (incl. sitemaps) not compressed by `next start` | Gzip option for the index handler considered (v0.3) |
| [#54665](https://github.com/vercel/next.js/issues/54665) | `sitemap.ts` with custom `pageExtensions` | `herald routes` reads `pageExtensions` from config |

### schema-dts and others
- [schema-dts](https://github.com/google/schema-dts): complete schema.org types, no runtime, no serializer. We stay type-compatible ([ADR 0005](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0005-own-schema-types.md)).
- [react-schemaorg](https://www.npmjs.com/package/react-schemaorg): small JSON-LD component on schema-dts types.
- Validators: [Rich Results Test](https://search.google.com/test/rich-results), [Schema.org validator](https://validator.schema.org/) — web tools, not CI. Lighthouse SEO audits run a browser per page.

## Positioning derived from this
A Next 16 App Router–only kit that **returns native objects**, fills the native gaps (index, news, deep merge, safe JSON-LD, hreflang), and ships a dependency-free **CI checker**. Not: "faster", "better rankings", or "replaces Next metadata".

## Rejected ideas
| Idea | Why rejected |
|---|---|
| `<Seo>` head component (next-seo style) | Conflicts with Next's head management and streaming metadata (ADR 0001) |
| Postbuild script writing to `public/` | Not deployed on Next 16/Vercel (next-sitemap#1048); outside build output |
| Reading `.next` manifests for routes/check | Internal, unstable (ADR 0008/0009) |
| Deriving `siteUrl` from `Host` | Breaks Cache Components, enables host-header poisoning (ADR 0006) |
| Headless browser crawling (Puppeteer/Playwright) in `herald check` | Heavy dependency, slow; ADR 0002 |
| `schema-dts` dependency / full generated vocabulary | ADR 0002/0005; curated + escape hatch |
| Bundled fonts and hosted OG service | Package size, cost (ADR 0011) |
| AI-generated titles/descriptions | Out of scope; quality and cost concerns; not a technical-SEO check |
| Auto `lastModified` = build time | Misleads crawlers; Google uses lastmod only if consistently accurate |
| Patching Next router bugs (e.g. #98684) client-side | Fragile, fights the framework; report upstream instead |
| Pages Router / Next 15 support | ADR 0003 |
| Keyword density / content scoring rules | Not reliable signals; noise in CI |
