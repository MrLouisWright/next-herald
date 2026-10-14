# Architecture Decision Records

Format: [MADR](https://adr.github.io/madr/)-lite. Status values: Proposed · Accepted · Superseded. All ADRs are pre-release and may be revisited until v0.9 (API freeze).

| # | Title | Status |
|---|---|---|
| [0001](0001-build-on-native-metadata-apis.md) | Build on native Next.js metadata APIs | Accepted |
| [0002](0002-zero-dependencies.md) | Zero dependencies beyond `next` and `react` | Accepted |
| [0003](0003-nextjs-16-only.md) | Next.js 16+ App Router only; canary CI | Accepted |
| [0004](0004-safe-json-ld-serialization.md) | JSON-LD is serialized safely by default | Accepted |
| [0005](0005-own-schema-types.md) | Own curated schema.org types | Accepted |
| [0006](0006-cache-components-safety.md) | Never read request data in library code | Accepted |
| [0007](0007-sitemap-index-via-route-handlers.md) | Sitemap index and news sitemaps via route handlers | Accepted |
| [0008](0008-check-crawls-http.md) | `herald check` crawls HTTP or static output | Accepted |
| [0009](0009-cli-config-and-route-scan.md) | Config loading and the route manifest | Accepted |
| [0010](0010-optional-next-tongue-integration.md) | next-tongue integration is optional and structural | Accepted |
| [0011](0011-og-templates-on-next-og.md) | OG templates are plain JSX on `next/og` | Accepted |

New ADR: copy [template.md](template.md), next number, open a PR labelled `type: adr`.
