# AEO stance: answer engines and AI search

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

next-herald makes **no ranking or AI-citation claims**. It helps your site stay *legible* to search and answer engines: AI crawler rules split by purpose, an entity graph, accurate dates, and checks that what bots see without JavaScript is what users see. Everything here is based on what engines document publicly. Sources were checked on 2026-10-14 ([review and verification](SEO-AEO-Review), [ADR 0012](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0012-ai-crawler-policy-and-aeo-stance.md)).

## What works (documented by the engines)

| Practice | Why | next-herald |
|---|---|---|
| Crawlable **server HTML** | Most AI crawlers do not run JavaScript ([Vercel, Dec 2024](https://vercel.com/blog/the-rise-of-the-ai-crawler); re-test before relying on it). Google: "Making sure that important content is available in textual form". | `content-js-only`, `content-bot-parity`, `metadata-in-body` ([#46](https://github.com/MrLouisWright/next-herald/issues/46), v0.2) |
| Let **AI search** bots in, even if you block training | OpenAI: "Sites that are opted out of OAI-SearchBot will not be shown in ChatGPT search answers" ([OpenAI](https://platform.openai.com/docs/bots)). Anthropic and Perplexity document separate search bots too. | `defineRobots({ ai })` ([#11](https://github.com/MrLouisWright/next-herald/issues/11), v0.1), `robots-blocks-ai-search` ([#45](https://github.com/MrLouisWright/next-herald/issues/45), v0.2) |
| Normal **Google Search** eligibility | "There are no additional requirements to appear in AI Overviews or AI Mode" ([Google](https://developers.google.com/search/docs/appearance/ai-features)). | All v0.2 checks |
| **Accurate dates** | Google uses `lastmod` only if it is "consistently and verifiably accurate" ([Google](https://developers.google.com/search/docs/crawling-indexing/sitemaps/build-sitemap)). | `--git-lastmod`, `lastmod-*` rules ([#50](https://github.com/MrLouisWright/next-herald/issues/50), v0.3) |
| **Entity markup** consistent with visible content | Google: "Making sure your structured data matches the visible text on the page"; Article author best practices ([Google](https://developers.google.com/search/docs/appearance/structured-data/article)). | Entity graph by default ([#49](https://github.com/MrLouisWright/next-herald/issues/49), v0.2), structure and entity checks ([#51](https://github.com/MrLouisWright/next-herald/issues/51), [#52](https://github.com/MrLouisWright/next-herald/issues/52), v0.4) |
| **Fast discovery** on Bing and other IndexNow engines | IndexNow is supported by Bing, Yandex, Seznam, Naver and others ([IndexNow](https://www.indexnow.org/documentation)); Google does not use it. | `herald indexnow` ([#53](https://github.com/MrLouisWright/next-herald/issues/53), v0.5) |

## Google: AI Overviews, AI Mode and Google-Extended

- AI Overviews and AI Mode are part of Search. They use **Googlebot** and normal Search eligibility; a page must be "indexed and eligible to be shown in Google Search with a snippet".
- **Google-Extended does not control AI Overviews or AI Mode.** It limits "AI training and grounding in some of Google's other systems" (Gemini models and apps, Vertex AI). Blocking it does not keep you out of AI Overviews and does not affect Search ranking.
- To limit what Search (including its AI features) shows from a page, use `nosnippet`, `data-nosnippet`, `max-snippet` or `noindex` ([robots meta tag](https://developers.google.com/search/docs/crawling-indexing/robots-meta-tag)). In next-herald: `pageMetadata({ extra: { robots: { 'max-snippet': 50 } } })` or `noindex: true`.
- Google: "You don't need to create new machine readable files, AI text files, or markup to appear in these features."

Sources: [AI features and your website](https://developers.google.com/search/docs/appearance/ai-features), [Google common crawlers](https://developers.google.com/search/docs/crawling-indexing/google-common-crawlers).

## Unproven: shipped only as experimental opt-ins (v0.5)

| Feature | Honest note |
|---|---|
| **llms.txt / llms-full.txt** ([#55](https://github.com/MrLouisWright/next-herald/issues/55)) | **No proven search or AI-citation benefit.** A proposal ([llmstxt.org](https://llmstxt.org/)) with no standards body. No major AI search engine has documented using it. Google's John Mueller: "AFAIK none of the AI services have said they're using LLMs.TXT … it's comparable to the keywords meta tag" ([SEJ](https://www.searchenginejournal.com/google-says-llms-txt-comparable-to-keywords-meta-tag/544804/)). Real consumers are coding agents and doc tools. Off by default; generated only if you add the route. |
| **Markdown alternates** ([#54](https://github.com/MrLouisWright/next-herald/issues/54)) | Cloudflare and Vercel serve markdown to agents ([Cloudflare](https://blog.cloudflare.com/markdown-for-agents/), [Vercel](https://vercel.com/blog/making-agent-friendly-pages-with-content-negotiation)). No engine has confirmed ranking or citation use. Static `.md` routes only; Accept negotiation lives in a `proxy.ts` recipe (below), never in library code. |

### Recipe: Accept negotiation in `proxy.ts` (experimental)

Library code never reads request headers ([ADR 0006](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0006-cache-components-safety.md)). If you want `Accept: text/markdown` to return markdown, do it in your own proxy and rewrite to the static markdown route:

```ts
// proxy.ts (your app, not next-herald)
import { NextResponse, type NextRequest } from 'next/server';

export function proxy(req: NextRequest) {
  const accept = req.headers.get('accept') ?? '';
  if (accept.includes('text/markdown') && !accept.includes('text/html')) {
    const url = req.nextUrl.clone();
    url.pathname = `${url.pathname.replace(/\/$/, '')}/md`; // static route from markdownRoute()
    const res = NextResponse.rewrite(url);
    res.headers.set('Vary', 'Accept');
    return res;
  }
}

export const config = { matcher: ['/blog/:slug*'] };
```

Pages stay in the static shell; only the proxy runs per request. Check that your CDN respects `Vary: Accept`.

## Measuring AI referrals and citations (docs, [#56](https://github.com/MrLouisWright/next-herald/issues/56))

- **Referrers:** classify `document.referrer` / log referrers for `chatgpt.com`, `perplexity.ai`, `copilot.microsoft.com`, `gemini.google.com`, `claude.ai`, and `utm_source=chatgpt.com` (widely reported on ChatGPT links; we could not confirm it on a first-party page on 2026-10-14, so confirm it in your own analytics).
- **Google:** AI Overviews and AI Mode clicks are counted in Search Console Performance under the "Web" search type; there is no separate report ([Google](https://developers.google.com/search/docs/appearance/ai-features)).
- **Bing:** Bing Webmaster Tools **AI Performance** (public preview since 10 Feb 2026) shows citations across Microsoft Copilot, Bing AI summaries and select partners. It counts citations, not clicks or rankings ([Bing blog](https://blogs.bing.com/webmaster/February-2026/Introducing-AI-Performance-in-Bing-Webmaster-Tools-Public-Preview), [help](https://www.bing.com/webmasters/help/ai-performance-9f8e7d6c)).
- **Bot hits:** count requests in access logs by the user agents in the purpose table ([Robots](Robots#ai-crawlers-by-purpose)); verify IPs where vendors publish ranges.

## Hype: rejected

See [Research → rejected / hype](Research#aeo-rejected-and-hype): llms.txt as a ranking lever, "GEO scores", `speakable`, HowTo, the Indexing API for ordinary pages, blocking Google-Extended to avoid AI Overviews, Accept negotiation in library code, AI-generated FAQ blocks, pay-per-crawl, robots.txt as enforcement, headless "see what AI sees".

## Considered, not scheduled

- **Content-Signal / Content-Usage lines** in robots.txt (Cloudflare's [content signals](https://blog.cloudflare.com/content-signals-policy/), IETF aipref; supported by [Nuxt Robots](https://nuxtseo.com/docs/robots/guides/ai-directives)). Preference signals only, voluntary compliance. Not in the approved set; revisit after v0.5. Until then, add the lines through your own `robots.ts` post-processing.
