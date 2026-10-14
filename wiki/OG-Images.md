# OG images

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

Templates are JSX for [`ImageResponse`](https://nextjs.org/docs/app/api-reference/functions/image-response) from `next/og` ([ADR 0011](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0011-og-templates-on-next-og.md)), used in [`opengraph-image.tsx`](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/opengraph-image).

```tsx
// app/blog/[slug]/opengraph-image.tsx
import { ogImage, ArticleCard, ogSize } from 'next-herald/og';
import { getPost } from '@/lib/posts';

export const size = ogSize;            // { width: 1200, height: 630 }
export const contentType = 'image/png';
export const alt = 'Blog post cover';

export default async function Image({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  const post = await getPost(slug);
  return ogImage(ArticleCard, { title: post.title, eyebrow: 'Blog', footer: 'example.com', theme: 'dark' });
}
```

- Templates: `SimpleCard`, `ArticleCard`, `ProductCard` (v0.4). Props are typed; long titles are clamped heuristically (Satori has no line clamp).
- **Fonts are not bundled.** Pass `fonts: [{ name, data, weight }]` loaded with `readFile` in Node or `fetch` at build. CJK, Arabic and Hebrew need a font with those glyphs.
- Only the Satori CSS subset is used (flexbox; no grid). Custom templates are plain functions returning JSX.
- Under Cache Components, OG images that depend only on `params` are prerendered when `generateStaticParams` covers them.
- Static export: images must be fully static (no request data).

Checks: `og-image-missing` (warn), `og-image-status` (error), `og-image-dimensions` (warn: not ≥ 1200×630 or not ~1.91:1), `og-image-bytes` (warn: > 5 MB, configurable), `og-image-type` (warn: not PNG/JPEG/WebP/GIF), `og-image-alt` (info).
