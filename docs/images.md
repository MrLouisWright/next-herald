# Image SEO (v0.4)

> **Status: planned / pre-release.** This page documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](../ROADMAP.md).

Image sitemap entries from `next/image` usage and route data, and rules for alt text, dimensions and formats ([#60](https://github.com/MrLouisWright/next-herald/issues/60)).

## Image sitemap entries
`herald routes --images` scans `app/` at build time for `next/image` `<Image>` elements whose `src` is a string literal or a static import, and writes them to `routes[i].images` in `herald-routes.gen.ts`. Images from data come from `entry()`:

```ts
// app/sitemap.ts
import { defineSitemap } from 'next-herald/sitemap';
import { routes } from '@/herald-routes.gen';
import herald from '@/herald.config';
import { listProducts } from '@/lib/products';

export default defineSitemap(herald, {
  routes,
  params: { '/products/[slug]': async () => (await listProducts()).map((p) => ({ slug: p.slug })) },
  entry: async (path, params) => ({ images: params.slug ? (await listProducts()).find((p) => p.slug === params.slug)?.images : undefined }),
});
```

`defineSitemap` merges both into the native `images` field ([Sitemaps](sitemaps.md#images-and-videos)): absolute, deduped, at most 1,000 per URL, `/_next/image?url=…` resolved to the source image. The scan uses Node built-ins only and runs in the CLI, never in your app.

## Check rules
| ID | Severity | What it reports |
|---|---|---|
| `img-alt-missing` | info (warn for sitemap images) | `<img>` without `alt` in main content (shared with [#51](https://github.com/MrLouisWright/next-herald/issues/51)) |
| `img-dimensions-missing` | warn | `<img>` without `width` and `height` (layout shift; `next/image` sets them unless `fill`) |
| `img-format-legacy` | info | JPEG/PNG/GIF over 100 KB (configurable) served without a WebP/AVIF alternative |
| `sitemap-image-status` | error | Image sitemap URL returns non-200 or a non-image content type |

Dimensions and type are read from file headers (shared with the OG image readers); no image decoding. See [CLI](cli.md#rules).
