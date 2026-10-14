# 0007. Sitemap index and news sitemaps via route handlers

- Status: Accepted
- Date: 2026-10-14

## Context
The [sitemaps protocol](https://www.sitemaps.org/protocol.html) limits a file to 50,000 URLs and 50 MB uncompressed ([Google](https://developers.google.com/search/docs/crawling-indexing/sitemaps/large-sitemaps)). Next's [`generateSitemaps`](https://nextjs.org/docs/app/api-reference/functions/generate-sitemaps) splits a sitemap into `/…/sitemap/[id].xml` but does not emit a sitemap index. `MetadataRoute.Sitemap` supports `images`, `videos` and `alternates.languages`, but not the [news extension](https://developers.google.com/search/docs/crawling-indexing/sitemaps/news-sitemap). Known upstream issues: [vercel/next.js#98126](https://github.com/vercel/next.js/issues/98126) (per-param sitemaps under dynamic segments collapsed, 16.3), [#87314](https://github.com/vercel/next.js/issues/87314) (`sitemap.ts` and `sitemap.xml/route.ts` conflict).

## Decision
- Small sites: `app/sitemap.ts` returns `defineSitemap(...)` output (native).
- Large sites: `app/sitemaps/sitemap.ts` uses `generateSitemaps` from `splitSitemap({ pageSize: 50_000 })` (also enforces 50 MB), and `app/sitemap.xml/route.ts` exports `sitemapIndex(...)`, which returns an XML `Response` listing every chunk. There is no `app/sitemap.ts` in this mode (avoids #87314).
- News: `app/news-sitemap.xml/route.ts` exports `newsSitemap(...)` (only articles from the last 2 days, max 1,000 URLs, per Google).
- `herald check` verifies the index, chunk sizes and that every chunk is reachable.
- Dynamic segments with `generateSitemaps` are avoided in docs until #98126 is fixed.

## Consequences
\+ Works within Next's conventions; no files written to `public/`. − Users create two files for large sites (`herald init --large` scaffolds them).

## Alternatives considered
Writing XML to `public/` post-build: rejected ([ADR 0001](0001-build-on-native-metadata-apis.md)).
