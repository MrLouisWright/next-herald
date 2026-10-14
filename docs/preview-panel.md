# SEO preview panel (dev only, v0.3)

> **Status: planned / pre-release.** This page documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](../ROADMAP.md).

`<HeraldPreview />` is a development overlay that shows how the current page will look as a Google result, an Open Graph / X card and a chat link preview, with live `herald check` warnings ([#57](https://github.com/MrLouisWright/next-herald/issues/57)). It renders nothing in production builds.

```tsx
// app/layout.tsx (excerpt)
import { HeraldPreview } from 'next-herald/preview';

export default function RootLayout({ children }: LayoutProps<'/'>) {
  return (
    <html lang="en">
      <body>
        {children}
        <HeraldPreview />
      </body>
    </html>
  );
}
```

| Prop | Type | Default | Notes |
|---|---|---|---|
| `position` | `'bottom-right' \| 'bottom-left'` | `'bottom-right'` | Toggle button position. |
| `defaultOpen` | `boolean` | `false` | Open the panel on load. |
| `rules` | `CheckConfig['rules']` | from `check.rules` | Severity overrides for live warnings. |

## What it shows
- **Google result:** title, URL breadcrumb and description, truncated at documented widths. An approximation: Google rewrites titles and snippets.
- **Social cards:** Open Graph (`summary_large_image` and `summary`) and X, from the rendered `og:*` / `twitter:*` tags and images.
- **Chat link preview:** generic title, description, image and site name card as shown by chat apps.
- **Live warnings:** the page-level rules from `checkHtml` ([Testing](testing.md)) run against the current DOM and update on client navigation. Site-level rules (duplicates, sitemaps, redirects) need `herald check` ([CLI](cli.md)).

## Production and Cache Components safety
- Client Component; returns `null` unless `process.env.NODE_ENV === 'development'`, so the bundler removes it from production builds. Client JS added in production: 0 (asserted in e2e).
- Reads the rendered DOM in the browser only. It never calls `headers()`, `cookies()` or any request API ([ADR 0006](adr/0006-cache-components-safety.md)), so the layout stays in the static shell.
- No dependencies beyond `next` and `react` ([ADR 0002](adr/0002-zero-dependencies.md)).
