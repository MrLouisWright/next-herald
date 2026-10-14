# 0011. OG image templates are plain JSX on `next/og`

- Status: Accepted
- Date: 2026-10-14

## Context
`next/og` `ImageResponse` (Satori + Resvg) supports a CSS subset (flexbox, no grid) and needs font data for non-default fonts. Shipping fonts would bloat the package.

## Decision
- `next-herald/og` exports template components (`ArticleCard`, `ProductCard`, `SimpleCard`) and `ogImage(template, props, options)` returning `new ImageResponse(...)` sized 1200×630 by default.
- **No bundled fonts.** Users pass `fonts` (read with `readFile` or `fetch`) — Next's default font is used otherwise. Templates only use the Satori-supported CSS subset and are snapshot-tested as PNG on CI.
- Text is truncated with a measured-length heuristic; RTL and CJK need a user font (documented).

## Consequences
\+ Small package, full control. − Users manage fonts.

## Alternatives considered
A hosted OG service or Puppeteer rendering: rejected (deps, cost).
