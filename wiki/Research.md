# Research: competitors and Next.js metadata issues (October 2026)

Checked 2026-10-14 (AEO sections added the same day after the [SEO/AEO review](SEO-AEO-Review)). Weekly downloads from `https://api.npmjs.org/downloads/point/last-week/<pkg>` for 2026-10-06 → 2026-10-12. Release dates from the npm registry. Every link was opened during this review.

## Download numbers

| Package | Downloads/week | Latest | Notes |
|---|---|---|---|
| [schema-dts](https://www.npmjs.com/package/schema-dts) | 3,138,833 | 2.1.0 (2026-10-02) | Types only; depends on `schema-dts-lib` |
| [@vercel/og](https://www.npmjs.com/package/@vercel/og) | 2,142,440 | — | Same engine as `next/og` |
| [next-sitemap](https://www.npmjs.com/package/next-sitemap) | 638,748 | 4.2.3 (2023-09-06) | 4 runtime deps |
| [next-seo](https://www.npmjs.com/package/next-seo) | 472,622 | 7.3.0 (2026-07-29) | No runtime deps; peers `next >=13.4`, `react >=18.2` |
| [react-schemaorg](https://www.npmjs.com/package/react-schemaorg) | 138,712 | — | JSON-LD component typed with schema-dts |

## Competitors

### next-seo
- Active: 7.1.0 (Jan 2026), 7.2.0 (Feb 2026), 7.3.0 (Jul 2026); ~8.5k stars; 0 open issues at time of check.
- v7 README: "For standard meta tags (`<title>`, `<meta>`), use Next.js's built-in `generateMetadata`". The App Router value is JSON-LD components; Pages Router lives under `next-seo/pages` ([README](https://github.com/garmeeh/next-seo)).
- No sitemap, robots, hreflang checks or CI tool. → next-herald complements rather than replaces it for JSON-LD; migration guide covers both.

### next-sitemap
- Last release 4.2.3 on 2023-09-06; repo last pushed 2026-05; 2 open issues.
- Postbuild script writing to `public/`. With Next 16 on Vercel those files are not deployed: [iamvishnusankar/next-sitemap#1048](https://github.com/iamvishnusankar/next-sitemap/issues/1048). Also [#1050](https://github.com/iamvishnusankar/next-sitemap/issues/1050) (duplicate `Content-Type`).
- Runtime deps: `@corex/deepmerge`, `@next/env@^13.4.3`, `fast-glob`, `minimist`.
- → Strongest gap: a maintained App Router-native sitemap index + robots solution. ADR 0001/0007.

### Native Next.js 16 (latest 16.4.0, canary 16.5.0-canary.10)
Supports `metadata`/`generateMetadata`, `sitemap.ts` with `images`, `videos`, `alternates.languages` and `generateSitemaps` ([docs](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/sitemap), [generateSitemaps](https://nextjs.org/docs/app/api-reference/functions/generate-sitemaps)), `robots.ts`, `opengraph-image`, `ImageResponse`. Gaps: no sitemap index, no news extension, shallow metadata merging, no JSON-LD helper beyond a [guide](https://nextjs.org/docs/app/guides/json-ld) that warns `JSON.stringify` is not XSS-safe, no checker.

Open upstream issues (all open on 2026-10-14):

| Issue | Summary | Impact on next-herald |
|---|---|---|
| [vercel/next.js#98684](https://github.com/vercel/next.js/issues/98684) | Clicking a `<Link>` while its prefetch is in flight commits with no `<title>` (pending segment-cache entry) | Client-side only; documented; not detectable by `herald check` |
| [#97472](https://github.com/vercel/next.js/issues/97472) | Soft navigation leaves previous route's OG/Twitter tags mounted (16.3, Vercel) | Documented |
| [#95836](https://github.com/vercel/next.js/issues/95836) | `not-found.tsx` metadata replaced after hydration | Documented |
| [#83510](https://github.com/vercel/next.js/issues/83510) | Duplicate metadata after server-action revalidation | Documented |
| [#80583](https://github.com/vercel/next.js/issues/80583) | Link prefetch calls `generateMetadata` when streaming | Docs: cache metadata data (`"use cache"`) |
| [#95406](https://github.com/vercel/next.js/issues/95406), [#96594](https://github.com/vercel/next.js/issues/96594) | `htmlLimitedBots` customisation breaks PPR resume / caching and drops metadata | `herald check --ua` compares bot and browser HTML |
| [#98126](https://github.com/vercel/next.js/issues/98126) | Per-param sitemaps under a dynamic segment collapsed (16.3 regression) | ADR 0007 avoids that layout |
| [#87314](https://github.com/vercel/next.js/issues/87314) | `sitemap.ts` and `sitemap.xml/route.ts` conflict | Large-site mode has no `sitemap.ts` |
| [#99055](https://github.com/vercel/next.js/issues/99055) | `revalidate` sitemap emitted as static in adapter outputs | Documented |
| [#99715](https://github.com/vercel/next.js/issues/99715) | Metadata routes send `max-age=0` instead of `s-maxage` | Documented |
| [#98007](https://github.com/vercel/next.js/issues/98007) | Route handlers (incl. sitemaps) not compressed by `next start` | Gzip option for the index handler considered (v0.3) |
| [#54665](https://github.com/vercel/next.js/issues/54665) | `sitemap.ts` with custom `pageExtensions` | `herald routes` reads `pageExtensions` from config |

### schema-dts and others
- [schema-dts](https://github.com/google/schema-dts): complete schema.org types, no runtime, no serializer. We stay type-compatible ([ADR 0005](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0005-own-schema-types.md)).
- [react-schemaorg](https://www.npmjs.com/package/react-schemaorg): small JSON-LD component on schema-dts types.
- Validators: [Rich Results Test](https://search.google.com/test/rich-results), [Schema.org validator](https://validator.schema.org/) — web tools, not CI. Lighthouse SEO audits run a browser per page.

### Beyond Next.js: Nuxt SEO, Astro, WordPress (added 2026-10-14)
Full matrix and sources: [SEO/AEO review](SEO-AEO-Review).
- **Nuxt SEO** is the best-in-class benchmark (@nuxtjs/seo 124,014/wk; @nuxtjs/sitemap 306,316; @nuxtjs/robots 259,772; nuxt-schema-org 175,745; nuxt-og-image 182,419; nuxt-llms 43,857): [sitemap](https://nuxtseo.com/docs/sitemap/getting-started/introduction), [robots](https://nuxtseo.com/docs/robots/getting-started/introduction) with [AI directives](https://nuxtseo.com/docs/robots/guides/ai-directives) (Content-Signal and Content-Usage), [schema.org graph with `@id` linking](https://nuxtseo.com/docs/schema-org/getting-started/introduction), [link checker](https://nuxtseo.com/docs/link-checker/getting-started/introduction), [nuxt-llms](https://github.com/nuxt-content/nuxt-llms). next-herald aims at Nuxt SEO parity for Next.js.
- **Astro:** [@astrojs/sitemap](https://docs.astro.build/en/guides/integrations-guide/sitemap/) (3,814,836/wk) with sitemap index and i18n; meta via community packages (astro-seo 103,492/wk).
- **WordPress:** Yoast SEO Premium has [Bot Blocker](https://yoast.com/features/bot-blocker/) toggles (GPTBot, CCBot, Google-Extended) and [IndexNow](https://yoast.com/what-is-indexnow/); Rank Math has [IndexNow](https://rankmath.com/kb/how-to-use-indexnow/) in the free plugin; both generate llms.txt ([Yoast](https://yoast.com/llms-txt/), [Rank Math](https://rankmath.com/kb/llms-txt/)).
- **Lighthouse** (5,404,521/wk) and **unlighthouse** (40,625/wk) run [SEO audits](https://developer.chrome.com/docs/lighthouse/seo/) but need Chrome; `herald check` covers equivalents over HTTP ([#47](https://github.com/MrLouisWright/next-herald/issues/47), [#48](https://github.com/MrLouisWright/next-herald/issues/48)).

## AEO stance
Summary; details in [AEO stance](AEO-Stance) and [ADR 0012](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0012-ai-crawler-policy-and-aeo-stance.md).
- **Works (documented by engines):** crawlable server HTML; AI search bots allowed (OpenAI: OAI-SearchBot opt-out means not "shown in ChatGPT search answers"); normal Google Search eligibility for AI Overviews/AI Mode; accurate dates; entity markup that matches visible content; IndexNow for Bing and other IndexNow engines.
- **Unproven (experimental opt-ins, v0.5):** llms.txt / llms-full.txt, markdown alternates.
- **Hype:** see below.

## Positioning derived from this
A Next 16 App Router–only kit that **returns native objects**, fills the native gaps (index, news, deep merge, safe JSON-LD, hreflang), and ships a dependency-free **CI checker** (a headless-free Lighthouse-SEO alternative for CI). It keeps sites legible to AI search with purpose-split crawler rules, an entity graph and no-JS visibility checks. Not: "faster", "better rankings", "gets you cited by AI", or "replaces Next metadata".

## Rejected ideas
| Idea | Why rejected |
|---|---|
| `<Seo>` head component (next-seo style) | Conflicts with Next's head management and streaming metadata (ADR 0001) |
| Postbuild script writing to `public/` | Not deployed on Next 16/Vercel (next-sitemap#1048); outside build output |
| Reading `.next` manifests for routes/check | Internal, unstable (ADR 0008/0009) |
| Deriving `siteUrl` from `Host` | Breaks Cache Components, enables host-header poisoning (ADR 0006) |
| Headless browser crawling (Puppeteer/Playwright) in `herald check` | Heavy dependency, slow; ADR 0002 |
| `schema-dts` dependency / full generated vocabulary | ADR 0002/0005; curated + escape hatch |
| Bundled fonts and hosted OG service | Package size, cost (ADR 0011) |
| AI-generated titles/descriptions | Out of scope; quality and cost concerns; not a technical-SEO check |
| Auto `lastModified` = build time | Misleads crawlers; Google uses lastmod only if consistently accurate |
| Patching Next router bugs (e.g. #98684) client-side | Fragile, fights the framework; report upstream instead |
| Pages Router / Next 15 support | ADR 0003 |
| Keyword density / content scoring rules | Not reliable signals; noise in CI |

## AEO: rejected and hype
From the [SEO/AEO review](SEO-AEO-Review#3-rejected--hype), verified 2026-10-14.

| Idea | Verdict | Reason |
|---|---|---|
| llms.txt as a ranking/citation lever | Hype (shipped only as an experimental opt-in, [#55](https://github.com/MrLouisWright/next-herald/issues/55)) | A proposal ([llmstxt.org](https://llmstxt.org/)) with no standards body; no major AI search engine documents using it. Google's John Mueller: "AFAIK none of the AI services have said they're using LLMs.TXT … it's comparable to the keywords meta tag" ([SEJ](https://www.searchenginejournal.com/google-says-llms-txt-comparable-to-keywords-meta-tag/544804/)). Google: "You don't need to create new machine readable files, AI text files, or markup" ([AI features](https://developers.google.com/search/docs/appearance/ai-features)). |
| "GEO scoring" / AI-visibility scores | Reject | Not measurable from HTML; same reason as keyword density. |
| `speakable` schema | Reject | Beta, limited to news publishers in English in the US, aimed at Google Assistant ([speakable](https://developers.google.com/search/docs/appearance/structured-data/speakable)). |
| HowTo builder | Reject | HowTo rich results removed ([2023](https://developers.google.com/search/blog/2023/08/howto-faq-changes)). |
| Marketing FAQPage as a rich-result win | Reject | FAQ rich results limited to well-known government and health sites since 2023; builder kept without the promise. |
| Google Indexing API for ordinary pages | Reject | Officially limited to `JobPosting` and `BroadcastEvent` in `VideoObject` ([quickstart](https://developers.google.com/search/apis/indexing-api/v3/quickstart)); needs a service-account credential. |
| Blocking Google-Extended to "stay out of AI Overviews" | Reject as advice | AI Overviews use Googlebot ([AI features](https://developers.google.com/search/docs/appearance/ai-features)). |
| Accept-header negotiation in library code | Reject | Reads request headers (ADR 0006). `proxy.ts` recipe only ([AEO stance](AEO-Stance#recipe-accept-negotiation-in-proxyts-experimental)). |
| AI-generated FAQ blocks / "answer-first" rewriting | Reject | Content generation is out of scope; risks scaled-content spam policies. |
| Pay-per-crawl integration | Reject (out of scope) | CDN-level feature, not something a framework library can enforce. |
| robots.txt as enforcement against bots | Reject the claim | Compliance is voluntary; use WAF/bot management. |
| Headless rendering to "see what AI sees" | Reject | Most AI crawlers do not render JS, so raw server HTML *is* what they see (ADR 0008). |
| A single `aiCrawlers` toggle | Replaced | Merged training, search and user-fetch bots; blocking "AI" wholesale would also remove a site from AI search answers ([ADR 0012](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0012-ai-crawler-policy-and-aeo-stance.md)). |
