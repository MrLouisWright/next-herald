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
- `listingMetadata(config, { path, params, indexable, pageParam?, drop?, title?, description? }): Metadata` — canonical/robots for filtered, sorted and paginated pages; `params` is a plain object you pass (v0.3, [Pagination](Pagination-and-Facets))
- `listingHref(path, params, { indexable, pageParam?, drop? }): string` — the same normalised URL for links

## `next-herald/jsonld`
- `<JsonLd data: JsonLdInput | JsonLdInput[] nonce?: string />` (Server Component)
- `serializeJsonLd(data): string` — safe serialization ([ADR 0004](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0004-safe-json-ld-serialization.md))
- `jsonLd.organization | webSite | webPage | breadcrumbList | breadcrumbsFromPath | article | blogPosting | newsArticle | product | offer | aggregateRating | review | faqPage | person | localBusiness | event | videoObject | softwareApplication | recipe | jobPosting` (phased)
- `jsonLd.with(config)` — same builders with URL resolution against `siteUrl`
- `graph(...nodes)` — `{ '@context', '@graph' }` with `@id` references; dangling ids are a dev error (v0.2, [#49](https://github.com/MrLouisWright/next-herald/issues/49))
- `jsonLd.siteGraph(config, { path, type?, node? })` — Organization/Person → WebSite → WebPage (→ `node`, e.g. an Article) linked by stable `@id`s (v0.2)
- `jsonLd.ref(config, 'publisher' | 'website')` — `{ '@id' }` reference for `author` / `publisher`

## `next-herald/sitemap`
- `defineSitemap(config, { routes, params, exclude?, extra?, entry? }): () => Promise<MetadataRoute.Sitemap>`
- `splitSitemap(source: SitemapSource, { pageSize?: number }): { generateSitemaps, sitemap }`
- `sitemapIndex(source, { path }): (req: Request) => Promise<Response>`
- `newsSitemap(config, { publication, articles }): (req: Request) => Promise<Response>`
- `sitemapImage(input)`, `sitemapVideo(input)`
- `routes[i].images` from `herald routes --images` (`next/image` sources) merged with `entry()` images: absolute, deduped, ≤ 1,000 per URL (v0.4, [Image SEO](Image-SEO))
- `lastModified` comes from your data (`entry()`), or from the manifest written by `herald routes --git-lastmod` (`routes[i].lastModified`, build time). Never defaulted to build time (v0.3, [#50](https://github.com/MrLouisWright/next-herald/issues/50))
- `SitemapSource = { count(): Promise<number>; page(index: number, size: number): Promise<SitemapEntry[]> }`

## `next-herald/robots`
- `defineRobots(config, { rules?, sitemaps?: 'auto' | string[], ai?: AiCrawlerPolicy }): MetadataRoute.Robots`
- `type AiCrawlerPolicy = { training: 'allow' | 'disallow'; search?: 'allow' | 'disallow' /* default 'allow' */; userFetch?: 'allow' | 'disallow' /* default 'allow' */ }` ([Robots](Robots#ai-crawlers-by-purpose))
- `aiUserAgents: readonly { token: string; vendor: string; purpose: 'training' | 'search' | 'userFetch'; source: string }[]` and `aiUserAgentsUpdated: string` (ISO date) — the bundled table

## `next-herald/i18n`
- `type I18nSource = { locales: readonly string[]; defaultLocale: string; href(locale: string, pathname: string): string; xDefault?: 'default-locale' | string | false }`
- `alternates(config, path, { locale, locales? }): Metadata['alternates']`
- `fromTongue(tongueConfig, getPathname): I18nSource` (structural; no import of next-tongue)
- `isHreflangCode(code): boolean`

## `next-herald/og`
- `ogImage(Template, props, { fonts?, size?, emoji? }): ImageResponse`
- `ogSize` = `{ width: 1200, height: 630 }`
- `SimpleCard`, `ArticleCard`, `ProductCard`

## `next-herald/preview` (v0.3, dev only)
- `<HeraldPreview position? defaultOpen? rules? />` (Client Component) — Google snippet, OG/X cards, chat link preview and live `checkHtml` warnings; `null` unless `NODE_ENV === 'development'` ([Preview panel](SEO-Preview-Panel))

## `next-herald/redirects` (v0.3)
- `defineRedirects(config, rules: RedirectRule[]): Redirects` — `RedirectRule = { source: string; destination: string; permanent?: boolean /* default true */; has?; missing? }`
- `redirects.toNextConfig(): Redirect[]` — for `next.config` `redirects()`
- `redirects.match(pathname): { destination: string; status: 301 | 302 | 307 | 308 } | null` — pure; call it from your own `proxy.ts` ([Redirects](Redirects))

## `next-herald/experimental` (v0.5, opt-in)
No proven search or AI-citation benefit; may change or be removed before v0.9 ([AEO stance](AEO-Stance)). Everything returns `force-static` route handlers that use only `params`, never request headers.
- `markdownRoute(source: (params) => Promise<string | null>): { GET, dynamic: 'force-static' }` — serves `text/markdown; charset=utf-8`; `null` → 404 ([#54](https://github.com/MrLouisWright/next-herald/issues/54))
- `markdownAlternate(config, path): Metadata['alternates']` — `types: { 'text/markdown': url }`
- `defineLlmsTxt(config, { summary, sections, routes? }): { GET, dynamic }` — llms.txt from the sitemap route source ([#55](https://github.com/MrLouisWright/next-herald/issues/55))
- `defineLlmsFullTxt(config, { pages: () => Promise<{ url: string; title: string; markdown: string }[]> }): { GET, dynamic }`

## `next-herald/testing`
- `checkHtml(html, { url, rules? }): Finding[]` — run page-level rules on a string (unit tests)
- `renderMetadata(metadata): string` — approximate tag rendering for snapshots (not Next's renderer)

## CLI
See [CLI](CLI-and-Check-Rules).
