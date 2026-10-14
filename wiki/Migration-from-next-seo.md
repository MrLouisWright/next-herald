# Migration from next-seo

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

[next-seo](https://github.com/garmeeh/next-seo) 7.x (latest 7.3.0, July 2026) already recommends Next's `generateMetadata` for meta tags in the App Router and keeps `<NextSeo>` for the Pages Router under `next-seo/pages`. Its value in the App Router is JSON-LD components. There is no reason to migrate a **Pages Router** app: next-herald does not support it ([ADR 0003](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0003-nextjs-16-only.md)).

| next-seo | next-herald |
|---|---|
| `<DefaultSeo>` / `next-seo.config` | `defineHerald()` + `createMetadata()` in the root layout |
| `<NextSeo title description canonical openGraph>` (Pages) | `pageMetadata(herald, { path, title, description, images })` from `generateMetadata` |
| `titleTemplate` | `title.template` (`%s`) |
| `noindex`/`nofollow` props | `pageMetadata({ noindex: true })`; `indexing` config for environments |
| `languageAlternates` | `i18n` config → automatic `alternates.languages` |
| JSON-LD components (`ArticleJsonLd`, `ProductJsonLd`, …) | `<JsonLd data={jsonLd.article({...})} />` |
| `additionalMetaTags` / `additionalLinkTags` | `Metadata.other`, `icons`, `manifest` (native) |

Steps:
1. Move config into `herald.config.ts`; export `createMetadata(herald)` from `app/layout.tsx`.
2. Replace each page's tags with `generateMetadata` returning `pageMetadata`.
3. Replace JSON-LD components one by one; compare output with the [Rich Results Test](https://search.google.com/test/rich-results).
4. Run `herald check --build` before and after; the diff of rule results is your regression report.

A codemod is planned after v1.0 ([ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md)).
