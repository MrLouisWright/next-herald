# API reference

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

All functions are pure unless noted. Types are exported from the same entry point.

## `next-herald/config`
- `defineHerald(options: HeraldOptions): HeraldConfig` — validates and freezes. Throws `HeraldConfigError` with the option path. Options: [Configuration](Configuration).

## `next-herald/metadata`
- `createMetadata(config, overrides?: Metadata): Metadata`
- `pageMetadata(config, input: PageInput): Metadata` — `PageInput = { path: string; title?: string | { absolute: string }; description?: string; images?: OgImageInput[]; type?: 'website' | 'article' | 'profile'; publishedTime?; modifiedTime?; authors?: string[]; tags?: string[]; noindex?: boolean; locale?: string; locales?: string[]; keepQuery?: boolean; extra?: Metadata }`
- `mergeMetadata(...parts: (Metadata | undefined)[]): Metadata` — deep merge for `openGraph`, `twitter`, `robots`, `alternates`, `other`; arrays replaced.
- `canonicalUrl(config, path): string`

## `next-herald/jsonld`
- `<JsonLd data: JsonLdInput | JsonLdInput[] nonce?: string />` (Server Component)
- `serializeJsonLd(data): string` — safe serialization ([ADR 0004](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0004-safe-json-ld-serialization.md))
- `jsonLd.organization | webSite | webPage | breadcrumbList | breadcrumbsFromPath | article | blogPosting | newsArticle | product | offer | aggregateRating | review | faqPage | person | localBusiness | event | videoObject | softwareApplication | recipe | jobPosting` (phased)
- `jsonLd.with(config)` — same builders with URL resolution against `siteUrl`
- `graph(...nodes)` — `{ '@context', '@graph' }` with `@id` references

## `next-herald/sitemap`
- `defineSitemap(config, { routes, params, exclude?, extra?, entry? }): () => Promise<MetadataRoute.Sitemap>`
- `splitSitemap(source: SitemapSource, { pageSize?: number }): { generateSitemaps, sitemap }`
- `sitemapIndex(source, { path }): (req: Request) => Promise<Response>`
- `newsSitemap(config, { publication, articles }): (req: Request) => Promise<Response>`
- `sitemapImage(input)`, `sitemapVideo(input)`
- `SitemapSource = { count(): Promise<number>; page(index: number, size: number): Promise<SitemapEntry[]> }`

## `next-herald/robots`
- `defineRobots(config, { rules?, sitemaps?: 'auto' | string[], aiCrawlers?: 'allow' | 'disallow' }): MetadataRoute.Robots`

## `next-herald/i18n`
- `type I18nSource = { locales: readonly string[]; defaultLocale: string; href(locale: string, pathname: string): string; xDefault?: 'default-locale' | string | false }`
- `alternates(config, path, { locale, locales? }): Metadata['alternates']`
- `fromTongue(tongueConfig, getPathname): I18nSource` (structural; no import of next-tongue)
- `isHreflangCode(code): boolean`

## `next-herald/og`
- `ogImage(Template, props, { fonts?, size?, emoji? }): ImageResponse`
- `ogSize` = `{ width: 1200, height: 630 }`
- `SimpleCard`, `ArticleCard`, `ProductCard`

## `next-herald/testing`
- `checkHtml(html, { url, rules? }): Finding[]` — run page-level rules on a string (unit tests)
- `renderMetadata(metadata): string` — approximate tag rendering for snapshots (not Next's renderer)

## CLI
See [CLI](CLI-and-Check-Rules).
