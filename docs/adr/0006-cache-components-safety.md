# 0006. Cache Components safety: never read request data in library code

- Status: Accepted
- Date: 2026-10-14

## Context
With `cacheComponents: true`, calling `headers()`, `cookies()` or `connection()` makes a scope dynamic and, inside `"use cache"`, is an error. SEO helpers commonly derive the site URL from the `Host` header, which silently turns static pages dynamic.

## Decision
- No next-herald entry point calls `headers()`, `cookies()`, `connection()`, `draftMode()` or reads `searchParams`. A lint test fails the build if these imports appear in `src/` ([#7](https://github.com/MrLouisWright/next-herald/issues/7)).
- The site URL comes from config (`siteUrl`), validated at load: absolute, `https:` (or `http://localhost`), no path query or hash. It also becomes `metadataBase`.
- Builders are pure and synchronous given their inputs, so they are safe inside `"use cache"`, `generateMetadata`, `sitemap.ts` and `opengraph-image.tsx`.
- Per-route data (titles from a CMS) is fetched by the user; docs show `"use cache"` + `cacheTag` patterns.

## Consequences
\+ Pages stay in the static shell; static export works. − Multi-domain sites must pass `siteUrl` per domain explicitly (documented recipe).

## Alternatives considered
Host detection with an opt-in flag: rejected (one flag that breaks prerendering is a footgun).
