# 0001. Build on native Next.js metadata APIs

- Status: Accepted
- Date: 2026-10-14

## Context
Pages-Router-era tools (`next-seo`'s `<NextSeo>`, `next-sitemap`'s postbuild script) render tags or write files outside Next's own pipeline. In the App Router, Next already owns `<head>`: `metadata`/`generateMetadata`, `sitemap.ts`, `robots.ts`, `opengraph-image.tsx` and `ImageResponse`. Tags rendered outside that pipeline can duplicate or fight Next's output, and files written to `public/` after the build are not part of the build output ([iamvishnusankar/next-sitemap#1048](https://github.com/iamvishnusankar/next-sitemap/issues/1048)).

## Decision
- next-herald returns **plain native objects**: `Metadata`, `MetadataRoute.Sitemap`, `MetadataRoute.Robots`, and `ImageResponse`. The user exports them from the native file conventions.
- next-herald never renders `<title>`, `<meta>` or `<link>` tags itself. The only element it renders is the JSON-LD `<script>` ([ADR 0004](0004-safe-json-ld-serialization.md)), which Next's [JSON-LD guide](https://nextjs.org/docs/app/guides/json-ld) also leaves to the app.
- Things Next has no API for (a sitemap index, `news:` sitemap extension) are emitted from **route handlers** that the user creates, never by writing to `public/`.

## Consequences
- \+ Streaming metadata, `htmlLimitedBots`, deduplication and future Next fixes apply automatically.
- \+ Users can drop next-herald at any time and keep the plain objects.
- − Next metadata bugs are inherited (e.g. [vercel/next.js#98684](https://github.com/vercel/next.js/issues/98684), [#97472](https://github.com/vercel/next.js/issues/97472)). We document them and test against canary; we do not patch around them.

## Alternatives considered
A `<Seo>` head component (next-seo style): rejected, it conflicts with Next's head management. A postbuild generator (next-sitemap style): rejected, see above.
