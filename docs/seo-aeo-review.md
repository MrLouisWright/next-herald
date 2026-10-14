# SEO and AEO/GEO review (2026-10-14)

> **Status:** approved by the maintainer on 2026-10-14. All top-ranked recommendations, plus llms.txt as an experimental opt-in in v0.5, are scheduled; see [ROADMAP](../ROADMAP.md), [AEO stance](aeo.md) and [ADR 0012](adr/0012-ai-crawler-policy-and-aeo-stance.md). The review text below is kept as written. The **verification addendum** at the end records which claims marked *(recall)* were confirmed against live sources and which were dropped or softened. Where the two disagree, the addendum wins.


Review only. I made no changes to the repo. Repo state reviewed: README, ROADMAP (v0.1 to post-1.0, scope freeze 2026-10-14), docs/*, ADR 0001–0011, docs/research.md, docs/review.md.
npm weekly downloads come from `api.npmjs.org/downloads/point/last-week/<pkg>` for **2026-10-06 → 2026-10-12**. Every URL below returned HTTP 200 when I checked it today, unless marked otherwise. Where a quote comes from my own knowledge and not a page fetched today, it is marked *(recall, re-verify)*.

---

## 0. Headline

next-herald is **ahead of everything in the Next.js ecosystem** on three things: native-object output, a sitemap index with news support, and a zero-dependency CI checker. It is **on par with Nuxt SEO and behind it overall**. Nuxt SEO is the real benchmark. It ships sitemap, robots, schema.org graph, OG images, a link checker, SEO utils and `nuxt-llms`, and it has a large install base (@nuxtjs/sitemap 306K/wk, @nuxtjs/robots 260K/wk). The plan is **almost silent on AEO**. Its only AEO item is an `aiCrawlers: 'allow' | 'disallow'` toggle, and that toggle merges together crawlers that do different jobs: training, search-indexing and user-fetch bots. That is the biggest gap, and the cheapest fix.

---

## 1. Competitor feature matrix

Legend: ✅ yes · ◐ partial · — no · (p) planned in next-herald

| Feature | next-herald (planned) | Native Next 16 | next-seo 7.3.0 | next-sitemap 4.2.3 | schema-dts 2.1.0 | Nuxt SEO 6.0.1 | Astro (@astrojs/sitemap 3.7.4, astro-seo 1.2.0) | Yoast / Rank Math | Lighthouse 13.5 / unlighthouse 0.19 |
|---|---|---|---|---|---|---|---|---|---|
| Downloads/wk | — | — | 472,622 | 638,748 | 3,138,833 | @nuxtjs/seo 124,014; sitemap 306,316; robots 259,772; schema-org 175,745; og-image 182,419; nuxt-llms 43,857 | sitemap 3,814,836; astro-seo 103,492; astro-robots-txt 25,002 | WP plugins (not npm) | lighthouse 5,404,521; unlighthouse 40,625 |
| Typed meta builders | ✅(p) native `Metadata` | ✅ | ◐ (defers to native) | — | — | ✅ `useSeoMeta` + site config | ◐ astro-seo component | ✅ UI | — |
| Deep metadata merge | ✅(p) | — | — | — | — | ✅ | — | n/a | — |
| Site-wide config / identity | ✅(p) `defineHerald` | ◐ metadataBase | ◐ | ◐ | — | ✅ nuxt-site-config | ◐ `site` | ✅ | — |
| JSON-LD typed + XSS-safe | ✅(p) | guide only | ✅ components | — | types only | ✅ `@graph` w/ `@id` linking, auto Org/WebSite/WebPage | — (manual) | ✅ auto graph | — |
| Auto-linked `@graph` (Org→WebSite→WebPage→Article→Person) | ◐(p v0.4 `graph()`) | — | — | — | — | ✅ | — | ✅ | — |
| Sitemap from routes | ✅(p) typed params | manual | — | ✅ postbuild | — | ✅ auto + sources | ✅ auto | ✅ | — |
| Sitemap index | ✅(p v0.3) | — | — | ✅ | — | ✅ multi-sitemaps | ✅ | ✅ | — |
| News / image / video sitemap | ✅(p) | img/video | — | — | — | ✅ | ◐ | ✅ (News = premium) | — |
| Accurate lastmod source | — (rejects build time, no source) | manual | — | build time | — | ✅ (content `updatedAt`, git-aware options) | ◐ `lastmod` option | ✅ post modified | — |
| hreflang | ✅(p) | manual | ◐ | ✅ | — | ✅ via i18n | ✅ i18n option | ✅ (WPML) | ✅ audit |
| robots.txt builder | ✅(p) | ✅ | — | ✅ | — | ✅ + non-prod lockdown, X-Robots-Tag | ◐ third party | ✅ | ✅ audit |
| AI crawler controls | ◐(p) single toggle | manual | — | — | — | ✅ robots groups incl. AI bots, content signals *(recall, re-verify)* | — | ✅ Yoast/Rank Math AI crawler toggles *(recall)* | — |
| llms.txt | — | — | — | — | — | ✅ nuxt-llms | 3rd party | ✅ Yoast & Rank Math generate it | — |
| Markdown versions of pages | — | — | — | — | — | ◐ (Nuxt Content raw md *(recall)*) | 3rd party | — | — |
| OG image templates | ✅(p) | ✅ ImageResponse | — | — | — | ✅ nuxt-og-image | 3rd party | ◐ | — |
| CI checker (HTTP crawl) | ✅(p) `herald check`, SARIF | — | — | — | — | ◐ link-checker + devtools | — | — | ✅ (needs Chrome) |
| Link checker (broken internal links) | ◐ (only canonical/sitemap targets) | — | — | — | — | ✅ | — | ◐ premium | ◐ |
| Content/readability analysis | — (non-goal) | — | — | — | — | — | — | ✅ | — |
| IndexNow | — | — | — | — | — | — *(recall)* | 3rd party | ✅ Rank Math & Yoast Premium *(recall)* | — |
| Redirect manager | — | next.config | — | — | — | — | — | ✅ | — |
| Zero runtime deps | ✅ | n/a | ✅ | — (4) | 1 | — | — | n/a | — |
| Cache Components / static-safe | ✅ | ✅ | ✅ | ✅ (static files) | n/a | n/a | n/a | n/a | n/a |

Sources: [nuxtseo sitemap](https://nuxtseo.com/docs/sitemap/getting-started/introduction), [robots](https://nuxtseo.com/docs/robots/getting-started/introduction), [schema-org](https://nuxtseo.com/docs/schema-org/getting-started/introduction), [seo-utils](https://nuxtseo.com/docs/seo-utils/getting-started/introduction), [link-checker](https://nuxtseo.com/docs/link-checker/getting-started/introduction), [nuxt-llms](https://github.com/nuxt-content/nuxt-llms), [@astrojs/sitemap](https://docs.astro.build/en/guides/integrations-guide/sitemap/), [Yoast features](https://yoast.com/features/), [Yoast llms.txt](https://yoast.com/llms-txt/), [Rank Math llms.txt](https://rankmath.com/kb/llms-txt/), [Lighthouse SEO audits](https://developer.chrome.com/docs/lighthouse/seo/), [unlighthouse](https://unlighthouse.dev/), [Next htmlLimitedBots](https://nextjs.org/docs/app/api-reference/config/next-config-js/htmlLimitedBots). (rankmath.com/features/ returned 404.)

**Ahead:** zero deps together with native objects; App Router sitemap index on Next 16, where next-sitemap is broken on Vercel (#1048); XSS-safe JSON-LD; the planned **headless-free CI checker with SARIF**. No Next/Astro competitor has that checker, and Lighthouse needs Chrome.
**On par:** hreflang, OG templates, news/image/video sitemaps, preview lockdown (Nuxt and Yoast have all of these).
**Behind:** a linked `@graph` entity model, AI-crawler granularity, llms.txt (CMS and Nuxt already ship it), a lastmod source, an internal broken-link check, IndexNow, and any AEO-oriented checks.

---

## 2. Recommended additions (ranked by impact ÷ effort)

The scope freeze stays: none of these go into v0.1 unless they are data or naming fixes inside existing v0.1 issues (items 1 and 2 qualify, because they change the shape of an existing API before it ships).

| # | Addition | SEO/AEO | Impact | Effort | Fit (zero deps, Cache Components) | Milestone | Evidence |
|---|---|---|---|---|---|---|---|
| 1 | **Split the AI-crawler toggle by purpose.** Replace `aiCrawlers: 'allow'|'disallow'` with `ai: { training: 'allow'|'disallow', search: 'allow'|'disallow', userFetch: 'allow' }` backed by a dated data table: training = GPTBot, ClaudeBot, Google-Extended, Applebot-Extended, CCBot, Bytespider, meta-externalagent; search = OAI-SearchBot, Claude-SearchBot, PerplexityBot; user = ChatGPT-User, Claude-User, Perplexity-User. Default: training per the user's choice, **search allowed**. | AEO | Very high: blocking "AI" wholesale silently removes the site from ChatGPT search citations | XS (data + types) | Pure data, static `robots.ts` ✅ | **v0.1** (#11 API shape fix) | [OpenAI bots](https://platform.openai.com/docs/bots) ("Sites that are opted out of OAI-SearchBot will not be shown in ChatGPT search answers"; settings independent), [Anthropic crawlers](https://support.anthropic.com/en/articles/8896518-does-anthropic-crawl-data-from-the-web-and-how-can-site-owners-block-the-crawler), [Perplexity bots](https://docs.perplexity.ai/guides/bots), [Google common crawlers (Google-Extended)](https://developers.google.com/search/docs/crawling-indexing/google-common-crawlers), [Applebot-Extended](https://support.apple.com/en-us/119829), [CCBot](https://commoncrawl.org/ccbot), [Meta crawlers](https://developers.facebook.com/docs/sharing/webmasters/web-crawlers) |
| 2 | **Document honestly what Google-Extended does *not* cover.** It controls Gemini training/grounding, **not** AI Overviews/AI Mode, which use Googlebot. You cannot opt out of AI Overviews without opting out of Search (beyond `nosnippet`/`max-snippet`). Expose `max-snippet`/`nosnippet`/`data-nosnippet` guidance. | AEO | High (stops a common mistake) | XS (docs) | ✅ | v0.1 docs | [Google common crawlers](https://developers.google.com/search/docs/crawling-indexing/google-common-crawlers), [AI features & your website](https://developers.google.com/search/docs/appearance/ai-features) |
| 3 | **Entity graph by default:** `defineHerald({ organization | person, sameAs: [...] })` → auto `Organization`/`WebSite`/`WebPage` with stable `@id`s (`${siteUrl}/#org`) and articles referencing `author`/`publisher` by `@id`. Move `graph()` and `person` up from v0.4. | SEO+AEO | High (entity consistency, E-E-A-T signals, Nuxt/Yoast parity) | M | Pure builders ✅ | v0.4 → pull to **v0.2-adjacent** after freeze | [Article structured data (author best practices)](https://developers.google.com/search/docs/appearance/structured-data/article), [nuxtseo schema-org](https://nuxtseo.com/docs/schema-org/getting-started/introduction) |
| 4 | **"Bots see content" checks:** `herald check --ua` already fetches as bot. Add `content-js-only` (rendered `<main>`/`<article>` text below N chars in server HTML), `content-bot-parity` (text diff of bot UA vs browser UA, catching `htmlLimitedBots` regressions) and `metadata-streamed` (title/meta outside `<head>` for non-limited bots). | AEO+SEO | High: AI crawlers don't run JS | S (scanner exists; add body-text extraction) | ✅ HTTP only, no browser | v0.2 | [Vercel "Rise of the AI crawler"](https://vercel.com/blog/the-rise-of-the-ai-crawler) ("none of the major AI crawlers currently render JavaScript": OAI-SearchBot, ChatGPT-User, GPTBot, ClaudeBot, Meta-ExternalAgent, Bytespider, PerplexityBot; Gemini/AppleBot render). Note: the study dates from Dec 2024, so re-test. |
| 5 | **Accurate lastmod source helpers:** `lastModified` from content `updatedAt`, plus a CLI `herald routes --git-lastmod` (via `git log -1 --format=%cI`, `child_process`, no dep). Rule `lastmod-future`, `lastmod-uniform` (all entries equal means build-time stamping) and `lastmod-mismatch` (sitemap lastmod vs JSON-LD `dateModified` / `article:modified_time`). | SEO+AEO (freshness) | High | S | Build-time data ✅; never request-time | v0.3 | [Google build sitemap ("Google uses lastmod if consistently and verifiably accurate")](https://developers.google.com/search/docs/crawling-indexing/sitemaps/build-sitemap), [Article dateModified](https://developers.google.com/search/docs/appearance/structured-data/article) |
| 6 | **IndexNow helper:** `herald indexnow --changed` (CLI, global `fetch`), key file via `app/[key].txt/route.ts` `force-static`, posting changed sitemap URLs (diff against the previous build's sitemap). Feeds Bing (which grounds ChatGPT search and Copilot), Yandex, Seznam, Naver. | AEO+SEO | Med-high | S | ✅ CLI only; no runtime | v0.5 | [IndexNow docs](https://www.indexnow.org/documentation) |
| 7 | **Markdown alternates** (opt-in recipe + helper): `app/[...slug]/md/route.ts` or `.md` route with `force-static`, and `<link rel="alternate" type="text/markdown">` via `alternates.types`. **Accept-header negotiation only as a middleware/proxy recipe**, because reading `Accept` in a page breaks Cache Components (ADR 0006). The helper takes markdown the user already has (MDX/CMS); herald does not convert HTML. | AEO | Medium, with evidence still forming (Cloudflare and Vercel ship it; no engine has confirmed ranking/citation use) | S-M | ✅ if static route; negotiation must live in `proxy.ts` | v0.5 (experimental) | [Cloudflare Markdown for Agents](https://blog.cloudflare.com/markdown-for-agents/), [Vercel content negotiation](https://vercel.com/blog/making-agent-friendly-pages-with-content-negotiation) |
| 8 | **llms.txt generator (opt-in, labelled low-evidence):** `defineLlmsTxt(herald, { sections })` route returning text from the same route list/sitemap source; optional `llms-full.txt` from user-provided markdown. Cheap because the route source already exists. Its main real use is coding agents and IDE doc tools, not search citation. | AEO (low) | Low-med | XS-S | ✅ static route | v0.5, flagged experimental | [llmstxt.org](https://llmstxt.org/), [Yoast](https://yoast.com/llms-txt/), [Rank Math](https://rankmath.com/kb/llms-txt/), [nuxt-llms](https://github.com/nuxt-content/nuxt-llms) |
| 9 | **Content signals line in robots** (`Content-Signal: search=yes, ai-input=yes, ai-train=no`) as an opt-in passthrough. It is non-standard, so it is emitted as a comment-compatible line and documented as an expression of preference, not enforcement. Mention pay-per-crawl as infrastructure (Cloudflare), out of scope. | AEO | Low-med (legal/preference signal) | XS | ✅ | v0.3 | [contentsignals.org](https://contentsignals.org/), [Cloudflare content signals](https://blog.cloudflare.com/content-signals-policy/), [Pay per crawl](https://blog.cloudflare.com/introducing-pay-per-crawl/) |
| 10 | **Citation-friendly structure checks** (info severity, HTML-only, no scoring): exactly one `<h1>`, heading order, a visible byline/date when Article JSON-LD claims `author`/`datePublished`, JSON-LD vs visible-content consistency (headline equals h1/title, Product price present in text). | AEO+SEO | Medium | S-M | ✅ | v0.4 | [Article guidelines](https://developers.google.com/search/docs/appearance/structured-data/article); Google structured-data policy that markup must reflect visible content |
| 11 | **Internal broken-link rule** (`link-broken`, `link-redirect`): the crawler already follows links. | SEO | Medium (Nuxt parity) | XS | ✅ | v0.2 | [nuxtseo link-checker](https://nuxtseo.com/docs/link-checker/getting-started/introduction), [Lighthouse SEO](https://developer.chrome.com/docs/lighthouse/seo/) |
| 12 | **Lighthouse SEO audit parity:** `viewport-missing`, `http-status` (4xx on indexable pages), `link-text-generic`, `img-alt-missing`, `crawlable-anchors` (no `href`/`javascript:`). | SEO | Medium | XS-S | ✅ | v0.2/v0.4 | [Lighthouse SEO audits](https://developer.chrome.com/docs/lighthouse/seo/) |
| 13 | **AI referral measurement recipe** (docs only): match `document.referrer`/server logs for chatgpt.com, perplexity.ai, copilot.microsoft.com, gemini.google.com, claude.ai; note ChatGPT appends `utm_source=chatgpt.com`; Bing Webmaster Tools has an AI Performance report *(recall, re-verify: I could not find a working URL)*; Google AI Overview/AI Mode clicks are folded into Search Console "Web". Optionally `herald logs` to count AI bot hits from an access log (Node `readline`). | AEO | Medium (users want proof) | XS docs / S CLI | ✅ no runtime code | v0.5 | [OpenAI bots](https://platform.openai.com/docs/bots), [Google AI features](https://developers.google.com/search/docs/appearance/ai-features) |
| 14 | **Demote FAQPage in batch 1**: keep the builder but document that FAQ rich results show only for well-known government/health sites (2023) and that Google later dropped several more types (2025). Don't market it as a rich-result win. Don't add HowTo. | SEO | Medium (honesty, scope) | XS | ✅ | v0.1 docs | [HowTo/FAQ changes](https://developers.google.com/search/blog/2023/08/howto-faq-changes), [Simplifying search results (2025)](https://developers.google.com/search/blog/2025/06/simplifying-search-results), [FAQPage doc](https://developers.google.com/search/docs/appearance/structured-data/faqpage) |

---

## 3. Rejected / hype

| Idea | Verdict | Reason |
|---|---|---|
| **llms.txt as a ranking/citation lever** | Hype (but cheap to ship opt-in) | It is a proposal ([llmstxt.org](https://llmstxt.org/)) with no standards body behind it. No major answer engine (OpenAI, Google, Perplexity, Anthropic search) has publicly documented that it reads llms.txt for search/citation. Google's John Mueller has said that no AI system uses it and compared it to the keywords meta tag, and Google's AI-features guidance says no special AI files or markup are needed *(recall, re-verify the exact wording on [ai-features](https://developers.google.com/search/docs/appearance/ai-features))*. Adoption: Yoast, Rank Math, Nuxt and many docs sites generate it. The real consumers are coding agents and doc tools. Ship it as experimental and never claim visibility gains. |
| "GEO scoring" / AI-visibility scores | Reject | Not measurable from HTML. Same reason the repo already rejects keyword density. |
| `speakable` schema | Reject | Beta, US/English news publishers only, aimed at Google Assistant; no AI-engine evidence ([speakable](https://developers.google.com/search/docs/appearance/structured-data/speakable)). |
| HowTo builder | Reject | HowTo rich results removed ([2023](https://developers.google.com/search/blog/2023/08/howto-faq-changes)). |
| Google Indexing API for ordinary pages | Reject | Officially limited to `JobPosting` and `BroadcastEvent`-in-`VideoObject`, with a default quota of 200 publish requests/day ([quickstart](https://developers.google.com/search/apis/indexing-api/v3/quickstart), [quota](https://developers.google.com/search/apis/indexing-api/v3/quota-pricing)). It also needs a service-account credential, which a zero-dep CLI should not manage. At most, a docs mention for job boards. |
| Blocking Google-Extended to "stay out of AI Overviews" | Reject as advice | It does not do that (see item 2). |
| Accept-header negotiation inside library code | Reject | It reads request headers, which breaks ADR 0006 and the static shell. Use a proxy recipe only. |
| AI-generated FAQ blocks / "answer-first" content rewriting | Reject | Content generation is out of scope; it also risks scaled-content spam policies. |
| Pay-per-crawl integration | Reject (out of scope) | CDN-level feature ([Cloudflare](https://blog.cloudflare.com/introducing-pay-per-crawl/)), not something a framework library can enforce. |
| Treating robots.txt as enforcement against Bytespider etc. | Reject the claim | Compliance is voluntary. Keep the existing "not a guarantee" note and point to WAF/bot management. |
| Headless rendering to "see what AI sees" | Reject | It would be wrong anyway: most AI crawlers do not render JS, so the raw server HTML *is* what they see. That is a point in favour of the existing ADR 0008. |

---

## 4. Proposed new `herald check` rules

| ID | Severity | Since | Reports |
|---|---|---|---|
| `robots-blocks-ai-search` | warn | v0.2 | robots.txt disallows OAI-SearchBot / Claude-SearchBot / PerplexityBot (or `*` without an allow) while `ai.search` is `allow` or unset: the site will not be cited |
| `robots-ai-inconsistent` | info | v0.2 | Training bots allowed but search bots blocked (usually the inverse of intent) |
| `robots-ai-list-stale` | info | v0.2 | Bundled AI-UA table older than 180 days (prompts an upgrade) |
| `content-js-only` | warn | v0.2 | Indexable page whose server HTML has under N words of body text (default 50) |
| `content-bot-parity` | warn | v0.2 | Bot-UA and browser-UA HTML differ materially in title/meta/body text |
| `metadata-in-body` | warn | v0.2 | `<title>`/canonical/description streamed outside `<head>` for a crawler UA |
| `link-broken` | error | v0.2 | Same-origin link returns 4xx/5xx |
| `link-redirect` | info | v0.2 | Internal link hits a redirect |
| `http-status-indexable` | error | v0.2 | Page in a sitemap or linked internally returns a non-200 status |
| `viewport-missing` | warn | v0.2 | No `meta name=viewport` |
| `anchor-not-crawlable` | warn | v0.2 | `<a>` without `href` or with `javascript:` |
| `h1-missing` / `h1-multiple` | info | v0.4 | Heading structure |
| `heading-order` | info | v0.4 | Skipped heading levels |
| `img-alt-missing` | info | v0.4 | `<img>` without `alt` in main content |
| `lastmod-future` | error | v0.3 | Sitemap `lastmod` in the future |
| `lastmod-uniform` | warn | v0.3 | ≥ 90% of sitemap entries have the same lastmod (build-time stamping) |
| `lastmod-mismatch` | warn | v0.4 | Sitemap lastmod ≠ JSON-LD `dateModified` / `article:modified_time` (beyond 1 day) |
| `jsonld-date-missing` | info | v0.4 | Article without `dateModified` |
| `jsonld-author` | warn | v0.4 | Article `author` missing, or a string instead of Person/Organization with `name` (+ `url`) |
| `jsonld-visible-mismatch` | warn | v0.4 | JSON-LD `headline`/`name` absent from visible h1/title; Product price absent from text |
| `jsonld-entity-id` | info | v0.4 | Organization/Person without a stable `@id`, or different `@id`s across pages |
| `sameas-inconsistent` | warn | v0.4 | Organization `sameAs` differs between pages, or a `sameAs` URL is non-200 |
| `jsonld-deprecated-type` | info | v0.4 | HowTo present; FAQPage on a non-gov/health site (no rich result expected) |
| `md-alternate-broken` | error | v0.5 | `rel=alternate type=text/markdown` target non-200 or wrong content-type |
| `llms-txt-invalid` | warn | v0.5 | `/llms.txt` present but not in the spec shape (H1, links) or links non-200 (only checked if present; absence is never reported) |
| `indexnow-key-missing` | warn | v0.5 | IndexNow configured but key file not served |

---

## 5. Positioning updates

1. Add one sentence to the positioning line: *"…and makes your site legible to AI search: purpose-split AI crawler rules, an entity graph, and checks that what bots see without JavaScript is what users see."* This stays evidence-based, with no "rank in ChatGPT" claim.
2. Rename the README comparison to include **Nuxt SEO** and **@astrojs/sitemap**. Leaving out the best-in-class benchmark (Nuxt SEO, roughly 124K/wk for the meta-module and 306K/wk for its sitemap module) makes the table look selective. State plainly: "next-herald aims at Nuxt SEO parity for Next.js."
3. Lead with the checker as a **headless-free Lighthouse-SEO alternative for CI** (lighthouse 5.4M/wk, unlighthouse 41K/wk, both needing Chrome).
4. Add an "AEO stance" section to research.md: what works (crawlable server HTML, search-bot access, accurate dates, entity markup, Bing/IndexNow for ChatGPT/Copilot), what is unproven (llms.txt, markdown alternates), and what is hype (speakable, GEO scores).
5. Fix the Google AI framing: AI Overviews/AI Mode = Googlebot + normal Search eligibility; Google-Extended = Gemini training/grounding only.
6. Keep "no ranking claims". Extend it explicitly to "no AI-citation claims".

## Notes / caveats
- The Vercel AI-crawler JS finding comes from a Dec 2024 study. Gemini (via Googlebot) and Applebot render JS. Re-test before quoting it in docs.
- I could not verify a working URL for Bing Webmaster Tools' AI Performance report (two candidate URLs returned 404). rankmath.com/features/ also returned 404.
- Items marked *(recall)* need a fresh fetch before they go into the repo.

---

## Verification addendum (2026-10-14)

Every claim the review marked *(recall)* or unverified was checked against a live source before it went into the docs.

| Claim | Result | Source |
|---|---|---|
| Google: no special AI files or markup needed | **Confirmed.** Exact wording: "You don't need to create new machine readable files, AI text files, or markup to appear in these features. There's also no special schema.org structured data that you need to add." | [Google: AI features and your website](https://developers.google.com/search/docs/appearance/ai-features) |
| Google-Extended does not cover AI Overviews/AI Mode | **Confirmed.** "robots.txt directives for Googlebot is the control"; limit what is shown with `nosnippet`, `data-nosnippet`, `max-snippet`, `noindex`; Google-Extended limits "AI training and grounding in some of Google's other systems". | same page; [Google common crawlers](https://developers.google.com/search/docs/crawling-indexing/google-common-crawlers) |
| Mueller: no AI system uses llms.txt; compared to keywords meta tag | **Confirmed, with softer wording.** His words: "AFAIK none of the AI services have said they're using LLMs.TXT … it's comparable to the keywords meta tag". Docs quote this, not "no AI system uses it". Reported by Search Engine Journal (secondary source; the original was a Reddit reply). | [SEJ](https://www.searchenginejournal.com/google-says-llms-txt-comparable-to-keywords-meta-tag/544804/), [SEJ follow-up](https://www.searchenginejournal.com/googles-mueller-says-llms-txt-cant-help-llms-differentiate-sites/579304/) |
| Nuxt SEO supports content signals | **Confirmed.** Nuxt Robots supports `Content-Signal` and IETF `Content-Usage` lines in robots.txt groups. | [Nuxt Robots: AI directives](https://nuxtseo.com/docs/robots/guides/ai-directives) |
| Yoast AI-crawler toggles | **Confirmed (Premium).** Yoast SEO Premium "Bot Blocker" toggles for GPTBot, CCBot and Google-Extended. | [Yoast Bot Blocker](https://yoast.com/features/bot-blocker/) |
| Rank Math AI-crawler toggle | **Dropped.** No first-party Rank Math page found; not claimed in the docs. | — |
| Yoast / Rank Math IndexNow | **Corrected.** Rank Math: IndexNow in the **free** plugin (Instant Indexing module). Yoast: IndexNow in **Premium**. | [Rank Math IndexNow](https://rankmath.com/kb/how-to-use-indexnow/), [Yoast IndexNow](https://yoast.com/what-is-indexnow/) |
| Nuxt has no IndexNow | **Not verified; dropped** from the docs comparison. | — |
| `Claude-User` agent name | **Confirmed**, alongside ClaudeBot and Claude-SearchBot. | [Anthropic crawlers](https://support.anthropic.com/en/articles/8896518-does-anthropic-crawl-data-from-the-web-and-how-can-site-owners-block-the-crawler) |
| Bing Webmaster Tools AI Performance report | **Confirmed.** Public preview since 10 Feb 2026; citations across Copilot, Bing AI summaries and select partners; not clicks or rankings; not in the API yet. | [Bing blog](https://blogs.bing.com/webmaster/February-2026/Introducing-AI-Performance-in-Bing-Webmaster-Tools-Public-Preview), [Bing help](https://www.bing.com/webmasters/help/ai-performance-9f8e7d6c) |
| `rankmath.com/features/` | **Still 404; not cited.** | — |
| "Bing grounds ChatGPT search" (item 6) | **Not verified from a first-party source; dropped.** The docs say IndexNow feeds Bing and other IndexNow engines, and that Bing's AI Performance report covers Copilot. | — |
| Vercel: major AI crawlers don't render JS | **Confirmed as published** ("none of the major AI crawlers currently render JavaScript"), dated Dec 2024; docs carry the date and a re-test note. | [Vercel](https://vercel.com/blog/the-rise-of-the-ai-crawler) |
| OpenAI: OAI-SearchBot opt-out removes ChatGPT search answers | **Confirmed** ("will not be shown in ChatGPT search answers, though can still appear as navigational links"). | [OpenAI crawlers](https://platform.openai.com/docs/bots) |
| `utm_source=chatgpt.com` on ChatGPT links (item 13) | **Not verified on a first-party page** (OpenAI help article did not load); docs call it "widely reported" with no link. | — |

Not scheduled: item 9 (Content-Signal passthrough) was not part of the approved list. It is recorded under *Considered* in [AEO stance](aeo.md).
