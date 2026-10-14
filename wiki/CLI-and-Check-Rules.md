# CLI and `herald check` rules

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

The `herald` binary uses Node built-ins only ([ADR 0002](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0002-zero-dependencies.md)).

| Command | Since | Purpose |
|---|---|---|
| `herald init [--large] [--i18n]` | v0.5 | Scaffold `herald.config.ts`, `app/sitemap.ts`, `app/robots.ts` (`--large`: index + chunks) |
| `herald routes [--watch]` | v0.2 | Scan `app/`, write `herald-routes.gen.ts` |
| `herald build` | v0.2 | `routes` + config validation; use in `prebuild` |
| `herald check` | v0.2 | Crawl and report |

## `herald check`
```bash
herald check --url http://localhost:3000        # running server
herald check --dir out                          # static export
herald check --build                            # next build && next start on a free port, then crawl
```
Options: `--seed /path` (repeatable), `--max-pages 5000`, `--concurrency 8`, `--timeout 10000`, `--format pretty|json|github|sarif`, `--env production|preview`, `--rule id=off|info|warn|error`, `--ignore '/drafts/**'`, `--ignore-robots`, `--ua bot|browser|<string>` (default `bot`, a UA matched by Next's `htmlLimitedBots`), `--site-url` (when crawling a preview URL for a production config: rewrites canonical/hreflang host comparison).

Seeds: `/`, `robots.txt` → sitemaps (index → chunks), `--seed`s, then same-origin links. It reads server HTML only, never `.next/` ([ADR 0008](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0008-check-crawls-http.md)). Exit code: 1 if any `error`, 2 on crawl failure (e.g. server unreachable), else 0.

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

Limits: no JavaScript execution; content injected client-side is invisible (as for many crawlers). Image dimensions are read from file headers; animated/complex formats may be reported as `og-image-type`.

## GitHub Actions
```yaml
- run: npx herald check --build --format github --env production
```
