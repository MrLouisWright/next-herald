# Robots

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

`defineRobots(config, options)` returns `MetadataRoute.Robots` for [`app/robots.ts`](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/robots).

```ts
export default defineRobots(herald, {
  rules: [{ userAgent: '*', allow: '/', disallow: ['/admin', '/api/'] }],
  sitemaps: 'auto',            // '/sitemap.xml' (+ '/news-sitemap.xml' if configured), absolute
  ai: { training: 'disallow' }, // search and userFetch default to 'allow'
});
```

- `indexing: 'off'` (previews) produces `Disallow: /` for all agents, regardless of `rules` and `ai`.
- `host` is omitted (non-standard; ignored by Google).
- Paths are validated per [RFC 9309](https://www.rfc-editor.org/rfc/rfc9309) (must start with `/` or `*`).
- Pure data and a static `robots.ts`; nothing is read from the request ([ADR 0006](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0006-cache-components-safety.md)).

## AI crawlers by purpose

AI vendors run different bots for different jobs, and blocking one does not block the others. `ai` sets each purpose separately ([ADR 0012](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0012-ai-crawler-policy-and-aeo-stance.md), [#11](https://github.com/MrLouisWright/next-herald/issues/11)):

```ts
ai?: {
  training: 'allow' | 'disallow';   // model training crawlers
  search?: 'allow' | 'disallow';    // AI search indexing. Default 'allow'
  userFetch?: 'allow' | 'disallow'; // fetches a user asked for. Default 'allow'
}
```

Omitting `ai` adds no AI-specific groups (your `*` rules apply). Explicit `rules` for the same user agent win over `ai`, with a dev warning.

| Purpose | User agents (table dated 2026-10-14) | Effect of `disallow` (vendor wording) |
|---|---|---|
| `training` | `GPTBot`, `ClaudeBot`, `Google-Extended`, `Applebot-Extended`, `CCBot`, `Bytespider`, `meta-externalagent` | Content excluded from future model training |
| `search` | `OAI-SearchBot`, `Claude-SearchBot`, `PerplexityBot` | OpenAI: "Sites that are opted out of OAI-SearchBot will not be shown in ChatGPT search answers". Anthropic: "may reduce your site's visibility" in search results |
| `userFetch` | `ChatGPT-User`, `Claude-User`, `Perplexity-User` | The assistant cannot fetch your page when a user asks for it |

Sources: [OpenAI](https://platform.openai.com/docs/bots), [Anthropic](https://support.anthropic.com/en/articles/8896518-does-anthropic-crawl-data-from-the-web-and-how-can-site-owners-block-the-crawler), [Perplexity](https://docs.perplexity.ai/guides/bots), [Google common crawlers](https://developers.google.com/search/docs/crawling-indexing/google-common-crawlers), [Applebot-Extended](https://support.apple.com/en-us/119829), [CCBot](https://commoncrawl.org/ccbot), [Meta](https://developers.facebook.com/docs/sharing/webmasters/web-crawlers).

The table is data in the package with a date and a source URL per entry. It is updated in minors, and `herald check` reports `robots-ai-list-stale` when it is more than 180 days old ([#45](https://github.com/MrLouisWright/next-herald/issues/45)). `robots-blocks-ai-search` warns when search bots are blocked (for example by `User-agent: *` `Disallow: /`) while `ai.search` is `allow` or unset.

**Not enforcement.** robots.txt compliance is voluntary. Some crawlers ignore it; use your CDN/WAF bot management for enforcement.

### Google-Extended is not an AI Overviews opt-out

Google-Extended only controls whether content may be used for Gemini model training and grounding in Google's other systems (Gemini apps, Vertex AI). **AI Overviews and AI Mode are part of Search and use Googlebot.** Blocking Google-Extended does not remove you from them and does not affect ranking. To limit what Search (including AI features) shows, use `nosnippet`, `data-nosnippet`, `max-snippet` or `noindex` per page ([robots meta tag](https://developers.google.com/search/docs/crawling-indexing/robots-meta-tag), [AI features](https://developers.google.com/search/docs/appearance/ai-features)). Blocking Googlebot removes you from Search entirely. More in [AEO stance](AEO-Stance).

## Crawling vs indexing

robots.txt controls crawling, not indexing. A page blocked in robots.txt can still be indexed from links, and Google cannot see its `noindex`. `herald check` reports `robots-blocks-noindex` for that combination and `robots-blocks-sitemap-url` when a sitemap URL is disallowed.
