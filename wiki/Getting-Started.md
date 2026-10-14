# Getting started

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

Requirements: Next.js 16 App Router, React 19.2+, Node 20.9+ (Node ≥ 22.18 for a TypeScript CLI config, see [Configuration](Configuration)).

## 1. Install
```bash
npm install next-herald   # not yet published
npx herald init           # writes herald.config.ts, app/sitemap.ts, app/robots.ts
```

## 2. Configure the site once
```ts
// herald.config.ts
import { defineHerald } from 'next-herald/config';

export default defineHerald({
  siteUrl: 'https://example.com',          // never derived from request headers (ADR 0006)
  siteName: 'Example',
  title: { default: 'Example', template: '%s · Example' },
  description: 'What Example does, in one sentence.',
  twitter: { site: '@example' },
});
```

## 3. Root metadata
```tsx
// app/layout.tsx
import { createMetadata } from 'next-herald/metadata';
import herald from '../herald.config';

export const metadata = createMetadata(herald); // sets metadataBase, title template, openGraph, twitter
```

## 4. Per-page metadata
```tsx
// app/blog/[slug]/page.tsx
import { pageMetadata } from 'next-herald/metadata';
import herald from '@/herald.config';
import { getPost } from '@/lib/posts';

export async function generateMetadata({ params }: PageProps<'/blog/[slug]'>) {
  const { slug } = await params;
  const post = await getPost(slug);
  return pageMetadata(herald, {
    path: `/blog/${slug}`,      // becomes the canonical URL
    title: post.title,
    description: post.excerpt,
    type: 'article',
    publishedTime: post.date,
  });
}
```

## 5. Structured data
```tsx
import { JsonLd, jsonLd } from 'next-herald/jsonld';

<JsonLd data={jsonLd.article({ headline: post.title, datePublished: post.date, author: { name: post.author }, image: [post.cover] })} />
```

## 6. Sitemap and robots
```ts
// app/sitemap.ts
import { defineSitemap } from 'next-herald/sitemap';
import { routes } from '@/herald-routes.gen';   // from `herald routes` (v0.2); in v0.1 pass a hand-written list
import herald from '@/herald.config';
import { listSlugs } from '@/lib/posts';

export default defineSitemap(herald, {
  routes,
  params: { '/blog/[slug]': async () => (await listSlugs()).map((slug) => ({ slug })) },
  exclude: ['/admin/**'],
});
```
```ts
// app/robots.ts
import { defineRobots } from 'next-herald/robots';
import herald from '@/herald.config';
export default defineRobots(herald, { disallow: ['/admin'] }); // adds the sitemap URL
```

## 7. Check in CI
```bash
npx herald check --build        # next build, next start on a free port, crawl, report
```
See [CLI](CLI-and-Check-Rules) for rule IDs. Next: [Configuration](Configuration).
