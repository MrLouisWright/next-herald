<div align="center">

# next-herald

**Typed metadata, safe JSON-LD, sitemaps with hreflang, OG templates and an SEO CI check for the Next.js 16 App Router. Built on native Next APIs, zero dependencies beyond `next` and `react`.**

![status: pre-alpha](https://img.shields.io/badge/status-pre--alpha-red)
![license: MIT](https://img.shields.io/badge/license-MIT-blue)
![Next.js 16](https://img.shields.io/badge/Next.js-16-black)

</div>

> [!WARNING]
> **next-herald is pre-alpha and not published to npm.** Everything below describes the **planned** API. No library code exists yet; the design is open for feedback in [issues](https://github.com/MrLouisWright/next-herald/issues) and the [ADRs](docs/adr/README.md).

## Why

The App Router moved SEO into Next itself (`generateMetadata`, `sitemap.ts`, `robots.ts`, `opengraph-image`). The popular packages come from the Pages Router era:

- **next-sitemap** (~639K downloads/week) writes files to `public/` in a postbuild step. Its last release was 4.2.3 in September 2023, and on Next 16 + Vercel those files are not deployed ([#1048](https://github.com/iamvishnusankar/next-sitemap/issues/1048)).
- **next-seo** (~473K/week) is maintained (7.3.0, July 2026). In the App Router it now recommends native `generateMetadata` for meta tags and focuses on JSON-LD components.
- **Native Next.js** covers the basics but has no sitemap index, no news sitemaps, shallow metadata merging, a JSON-LD [guide](https://nextjs.org/docs/app/guides/json-ld) that warns `JSON.stringify` is not XSS-safe, and no way to check a build. Metadata bugs keep appearing (e.g. [vercel/next.js#98684](https://github.com/vercel/next.js/issues/98684), [#97472](https://github.com/vercel/next.js/issues/97472), [#98126](https://github.com/vercel/next.js/issues/98126)).

Details and sources: [Research](docs/research.md).

## Positioning

> next-herald is a Next.js 16 App Router–only SEO kit that returns **native** `Metadata`, `MetadataRoute.Sitemap`/`Robots` and `ImageResponse` objects, fills the gaps Next leaves (sitemap index, news sitemaps, deep merging, hreflang, XSS-safe JSON-LD), and ships `herald check`, a dependency-free CI crawler for technical SEO mistakes. It is pre-alpha. It makes no ranking or performance claims; bundle sizes will be published with reproducible scripts.

## Design principles

1. **Native first.** Return objects Next renders; never render our own `<head>` tags. ([ADR 0001](docs/adr/0001-build-on-native-metadata-apis.md))
2. **Zero dependencies.** Only the `next` and `react` peers; the CLI uses Node built-ins. Enforced in CI. ([ADR 0002](docs/adr/0002-zero-dependencies.md))
3. **Static by default.** Never read headers or cookies, so pages stay in the Cache Components static shell. ([ADR 0006](docs/adr/0006-cache-components-safety.md))
4. **Safe by default.** JSON-LD is escaped; previews are `noindex` unless built for production. ([ADR 0004](docs/adr/0004-safe-json-ld-serialization.md))
5. **Check what crawlers see.** `herald check` reads served HTML, not Next internals. ([ADR 0008](docs/adr/0008-check-crawls-http.md))

**Non-goals (v1):** Pages Router, Next.js ≤ 15, rendering `<head>` tags, headless-browser crawling, content/keyword scoring, a hosted service.

## Features (planned)

- **Typed metadata builders** — `createMetadata` (site defaults, title template, `metadataBase`), `pageMetadata` (canonical, OG/Twitter filled from title/description), `mergeMetadata` (deep merge, since Next merges per top-level key). ([Metadata](docs/metadata.md))
- **Typed JSON-LD** — builders for common Google rich-result types with required fields enforced by TypeScript, and a `<JsonLd>` Server Component with XSS-safe serialization; accepts schema-dts values. ([JSON-LD](docs/json-ld.md))
- **Sitemaps from your `app/` routes** — `herald routes` scans `app/`, `defineSitemap` requires a typed `params` enumerator for every dynamic route; image, video and language alternates. ([Sitemaps](docs/sitemaps.md))
- **Large sites** — split at 50,000 URLs / 50 MB with a generated sitemap index; Google News sitemaps. ([ADR 0007](docs/adr/0007-sitemap-index-via-route-handlers.md))
- **Robots** — `robots.ts` with automatic sitemap lines, preview lock-down and an optional AI-crawler list. ([Robots](docs/robots.md))
- **hreflang** — reciprocal alternates and `x-default` from one `I18nSource`; optional [next-tongue](https://github.com/MrLouisWright/next-tongue) adapter, recipes for next-intl and Paraglide. ([hreflang & i18n](docs/i18n-hreflang.md))
- **OG image templates** — typed JSX templates on `next/og` `ImageResponse`, no bundled fonts. ([OG images](docs/og-images.md))
- **`herald check`** — crawls a running server or a static export for missing/duplicate titles and descriptions, broken canonicals, hreflang reciprocity, `noindex` leaks, sitemap/robots consistency, structured data validity and OG image size; `pretty`, `json`, GitHub annotations and SARIF output. ([CLI & rules](docs/cli.md))
- **Cache Components and static export safe** — pure builders; works with `cacheComponents: true` and `output: 'export'`. ([Cache Components](docs/cache-components.md), [Static export](docs/static-export.md))

## Quick start (planned API)

```bash
npm install next-herald     # not yet published
npx herald init
```

```ts
// herald.config.ts
import { defineHerald } from 'next-herald/config';

export default defineHerald({
  siteUrl: 'https://example.com',
  siteName: 'Example',
  title: { default: 'Example', template: '%s · Example' },
  description: 'What Example does, in one sentence.',
});
```

```tsx
// app/layout.tsx (excerpt)
import { createMetadata } from 'next-herald/metadata';
import herald from '@/herald.config';
export const metadata = createMetadata(herald);
```

```tsx
// app/blog/[slug]/page.tsx
import { pageMetadata } from 'next-herald/metadata';
import { JsonLd, jsonLd } from 'next-herald/jsonld';
import herald from '@/herald.config';
import { getPost } from '@/lib/posts';

export async function generateMetadata({ params }: PageProps<'/blog/[slug]'>) {
  const { slug } = await params;
  const post = await getPost(slug);
  return pageMetadata(herald, { path: `/blog/${slug}`, title: post.title, description: post.excerpt, type: 'article' });
}

export default async function Page({ params }: PageProps<'/blog/[slug]'>) {
  const { slug } = await params;
  const post = await getPost(slug);
  return (
    <article>
      <JsonLd data={jsonLd.article({ headline: post.title, datePublished: post.date, author: { name: post.author } })} />
      <h1>{post.title}</h1>
    </article>
  );
}
```

```ts
// app/sitemap.ts
import { defineSitemap } from 'next-herald/sitemap';
import { routes } from '@/herald-routes.gen'; // written by `herald routes` (v0.2); v0.1: a hand-written list
import herald from '@/herald.config';
import { listSlugs } from '@/lib/posts';

export default defineSitemap(herald, {
  routes,
  params: { '/blog/[slug]': async () => (await listSlugs()).map((slug) => ({ slug })) }, // missing key = type error
});
```

```ts
// app/robots.ts
import { defineRobots } from 'next-herald/robots';
import herald from '@/herald.config';
export default defineRobots(herald, { rules: [{ userAgent: '*', disallow: ['/admin'] }] });
```

```bash
npx herald check --build --format github   # CI
```

Full walkthrough: [Getting started](docs/getting-started.md).

## Comparison

Feature-level comparison as of October 2026. next-herald's column is **planned**, not shipped. Weekly downloads from api.npmjs.org (2026-10-06 → 10-12).

| | next-herald (planned) | Native Next.js 16 | next-seo 7.3 | next-sitemap 4.2.3 | schema-dts 2.1 |
|---|---|---|---|---|---|
| Status | pre-alpha | stable | active (Jul 2026) | last release Sep 2023 | active (Oct 2026) |
| Downloads/week | — | — | 472,622 | 638,748 | 3,138,833 |
| Runtime dependencies | 0 (peers `next`, `react`) | — | 0 | 4 | 1 (`schema-dts-lib`) |
| Meta tags | builds native `Metadata` | `metadata` / `generateMetadata` | recommends native in App Router; `<NextSeo>` for Pages | — | — |
| Deep merge across segments | yes | no (per top-level key) | — | — | — |
| JSON-LD | typed builders + XSS-safe `<JsonLd>` | guide only | components | — | types only |
| Sitemap | native `sitemap.ts` from `app/` routes | `sitemap.ts`, `generateSitemaps` | — | postbuild files in `public/` | — |
| Sitemap index > 50k | yes (route handler) | split only, no index | — | yes | — |
| News sitemap | yes | no | — | no | — |
| hreflang alternates | from one `I18nSource` | manual | `languageAlternates` (Pages) | `alternateRefs` | — |
| OG image templates | yes (`next/og`) | `ImageResponse` | — | — | — |
| CI checker | `herald check` | — | — | — | — |
| Pages Router | no | — | yes | yes | n/a |

Sources in [Research](docs/research.md). Corrections welcome — open an issue if any cell is wrong.

## Documentation

Also available as a [GitHub wiki](https://github.com/MrLouisWright/next-herald/wiki) (same content as `docs/`).

| Guides | Reference |
|---|---|
| [Getting started](docs/getting-started.md) | [API reference](docs/api-reference.md) |
| [Configuration](docs/configuration.md) | [CLI & check rules](docs/cli.md) |
| [Metadata](docs/metadata.md) | [Architecture](docs/architecture.md) |
| [JSON-LD](docs/json-ld.md) | [Architecture Decision Records](docs/adr/README.md) |
| [Sitemaps](docs/sitemaps.md) | [FAQ](docs/faq.md) |
| [Robots](docs/robots.md) | [Troubleshooting](docs/troubleshooting.md) |
| [hreflang & i18n](docs/i18n-hreflang.md) | [Testing](docs/testing.md) |
| [OG images](docs/og-images.md) | [Security](docs/security.md) |
| [Cache Components & PPR](docs/cache-components.md) | [Research (Oct 2026)](docs/research.md) |
| [Static export](docs/static-export.md) | [Critical review](docs/review.md) |
| [Migration from next-seo](docs/migration/next-seo.md) | [Docs index](docs/README.md) |
| [Migration from next-sitemap](docs/migration/next-sitemap.md) | |

## Roadmap

| Milestone | Theme |
|---|---|
| v0.1 | **Core builders, ship fast:** config, metadata builders, `<JsonLd>` + 7 types, basic sitemap/robots, CI with dependency budget |
| v0.2 | `herald check` core (titles, descriptions, canonicals, noindex, sitemap/robots) + `herald routes` |
| v0.3 | Large sites & i18n: sitemap index, news, hreflang, next-tongue adapter, static export |
| v0.4 | OG templates, JSON-LD and OG check rules, more schema types, SARIF |
| v0.5 | `herald init`, verified migration guides, published size/perf report |
| v0.9 | Beta: API and rule-ID freeze |
| v1.0 | Stable, semver guarantees |
| post-1.0 | next-seo / next-sitemap codemods |

**Scope freeze:** no new features before v0.1 ships.

Full detail with acceptance criteria: [ROADMAP.md](ROADMAP.md) · [Milestones](https://github.com/MrLouisWright/next-herald/milestones) · [Project board](https://github.com/users/MrLouisWright/projects/8).

## Contributing

Design feedback is the most valuable contribution right now. Read [CONTRIBUTING.md](CONTRIBUTING.md) or comment on an [ADR](docs/adr/README.md). Please follow the [Code of Conduct](CODE_OF_CONDUCT.md); report vulnerabilities per [SECURITY.md](SECURITY.md).

## License

[MIT](LICENSE) © 2026 Louis Wright
