# Metadata

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

next-herald builds plain `Metadata` objects ([ADR 0001](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0001-build-on-native-metadata-apis.md)). You export them from `layout.tsx` / `page.tsx` as `metadata` or from `generateMetadata`. Next renders the tags.

## `createMetadata(config, overrides?)`
For the root layout. Produces `metadataBase`, `title: { default, template }`, `description`, `openGraph` (siteName, locale, type `website`), `twitter`, `robots` (from `indexing`), `alternates.canonical: '/'` only for the root page if requested.

## `pageMetadata(config, input)`
For pages. `input`: `path` (required, used for canonical and `og:url`), `title`, `description`, `images`, `type` (`website`·`article`·`profile`), article fields, `noindex`, `alternates` (hreflang, see [i18n](hreflang-and-i18n)). It:
- copies `title`/`description` into `openGraph` and `twitter` explicitly when absent, so the output does not depend on Next's per-segment merge rules;
- normalizes the canonical path (`trailingSlash`, collapses `//`, strips query and hash unless `keepQuery`);
- warns in dev on empty strings, titles over 70 characters and descriptions over 160 (thresholds configurable; warnings only).

## Merging: `mergeMetadata(...parts)`
Next merges metadata **shallowly** per top-level key across segments: a page's `openGraph` replaces the layout's `openGraph` entirely ([docs](https://nextjs.org/docs/app/api-reference/functions/generate-metadata)). `mergeMetadata` deep-merges `openGraph`, `twitter`, `robots`, `alternates` and `other`, with arrays (`images`, `keywords`) replaced, not concatenated. `pageMetadata` uses it with the config defaults, so pages keep `siteName` etc.

## Templates
`title.template` applies to child segments, not to the segment that defines it (Next behaviour). Use `title: { absolute: '…' }` to opt out on a page.

## Known Next.js issues
- Clicking a `<Link>` while its prefetch is in flight can commit with no `<title>` ([vercel/next.js#98684](https://github.com/vercel/next.js/issues/98684), open). This is client navigation; server HTML is correct, so `herald check` cannot see it.
- Soft navigation may leave the previous route's OG tags mounted ([#97472](https://github.com/vercel/next.js/issues/97472)); `generateMetadata` duplicated after server-action revalidation ([#83510](https://github.com/vercel/next.js/issues/83510)).
- Custom `htmlLimitedBots` patterns can drop metadata under PPR ([#95406](https://github.com/vercel/next.js/issues/95406)).

These are tracked in [Troubleshooting](Troubleshooting); next-herald does not patch Next internals.
