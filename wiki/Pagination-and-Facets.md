# Pagination and faceted URLs (v0.3)

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

Helpers for filtered, sorted and paginated listing pages: one normalised URL per indexable state, `noindex, follow` for the rest ([#59](https://github.com/MrLouisWright/next-herald/issues/59)).

```tsx
// app/shop/[category]/page.tsx
import { listingMetadata } from 'next-herald/metadata';
import herald from '@/herald.config';

const listing = { indexable: ['brand'], pageParam: 'page', drop: ['sort', 'utm_*'] } as const;

export async function generateMetadata({ params, searchParams }: PageProps<'/shop/[category]'>) {
  const { category } = await params;
  return listingMetadata(herald, { path: `/shop/${category}`, params: await searchParams, ...listing, title: category });
}
```

| Option | Type | Notes |
|---|---|---|
| `path` | `string` | Listing path without query. |
| `params` | `Record<string, string \| string[] \| undefined>` | Plain object you pass in. |
| `indexable` | `readonly string[]` | Params kept in the canonical; one value each. Multi-value or other combinations get `noindex, follow`. |
| `pageParam` | `string` | Default `'page'`. Page 1 is dropped; page N keeps a self-referencing canonical. |
| `drop` | `readonly string[]` | Params removed from the canonical (globs allowed), e.g. sort order, tracking. |

- Page 2+ is **never** canonicalised to page 1 ([Google](https://developers.google.com/search/docs/specialty/ecommerce/pagination-and-incremental-page-loading)). No `rel=prev/next`: Google does not use it.
- Kept params are ordered stably, so `?b=1&a=2` and `?a=2&b=1` share one canonical.
- `listingHref(path, params, options)` builds the same normalised URLs for your links.

## Cache Components
The library never reads `searchParams` or headers; you pass the object. Reading `searchParams` in a page makes that route dynamic. For static listing pages, put facets in path segments (`app/shop/[category]/[[...facets]]/page.tsx`) and pass parsed segments as `params` ([Cache Components](Cache-Components-and-PPR)).

## Check rules
| ID | Severity | What it reports |
|---|---|---|
| `params-duplicate-content` | warn | ≥ 2 crawled URLs that differ only in search params, have the same title and body text, and are each indexable with a self canonical |
| `pagination-canonical-first` | warn | Page 2+ of a listing canonicalises to page 1 |

See [CLI](CLI-and-Check-Rules#rules).
