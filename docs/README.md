# next-herald documentation

> **Status: planned / pre-release.** This page documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](../ROADMAP.md).

## Guides
1. [Getting started](getting-started.md)
2. [Configuration](configuration.md)
3. [Metadata](metadata.md)
4. [JSON-LD structured data](json-ld.md)
5. [Sitemaps](sitemaps.md)
6. [Robots](robots.md)
7. [hreflang & i18n (incl. next-tongue)](i18n-hreflang.md)
8. [OG images](og-images.md)
9. [CLI and `herald check` rules](cli.md)
10. [Cache Components & PPR](cache-components.md)
11. [Static export](static-export.md)
12. [Testing](testing.md)
13. Migration: [from next-seo](migration/next-seo.md) · [from next-sitemap](migration/next-sitemap.md)

## Reference
- [API reference](api-reference.md)
- [Architecture](architecture.md)
- [Architecture Decision Records](adr/README.md)
- [FAQ](faq.md)
- [Troubleshooting](troubleshooting.md)
- [Security](security.md)
- [Research: competitors and Next.js metadata issues (Oct 2026)](research.md)
- [Critical review (Oct 2026)](review.md)

## Package entry points (planned)

| Import | Runs in | Purpose |
|---|---|---|
| `next-herald/config` | anywhere | `defineHerald` |
| `next-herald/metadata` | Server (layouts, pages, `generateMetadata`) | `createMetadata`, `mergeMetadata`, `pageMetadata` |
| `next-herald/jsonld` | Server Components | `JsonLd`, `serializeJsonLd`, `jsonLd.*` builders, schema types |
| `next-herald/sitemap` | `sitemap.ts`, route handlers | `defineSitemap`, `splitSitemap`, `sitemapIndex`, `newsSitemap` |
| `next-herald/robots` | `robots.ts` | `defineRobots` |
| `next-herald/i18n` | Server | `alternates`, `fromTongue` |
| `next-herald/og` | `opengraph-image.tsx` | `ogImage`, templates |
| `next-herald/testing` | tests | `checkHtml`, `renderMetadata` |
| `herald` (bin) | Node built-ins only | `init`, `routes`, `build`, `check` |

The package has **zero dependencies** beyond the `next` and `react` peers ([ADR 0002](adr/0002-zero-dependencies.md)).
