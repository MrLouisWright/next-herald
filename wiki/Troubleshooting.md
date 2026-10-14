# Troubleshooting

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

| Symptom | Cause | Fix |
|---|---|---|
| `herald: siteUrl must be absolute` | Relative or missing `siteUrl` | Set `HERALD_SITE_URL` for the build or a literal origin |
| Pages become dynamic after adding metadata | Your `generateMetadata` reads `headers()`/`cookies()` or uncached data | Move data into `"use cache"`; next-herald itself never does this ([Cache Components](Cache-Components-and-PPR)) |
| Type error: `Property '/blog/[slug]' is missing in params` | New dynamic route without enumerator | Add `params['/blog/[slug]']` or `exclude` it |
| `herald-routes.gen.ts` stale | Routes added after last scan | `herald build` in `prebuild`; `herald routes --watch` in dev |
| `Duplicate page detected… /sitemap.xml` | Both `app/sitemap.ts` and `app/sitemap.xml/route.ts` | Use one ([vercel/next.js#87314](https://github.com/vercel/next.js/issues/87314)) |
| Per-param sitemaps collapsed to `/-/sitemap.xml` (16.3) | Upstream regression [vercel/next.js#98126](https://github.com/vercel/next.js/issues/98126) | Put `generateSitemaps` outside dynamic segments |
| Sitemap never refreshes on Vercel | [vercel/next.js#99055](https://github.com/vercel/next.js/issues/99055) | Redeploy or use a route handler with explicit caching |
| `<title>` empty after a fast click | Prefetch race [vercel/next.js#98684](https://github.com/vercel/next.js/issues/98684) | Upstream; server HTML is unaffected |
| Previous route's OG tags remain after navigation | [vercel/next.js#97472](https://github.com/vercel/next.js/issues/97472) | Upstream; affects client DOM only |
| Metadata missing for a custom bot | Custom `htmlLimitedBots` + PPR ([#95406](https://github.com/vercel/next.js/issues/95406)) | Use Next's default list or test with `herald check --ua` |
| `og:image` shows tofu boxes | Font lacks glyphs | Pass a font with the script's glyphs ([OG images](OG-Images)) |
| `canonical-broken` on every page | `trailingSlash` mismatch with `next.config` | Set the same value in `herald.config.ts` |
| `noindex-leak` in production | `indexing: 'auto'` and `HERALD_ENV` not set at build | Set `HERALD_ENV=production` (or rely on `VERCEL_ENV`) |
| CLI: `Cannot load herald.config.ts` | Node without type stripping | Node ≥ 22.18, or rename to `herald.config.mjs` |
