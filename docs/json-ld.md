# JSON-LD structured data

> **Status: planned / pre-release.** This page documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](../ROADMAP.md).

## `<JsonLd data nonce? />`
A Server Component that renders `<script type="application/ld+json">` with **safe serialization** ([ADR 0004](adr/0004-safe-json-ld-serialization.md)): `<`, `>`, `&`, U+2028 and U+2029 are escaped as `\uXXXX`, so a value like `"</script><script>alert(1)</script>"` cannot break out. `@context` is added when missing. `data` can be one object, an array, or `{ '@graph': [...] }`.

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
Curated types ([ADR 0005](adr/0005-own-schema-types.md)). v0.1: `organization`, `webSite` (with `searchAction`), `webPage`, `breadcrumbList`, `article` / `blogPosting`, `product` + `offer` + `aggregateRating` + `review`, `faqPage`. v0.4 adds `newsArticle`, `person`, `localBusiness`, `event`, `videoObject`, `softwareApplication`, `recipe`, `jobPosting`.

Builders type properties Google lists as *required* for a rich result as required, and dates as ISO 8601 strings or `Date`. Relative URLs are resolved against `siteUrl` when you pass the config: `jsonLd.with(herald).article(...)`.

### schema-dts
next-herald does not depend on [schema-dts](https://github.com/google/schema-dts). `<JsonLd>` accepts any `{ '@type': string }` object, so `WithContext<Recipe>` values from schema-dts type-check as-is.

## Breadcrumbs from the path
`jsonLd.breadcrumbsFromPath('/docs/sitemaps', { labels })` builds a `BreadcrumbList` with absolute URLs.

## Validation
`herald check` rules `jsonld-parse`, `jsonld-type`, `jsonld-required`, `jsonld-recommended` ([CLI](cli.md)). Also test with Google's [Rich Results Test](https://search.google.com/test/rich-results) and the [Schema.org validator](https://validator.schema.org/); herald's checks are a subset and do not guarantee rich results.
