# Robots

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

`defineRobots(config, options)` returns `MetadataRoute.Robots` for [`app/robots.ts`](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/robots).

```ts
export default defineRobots(herald, {
  rules: [{ userAgent: '*', allow: '/', disallow: ['/admin', '/api/'] }],
  sitemaps: 'auto',            // '/sitemap.xml' (+ '/news-sitemap.xml' if configured), absolute
  aiCrawlers: 'allow',          // or 'disallow': adds rules for a documented, dated list of AI user agents
});
```

- `indexing: 'off'` (previews) produces `Disallow: /` for all agents, regardless of `rules`.
- `host` is omitted (non-standard; ignored by Google).
- Paths are validated per [RFC 9309](https://www.rfc-editor.org/rfc/rfc9309) (must start with `/` or `*`).
- The AI-crawler list is data in the package, updated in minors; it is **not** a guarantee those bots obey robots.txt.

Remember: robots.txt controls crawling, not indexing. A page blocked in robots.txt can still be indexed from links, and Google cannot see its `noindex`. `herald check` reports `robots-blocks-noindex` for that combination and `robots-blocks-sitemap-url` when a sitemap URL is disallowed.
