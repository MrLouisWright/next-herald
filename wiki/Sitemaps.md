# Sitemaps

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

next-herald returns native `MetadataRoute.Sitemap` arrays for [`sitemap.ts`](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/sitemap) and XML `Response`s only where Next has no API ([ADR 0007](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0007-sitemap-index-via-route-handlers.md)).

## From the route manifest
`herald routes` scans `app/` and writes `herald-routes.gen.ts` ([ADR 0009](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0009-cli-config-and-route-scan.md)):
```ts
export const routes = ['/', '/about', '/blog', '/blog/[slug]', '/docs/[...slug]'] as const;
```
Route groups `(marketing)` are removed, `@slot`, intercepting `(.)` routes, `_private` folders, route handlers and metadata files are skipped. Run it in `prebuild` (`herald build` does) and commit or git-ignore the file.

`defineSitemap(config, { routes, params, exclude, entry })`:
- static routes are included automatically;
- every dynamic pattern needs a `params` enumerator — a **type error** otherwise — or must be listed in `exclude`;
- `entry(path, params)` may return `lastModified`, `changeFrequency`, `priority`, `images`, `videos`, `alternates`, or `null` to skip;
- URLs are absolute (from `siteUrl`), deduplicated and sorted; `trailingSlash` is applied;
- pages with `noindex` must be excluded by you; `herald check` reports `sitemap-noindex` if one slips in.

## Accurate `lastModified` (v0.3)
Google uses `lastmod` only if it is "consistently and verifiably accurate" ([Google](https://developers.google.com/search/docs/crawling-indexing/sitemaps/build-sitemap)), so next-herald never stamps build time ([#50](https://github.com/MrLouisWright/next-herald/issues/50)). Sources, best first:
1. Your content's `updatedAt` from `entry()`; use the same value for JSON-LD `dateModified` and `article:modified_time`.
2. `herald routes --git-lastmod`: the route file's last commit date (`git log -1 --format=%cI`), written to `herald-routes.gen.ts` at build time. Good for static pages; needs full history (not a shallow clone) and is wrong for data-driven pages.
3. Omit `lastModified`.

`herald check` reports `lastmod-future`, `lastmod-uniform` (≥ 90% identical: build-time stamping) and `lastmod-mismatch` (sitemap vs JSON-LD/OG by more than a day).

## Images and videos
Native: each entry accepts `images: string[]` and `videos: [...]` (Next renders `image:` / `video:` namespaces). Helpers `sitemapImage()` / `sitemapVideo()` validate required video fields (`title`, `thumbnail_loc`, `description`, content or player URL) per [Google](https://developers.google.com/search/docs/crawling-indexing/sitemaps/video-sitemaps).

## Language alternates
`alternates: { languages: { de: '…', 'x-default': '…' } }` per entry, generated for you when `i18n` is configured. See [hreflang](hreflang-and-i18n).

## Large sites: index splitting (v0.3)
A sitemap may hold at most 50,000 URLs and 50 MB uncompressed ([Google](https://developers.google.com/search/docs/crawling-indexing/sitemaps/large-sitemaps)). Next's `generateSitemaps` splits but writes **no index**.
```ts
// app/sitemaps/sitemap.ts  → /sitemaps/sitemap/0.xml, /1.xml, …
import { splitSitemap } from 'next-herald/sitemap';
import { source } from '@/lib/sitemap-source';
const s = splitSitemap(source, { pageSize: 50_000 });
export const generateSitemaps = s.generateSitemaps;
export default s.sitemap;   // ({ id }) => entries; id is a Promise in Next 16
```
```ts
// app/sitemap.xml/route.ts  → /sitemap.xml (index)
import { sitemapIndex } from 'next-herald/sitemap';
import { source } from '@/lib/sitemap-source';
export const GET = sitemapIndex(source, { path: '/sitemaps/sitemap' });
```
Do **not** keep `app/sitemap.ts` in this mode ([vercel/next.js#87314](https://github.com/vercel/next.js/issues/87314)). Avoid `generateSitemaps` under a dynamic segment until [#98126](https://github.com/vercel/next.js/issues/98126) is fixed. `source` exposes `count()` and `page(n, size)` so chunks never load all URLs.

## News sitemap (v0.3)
```ts
// app/news-sitemap.xml/route.ts
import { newsSitemap } from 'next-herald/sitemap';
export const GET = newsSitemap(herald, { publication: { name: 'Example Times', language: 'en' }, articles: getRecentArticles });
```
Per [Google News sitemaps](https://developers.google.com/search/docs/crawling-indexing/sitemaps/news-sitemap): articles from the last 2 days only, max 1,000 URLs; older items are dropped with a dev warning.

## Caching
`sitemap.ts` is a route handler. With Cache Components, wrap data access in `"use cache"` + `cacheLife`. Note upstream: `revalidate` in `sitemap.ts` may be emitted as static in adapter outputs ([#99055](https://github.com/vercel/next.js/issues/99055)); metadata routes send `max-age=0` ([#99715](https://github.com/vercel/next.js/issues/99715)).
