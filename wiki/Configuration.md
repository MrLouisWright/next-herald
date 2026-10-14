# Configuration

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

`herald.config.ts` default-exports `defineHerald(options)`. App code imports it like any module. The CLI loads it with Node's built-in type stripping (Node ≥ 22.18 / ≥ 23.6, erasable syntax only); on older Node use `herald.config.mjs` ([ADR 0009](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0009-cli-config-and-route-scan.md)).

| Option | Type | Default | Notes |
|---|---|---|---|
| `siteUrl` | `string` | — (required) | Absolute origin, `https:` (or `http://localhost`). No path/query/hash. Becomes `metadataBase`. Per environment: `process.env.HERALD_SITE_URL` read at **build** time, never from headers. |
| `siteName` | `string` | — | `og:site_name`, `WebSite` JSON-LD. |
| `title` | `{ default, template }` | — | `template` must contain `%s`; validated. |
| `description` | `string` | — | Default description; `herald check` warns on duplicates anyway. |
| `locale` | `string` | `'en'` | `og:locale` (BCP 47, converted to `en_US`-style for OG). |
| `openGraph` | `Metadata['openGraph']` | `{}` | Merged into every page. |
| `twitter` | `{ site?, creator?, card? }` | `card: 'summary_large_image'` | |
| `trailingSlash` | `boolean` | from `next.config` value passed here | Must match `next.config`; `herald check` flags canonicals that redirect. |
| `i18n` | `I18nSource` | — | See [hreflang & i18n](hreflang-and-i18n). |
| `indexing` | `'auto' \| 'on' \| 'off'` | `'auto'` | `auto`: `off` unless `HERALD_ENV=production` (or `VERCEL_ENV=production`), evaluated at build. When off, every page gets `robots: noindex` and `robots.txt` disallows all. |
| `organization` | `{ name, url?, logo?, sameAs?: string[] }` | — | Publisher entity for the default graph (`${siteUrl}/#organization`). Exclusive with `person`. v0.2 ([JSON-LD](JSON-LD#entity-graph-by-default-v02)). |
| `person` | `{ name, url?, image?, sameAs?: string[] }` | — | Personal-site entity (`${siteUrl}/#person`). Exclusive with `organization`. v0.2. |
| `indexNow` | `{ key?: string; keyPath?: string; endpoint?: string }` | — | Used by `herald indexnow` only (CLI, v0.5). `key` falls back to `HERALD_INDEXNOW_KEY`; the key is public by design (served as a file). |
| `app` | `{ dir?, pageExtensions? }` | `{ dir: 'app' }` | Used by `herald routes`. Supports `src/app`. |
| `check` | `CheckConfig` | — | Rule severities, thresholds, ignore globs. See [CLI](CLI-and-Check-Rules). |

AI crawler policy is set on `defineRobots({ ai })`, not here ([Robots](Robots#ai-crawlers-by-purpose)).

Validation errors are thrown at import time with the option path, e.g. `herald: siteUrl must be absolute (got "/").`

> The `indexing: 'auto'` default prevents preview deploys from being indexed. `herald check --env production` also verifies that **production** has no `noindex` leak (rule `noindex-leak`).
