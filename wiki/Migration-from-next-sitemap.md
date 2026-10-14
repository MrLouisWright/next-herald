# Migration from next-sitemap

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

[next-sitemap](https://github.com/iamvishnusankar/next-sitemap) runs as a `postbuild` script and writes `sitemap*.xml`/`robots.txt` into `public/`. Its last release is 4.2.3 (September 2023). With Next 16 on Vercel, files written to `public/` after the build are not deployed ([iamvishnusankar/next-sitemap#1048](https://github.com/iamvishnusankar/next-sitemap/issues/1048)).

| next-sitemap.config.js | next-herald |
|---|---|
| `siteUrl` | `siteUrl` in `herald.config.ts` |
| `generateRobotsTxt`, `robotsTxtOptions.policies` | `app/robots.ts` → `defineRobots(herald, { rules })` |
| `robotsTxtOptions.additionalSitemaps` | `defineRobots({ sitemaps: [...] })` |
| `sitemapSize` | `splitSitemap({ pageSize })` + `sitemapIndex` ([Sitemaps](Sitemaps#large-sites-index-splitting-v03)) |
| `exclude` | `defineSitemap({ exclude })` (same glob style: `*`, `**`) |
| `transform(config, path)` | `entry(path, params)` |
| `additionalPaths` | `defineSitemap({ extra: [...] })` |
| `alternateRefs` | `i18n` config |
| `changefreq`, `priority` defaults | `entry` return values (Google ignores both; optional) |
| server-side sitemap (`getServerSideSitemap`) | native `sitemap.ts` with `"use cache"` data |
| `autoLastmod` | Not automatic: pass real `lastModified` from your data (a build timestamp misleads crawlers) |

Steps: remove the `postbuild` script and generated files from `public/` (and `.gitignore`), add `app/sitemap.ts` / `app/robots.ts`, run `herald check --build`, and compare the URL set with the old sitemap (`herald check --format json` lists all sitemap URLs).

next-sitemap discovered pages from `.next` build manifests; next-herald scans `app/` instead and requires `params` for dynamic routes ([ADR 0009](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0009-cli-config-and-route-scan.md)).
