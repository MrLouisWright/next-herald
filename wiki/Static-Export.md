# Static export

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

With [`output: 'export'`](https://nextjs.org/docs/app/guides/static-exports), metadata, `sitemap.ts`, `robots.ts` and `opengraph-image` work if they are fully static.

- `defineSitemap` / `defineRobots`: supported. Dynamic patterns need `params` enumerators that run at build.
- Sitemap index via route handlers (`sitemapIndex`, `newsSitemap`): route handlers in export must be static `GET`s with `export const dynamic = 'force-static'`; covered by an e2e fixture (v0.3, [#28](https://github.com/MrLouisWright/next-herald/issues/28)). If it cannot be made reliable, the docs will say so and `herald build --emit-xml out/` is the fallback (written after `next build`, into the export output, not `public/`).
- OG images: dynamic routes need `generateStaticParams`.
- `indexing: 'auto'` is evaluated at build; set `HERALD_ENV=production` in the production build.
- Check it: `herald check --dir out` (no server). Clean URLs depend on your host; pass `--trailing-slash` to match `next.config`.
