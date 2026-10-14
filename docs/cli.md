# CLI and `herald check` rules

> **Status: planned / pre-release.** This page documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](../ROADMAP.md).

The `herald` binary uses Node built-ins only ([ADR 0002](adr/0002-zero-dependencies.md)).

| Command | Since | Purpose |
|---|---|---|
| `herald init [--large] [--i18n]` | v0.5 | Scaffold `herald.config.ts`, `app/sitemap.ts`, `app/robots.ts` (`--large`: index + chunks) |
| `herald routes [--watch] [--git-lastmod] [--images]` | v0.2 (`--git-lastmod` v0.3, `--images` v0.4) | Scan `app/`, write `herald-routes.gen.ts`; `--git-lastmod` adds each route file's last commit date at build time ([#50](https://github.com/MrLouisWright/next-herald/issues/50)); `--images` adds `next/image` sources for image sitemaps ([Image SEO](images.md), [#60](https://github.com/MrLouisWright/next-herald/issues/60)) |
| `herald build` | v0.2 | `routes` + config validation; use in `prebuild` (fails on redirect loops from v0.3, [#58](https://github.com/MrLouisWright/next-herald/issues/58)) |
| `herald check` | v0.2 | Crawl and report |
| `herald check --diff <base>` | v0.4 | Report only SEO changes against a snapshot or git ref ([Diff mode](check-diff.md), [#61](https://github.com/MrLouisWright/next-herald/issues/61)) |
| `herald indexnow [--changed --previous <file>] [--urls <file>] [--dry-run] [--init-key]` | v0.5 | Submit changed sitemap URLs to IndexNow ([#53](https://github.com/MrLouisWright/next-herald/issues/53)) |

## `herald check`
```bash
herald check --url http://localhost:3000        # running server
herald check --dir out                          # static export
herald check --build                            # next build && next start on a free port, then crawl
```
Options: `--seed /path` (repeatable), `--max-pages 5000`, `--concurrency 8`, `--timeout 10000`, `--format pretty|json|github|sarif|markdown` (`markdown` v0.4), `--env production|preview`, `--rule id=off|info|warn|error`, `--ignore '/drafts/**'`, `--ignore-robots`, `--ua bot|browser|<string>` (default `bot`, a UA matched by Next's `htmlLimitedBots`), `--site-url` (when crawling a preview URL for a production config: rewrites canonical/hreflang host comparison), `--snapshot <file>` and `--diff <snapshot|git-ref>` (v0.4, [diff mode](check-diff.md)).

Seeds: `/`, `robots.txt` → sitemaps (index → chunks), `--seed`s, then same-origin links. It reads server HTML only, never `.next/` ([ADR 0008](adr/0008-check-crawls-http.md)). Exit code: 1 if any `error`, 2 on crawl failure (e.g. server unreachable), else 0.

## `herald indexnow` (v0.5)
CLI only; nothing runs in your app at request time. It diffs the current sitemap(s) against a snapshot from the previous build (`--previous`), and POSTs added or changed URLs to `https://api.indexnow.org/indexnow` with global `fetch`, in batches of at most 10,000 URLs ([IndexNow](https://www.indexnow.org/documentation)). The key comes from `indexNow.key` or `HERALD_INDEXNOW_KEY`. `--init-key` writes `public/<key>.txt` so the key file is a static asset. IndexNow is used by Bing, Yandex, Seznam, Naver and others; **Google does not use it**. `indexnow-key-missing` checks that the key file is served.

```bash
herald indexnow --changed --previous .herald/sitemap-prev.json   # after deploy
```

## Rules
Severity is the default; override in `check.rules` config or `--rule`. IDs are stable from v0.9.

| ID | Severity | Since | What it reports |
|---|---|---|---|
| `title-missing` | error | v0.2 | No `<title>` or empty title in server HTML |
| `title-duplicate` | warn | v0.2 | Same title on ≥ 2 indexable pages (canonical groups counted once) |
| `title-length` | info | v0.2 | Title outside 10–70 chars (configurable) |
| `description-missing` | warn | v0.2 | No `meta name=description` |
| `description-duplicate` | warn | v0.2 | Same description on ≥ 2 indexable pages |
| `description-length` | info | v0.2 | Outside 50–160 chars |
| `canonical-missing` | warn | v0.2 | Indexable page without `link rel=canonical` |
| `canonical-multiple` | error | v0.2 | More than one canonical |
| `canonical-relative` | error | v0.2 | Canonical not absolute |
| `canonical-broken` | error | v0.2 | Canonical target returns non-200 or redirects |
| `canonical-host` | warn | v0.2 | Canonical host differs from `siteUrl` |
| `canonical-noindex` | error | v0.2 | Canonical points to a `noindex` page |
| `noindex-leak` | error | v0.2 | With `--env production`: an indexable route (in sitemap, not ignored) has `noindex` (meta or `X-Robots-Tag`) |
| `sitemap-noindex` | error | v0.2 | Sitemap lists a `noindex` URL |
| `sitemap-url-status` | error | v0.2 | Sitemap URL returns non-200 (redirects included) |
| `sitemap-non-canonical` | warn | v0.2 | Sitemap URL whose canonical points elsewhere |
| `sitemap-missing` | warn | v0.2 | No sitemap reachable from `robots.txt` or `/sitemap.xml` |
| `sitemap-invalid` | error | v0.2 | XML not parseable / wrong namespace / relative `loc` |
| `sitemap-size` | error | v0.3 | > 50,000 URLs or > 50 MB uncompressed in one file |
| `sitemap-index-broken` | error | v0.3 | Index entry unreachable or itself an index |
| `sitemap-orphan` | info | v0.2 | Crawled indexable page not in any sitemap |
| `robots-missing` | warn | v0.2 | No `/robots.txt` |
| `robots-blocks-sitemap-url` | error | v0.2 | robots.txt disallows a URL listed in a sitemap |
| `robots-blocks-noindex` | warn | v0.2 | Disallowed URL relies on `noindex` (never seen by crawlers) |
| `robots-sitemap-host` | warn | v0.2 | `Sitemap:` line on another host |
| `content-js-only` | warn | v0.2 | Indexable page whose server HTML has fewer than N words of body text (default 50): bots that don't run JS see an empty page ([#46](https://github.com/MrLouisWright/next-herald/issues/46)) |
| `content-bot-parity` | warn | v0.2 | Bot-UA and browser-UA HTML differ materially in title, meta, canonical or body text (catches `htmlLimitedBots` regressions) ([#46](https://github.com/MrLouisWright/next-herald/issues/46)) |
| `metadata-in-body` | warn | v0.2 | `<title>`, canonical or description streamed outside `<head>` for a crawler UA ([#46](https://github.com/MrLouisWright/next-herald/issues/46)) |
| `link-broken` | error | v0.2 | Same-origin link returns 4xx/5xx ([#47](https://github.com/MrLouisWright/next-herald/issues/47)) |
| `link-redirect` | info | v0.2 | Internal link hits a redirect ([#47](https://github.com/MrLouisWright/next-herald/issues/47)) |
| `http-status-indexable` | error | v0.2 | Page in a sitemap or linked internally returns non-200 ([#48](https://github.com/MrLouisWright/next-herald/issues/48)) |
| `viewport-missing` | warn | v0.2 | No `meta name=viewport` ([#48](https://github.com/MrLouisWright/next-herald/issues/48)) |
| `anchor-not-crawlable` | warn | v0.2 | `<a>` without `href` or with a `javascript:` href ([#48](https://github.com/MrLouisWright/next-herald/issues/48)) |
| `robots-blocks-ai-search` | warn | v0.2 | robots.txt blocks OAI-SearchBot / Claude-SearchBot / PerplexityBot (also via `*`) while `ai.search` is `allow` or unset ([#45](https://github.com/MrLouisWright/next-herald/issues/45)) |
| `robots-ai-inconsistent` | info | v0.2 | Training bots allowed but AI search bots blocked (usually the inverse of intent) ([#45](https://github.com/MrLouisWright/next-herald/issues/45)) |
| `robots-ai-list-stale` | info | v0.2 | Bundled AI user-agent table older than 180 days ([#45](https://github.com/MrLouisWright/next-herald/issues/45)) |
| `lastmod-future` | error | v0.3 | Sitemap `lastmod` in the future ([#50](https://github.com/MrLouisWright/next-herald/issues/50)) |
| `lastmod-uniform` | warn | v0.3 | ≥ 90% of sitemap entries share one `lastmod` (build-time stamping) ([#50](https://github.com/MrLouisWright/next-herald/issues/50)) |
| `lastmod-mismatch` | warn | v0.3 | Sitemap `lastmod` differs from JSON-LD `dateModified` / `article:modified_time` by > 1 day ([#50](https://github.com/MrLouisWright/next-herald/issues/50)) |
| `hreflang-code` | error | v0.3 | Invalid language/region code |
| `hreflang-self` | error | v0.3 | Page's alternates do not include itself |
| `hreflang-reciprocal` | error | v0.3 | A → B but B does not list A |
| `hreflang-target-status` | error | v0.3 | Alternate is non-200 or noindex |
| `hreflang-canonical` | warn | v0.3 | Canonical points to another locale's URL |
| `hreflang-x-default` | info | v0.3 | No `x-default` |
| `jsonld-parse` | error | v0.4 | `application/ld+json` script is not valid JSON |
| `jsonld-type` | error | v0.4 | Node without `@type` or unknown `@context` |
| `jsonld-required` | warn | v0.4 | Missing a Google-required property for a supported type |
| `jsonld-recommended` | info | v0.4 | Missing a recommended property |
| `og-image-missing` | warn | v0.4 | No `og:image` |
| `og-image-status` | error | v0.4 | `og:image` non-200 |
| `og-image-dimensions` | warn | v0.4 | Smaller than 1200×630 or aspect ratio off 1.91:1 by > 10% |
| `og-image-bytes` | warn | v0.4 | Larger than 5 MB (configurable) |
| `og-image-type` | warn | v0.4 | Not PNG/JPEG/WebP/GIF |
| `og-image-alt` | info | v0.4 | No `og:image:alt` |
| `html-lang` | warn | v0.3 | Missing `<html lang>` or it differs from the page's hreflang |
| `h1-missing` | info | v0.4 | No `<h1>` ([#51](https://github.com/MrLouisWright/next-herald/issues/51)) |
| `h1-multiple` | info | v0.4 | More than one `<h1>` ([#51](https://github.com/MrLouisWright/next-herald/issues/51)) |
| `heading-order` | info | v0.4 | Skipped heading levels ([#51](https://github.com/MrLouisWright/next-herald/issues/51)) |
| `img-alt-missing` | info | v0.4 | `<img>` without `alt` in main content; warn for images in the image sitemap ([#51](https://github.com/MrLouisWright/next-herald/issues/51), [#60](https://github.com/MrLouisWright/next-herald/issues/60)) |
| `jsonld-date-missing` | info | v0.4 | Article without `dateModified` ([#51](https://github.com/MrLouisWright/next-herald/issues/51)) |
| `jsonld-author` | warn | v0.4 | Article `author` missing, or a string instead of Person/Organization with `name` ([#51](https://github.com/MrLouisWright/next-herald/issues/51)) |
| `jsonld-visible-mismatch` | warn | v0.4 | JSON-LD `headline`/`name` absent from visible h1/title; Product price absent from text ([#51](https://github.com/MrLouisWright/next-herald/issues/51)) |
| `jsonld-entity-id` | info | v0.4 | Organization/Person without a stable `@id`, or different `@id`s across pages ([#52](https://github.com/MrLouisWright/next-herald/issues/52)) |
| `sameas-inconsistent` | warn | v0.4 | Organization `sameAs` differs between pages, or a `sameAs` URL is non-200 ([#52](https://github.com/MrLouisWright/next-herald/issues/52)) |
| `jsonld-deprecated-type` | info | v0.4 | HowTo present; FAQPage on a site that gets no FAQ rich result ([#52](https://github.com/MrLouisWright/next-herald/issues/52)) |
| `md-alternate-broken` | error | v0.5 | `rel=alternate type=text/markdown` target non-200 or wrong content type (experimental, [#54](https://github.com/MrLouisWright/next-herald/issues/54)) |
| `llms-txt-invalid` | warn | v0.5 | `/llms.txt` present but not in the llmstxt.org shape, or its links non-200. Absence is never reported (experimental, [#55](https://github.com/MrLouisWright/next-herald/issues/55)) |
| `indexnow-key-missing` | warn | v0.5 | IndexNow configured but the key file is not served ([#53](https://github.com/MrLouisWright/next-herald/issues/53)) |
| `redirect-chain` | warn | v0.3 | More than one redirect hop before a 200 ([Redirects](redirects.md), [#58](https://github.com/MrLouisWright/next-herald/issues/58)) |
| `redirect-loop` | error | v0.3 | Redirects that never reach a final response ([#58](https://github.com/MrLouisWright/next-herald/issues/58)) |
| `redirect-temporary` | warn | v0.3 | 302/307 where a permanent redirect is intended: rule declared `permanent`, or URL in a sitemap or linked internally ([#58](https://github.com/MrLouisWright/next-herald/issues/58)) |
| `params-duplicate-content` | warn | v0.3 | ≥ 2 indexable URLs that differ only in search params with the same title and body text, each self-canonical ([Pagination](pagination.md), [#59](https://github.com/MrLouisWright/next-herald/issues/59)) |
| `pagination-canonical-first` | warn | v0.3 | Page 2+ of a listing canonicalises to page 1 ([#59](https://github.com/MrLouisWright/next-herald/issues/59)) |
| `img-dimensions-missing` | warn | v0.4 | `<img>` without `width` and `height` ([Image SEO](images.md), [#60](https://github.com/MrLouisWright/next-herald/issues/60)) |
| `img-format-legacy` | info | v0.4 | JPEG/PNG/GIF over 100 KB (configurable) without a WebP/AVIF alternative ([#60](https://github.com/MrLouisWright/next-herald/issues/60)) |
| `sitemap-image-status` | error | v0.4 | Image sitemap URL non-200 or not an image content type ([#60](https://github.com/MrLouisWright/next-herald/issues/60)) |
| `diff-noindex-added` | error | v0.4 | `--diff` only: URL indexable in base is `noindex` now ([Diff mode](check-diff.md), [#61](https://github.com/MrLouisWright/next-herald/issues/61)) |
| `diff-canonical-changed` | warn | v0.4 | `--diff` only: canonical target changed ([#61](https://github.com/MrLouisWright/next-herald/issues/61)) |
| `diff-sitemap-url-removed` | warn | v0.4 | `--diff` only: URL in base sitemap missing now ([#61](https://github.com/MrLouisWright/next-herald/issues/61)) |
| `diff-status-changed` | warn | v0.4 | `--diff` only: HTTP status changed ([#61](https://github.com/MrLouisWright/next-herald/issues/61)) |
| `diff-title-changed` | info | v0.4 | `--diff` only: title changed ([#61](https://github.com/MrLouisWright/next-herald/issues/61)) |
| `diff-description-changed` | info | v0.4 | `--diff` only: description changed ([#61](https://github.com/MrLouisWright/next-herald/issues/61)) |

These 27 rules (from `content-js-only` to `lastmod-mismatch` and `h1-missing` to `indexnow-key-missing`) were added by the [SEO/AEO review](seo-aeo-review.md). The structure rules are consistency checks, not ranking or citation factors. The 14 rules from `redirect-chain` to `diff-description-changed` belong to the approved features #58–#61; the `diff-*` rules run only with `--diff`.

Limits: no JavaScript execution, on purpose: content injected client-side is invisible, as it is for most AI crawlers ([Vercel, Dec 2024](https://vercel.com/blog/the-rise-of-the-ai-crawler)). `content-js-only` and `content-bot-parity` make that visible. Image dimensions are read from file headers; animated/complex formats may be reported as `og-image-type`.

## GitHub Actions
```yaml
- run: npx herald check --build --format github --env production
```
