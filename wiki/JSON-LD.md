# JSON-LD structured data

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

## `<JsonLd data nonce? />`
A Server Component that renders `<script type="application/ld+json">` with **safe serialization** ([ADR 0004](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0004-safe-json-ld-serialization.md)): `<`, `>`, `&`, U+2028 and U+2029 are escaped as `\uXXXX`, so a value like `"</script><script>alert(1)</script>"` cannot break out. `@context` is added when missing. `data` can be one object, an array, or `{ '@graph': [...] }`.

```tsx
import { JsonLd, jsonLd } from 'next-herald/jsonld';

export default async function Page() {
  const product = await getProduct();
  return (
    <>
      <JsonLd data={jsonLd.product({
        name: product.name,                 // required by the builder type
        image: [product.image],
        offers: jsonLd.offer({ price: product.price, priceCurrency: 'EUR', availability: 'InStock' }),
      })} />
      <h1>{product.name}</h1>
    </>
  );
}
```

Place it in a page or layout; it is plain HTML, no client JS. With a strict CSP, pass the per-request `nonce` (that page is then dynamic, which is a CSP trade-off, not a next-herald one).

## Builders and types
Curated types ([ADR 0005](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0005-own-schema-types.md)). v0.1: `organization`, `webSite` (with `searchAction`), `webPage`, `breadcrumbList`, `article` / `blogPosting`, `product` + `offer` + `aggregateRating` + `review`, `faqPage`. v0.2 adds `person`, `graph()` and the default entity graph ([#49](https://github.com/MrLouisWright/next-herald/issues/49)). v0.4 adds `newsArticle`, `localBusiness`, `event`, `videoObject`, `softwareApplication`, `recipe`, `jobPosting`.

Builders type properties Google lists as *required* for a rich result as required, and dates as ISO 8601 strings or `Date`. Relative URLs are resolved against `siteUrl` when you pass the config: `jsonLd.with(herald).article(...)`.

### FAQPage and HowTo: no rich-result promise
`faqPage` is kept for correct markup, but **it is not a rich-result feature for most sites**: since August 2023 Google shows FAQ rich results only for well-known, authoritative government and health sites, and HowTo rich results were removed ([Google, 2023](https://developers.google.com/search/blog/2023/08/howto-faq-changes); see also [simplifying search results, 2025](https://developers.google.com/search/blog/2025/06/simplifying-search-results) and the [FAQPage doc](https://developers.google.com/search/docs/appearance/structured-data/faqpage)). next-herald ships no HowTo builder. `herald check` reports `jsonld-deprecated-type` (info) for these ([#52](https://github.com/MrLouisWright/next-herald/issues/52)).

### schema-dts
next-herald does not depend on [schema-dts](https://github.com/google/schema-dts). `<JsonLd>` accepts any `{ '@type': string }` object, so `WithContext<Recipe>` values from schema-dts type-check as-is.

## Entity graph by default (v0.2)
Declare who publishes the site once, and every page can emit the same linked graph ([#49](https://github.com/MrLouisWright/next-herald/issues/49)):

```ts
// herald.config.ts
export default defineHerald({
  siteUrl: 'https://example.com',
  siteName: 'Example',
  organization: { name: 'Example Ltd', logo: '/logo.png', sameAs: ['https://github.com/example', 'https://www.linkedin.com/company/example'] },
});
```

```tsx
<JsonLd data={jsonLd.siteGraph(herald, {
  path: `/blog/${slug}`,
  node: jsonLd.article({ headline: post.title, datePublished: post.date, dateModified: post.updated, author: jsonLd.ref(herald, 'publisher') }),
})} />
```

This emits `Organization` (`https://example.com/#organization`), `WebSite` (`/#website`, `publisher` → org), `WebPage` (`<url>#webpage`, `isPartOf` → website) and the Article with `author`/`publisher` by `@id`. Use `person` instead of `organization` for a personal site (`/#person`). The ids are stable across pages; `herald check` reports `jsonld-entity-id` and `sameas-inconsistent` when they drift ([#52](https://github.com/MrLouisWright/next-herald/issues/52)). Google recommends that Article `author` is a Person or Organization with a name and URL ([Article](https://developers.google.com/search/docs/appearance/structured-data/article)). Keep the markup consistent with visible content (`jsonld-visible-mismatch`, [#51](https://github.com/MrLouisWright/next-herald/issues/51)).

## Breadcrumbs from the path
`jsonLd.breadcrumbsFromPath('/docs/sitemaps', { labels })` builds a `BreadcrumbList` with absolute URLs.

## Validation
`herald check` rules `jsonld-parse`, `jsonld-type`, `jsonld-required`, `jsonld-recommended`, plus `jsonld-author`, `jsonld-date-missing`, `jsonld-visible-mismatch`, `jsonld-entity-id`, `sameas-inconsistent`, `jsonld-deprecated-type` ([CLI](CLI-and-Check-Rules)). Also test with Google's [Rich Results Test](https://search.google.com/test/rich-results) and the [Schema.org validator](https://validator.schema.org/); herald's checks are a subset and do not guarantee rich results.
