# Cache Components & PPR

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

With [`cacheComponents: true`](https://nextjs.org/docs/app/getting-started/cache-components), anything that reads request data (`headers()`, `cookies()`, `connection()`, `searchParams`) must sit in a dynamic boundary. next-herald never reads request data ([ADR 0006](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0006-cache-components-safety.md)), so metadata, JSON-LD, sitemaps and OG images stay prerenderable.

## Rules we follow
- `siteUrl` comes from config/env at build, never `Host`.
- All builders are pure; they can run inside `"use cache"`.
- A CI test greps the published code for request-API imports ([#7](https://github.com/MrLouisWright/next-herald/issues/7)), and an e2e fixture with `cacheComponents: true` asserts every example route is in the static shell.

## Patterns
```tsx
// generateMetadata with cached data
import { cacheLife, cacheTag } from 'next/cache';
async function getPost(slug: string) {
  'use cache';
  cacheLife('hours');
  cacheTag(`post:${slug}`);
  return db.post(slug);
}
export async function generateMetadata({ params }: PageProps<'/blog/[slug]'>) {
  const { slug } = await params;            // params are fine; with generateStaticParams the page stays static
  return pageMetadata(herald, { path: `/blog/${slug}`, title: (await getPost(slug)).title });
}
```
- Don't put the CSP `nonce` (a request value) into cached JSON-LD; nonce-based CSP makes the page dynamic by design.
- Bots: Next streams metadata to browsers but blocks for [`htmlLimitedBots`](https://nextjs.org/docs/app/api-reference/config/next-config-js/htmlLimitedBots). `herald check` sends a crawler UA by default (`--ua browser` to compare). Custom patterns have caused metadata loss under PPR ([vercel/next.js#95406](https://github.com/vercel/next.js/issues/95406), [#96594](https://github.com/vercel/next.js/issues/96594)).
- Client navigation title loss when a prefetch is in flight ([#98684](https://github.com/vercel/next.js/issues/98684)) is a Next router bug outside our control.
