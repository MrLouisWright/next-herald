# Configuration

> **Status: planned / pre-release.** This page documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](../ROADMAP.md).

`herald.config.ts` default-exports `defineHerald(options)`. App code imports it like any module. The CLI loads it with Node's built-in type stripping (Node ≥ 22.18 / ≥ 23.6, erasable syntax only); on older Node use `herald.config.mjs` ([ADR 0009](adr/0009-cli-config-and-route-scan.md)).

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
| `i18n` | `I18nSource` | — | See [hreflang & i18n](i18n-hreflang.md). |
| `indexing` | `'auto' \| 'on' \| 'off'` | `'auto'` | `auto`: `off` unless `HERALD_ENV=production` (or `VERCEL_ENV=production`), evaluated at build. When off, every page gets `robots: noindex` and `robots.txt` disallows all. |
| `app` | `{ dir?, pageExtensions? }` | `{ dir: 'app' }` | Used by `herald routes`. Supports `src/app`. |
| `check` | `CheckConfig` | — | Rule severities, thresholds, ignore globs. See [CLI](cli.md). |

Validation errors are thrown at import time with the option path, e.g. `herald: siteUrl must be absolute (got "/").`

> The `indexing: 'auto'` default prevents preview deploys from being indexed. `herald check --env production` also verifies that **production** has no `noindex` leak (rule `noindex-leak`).
