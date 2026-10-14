# 0008. `herald check` crawls HTTP or static output, not Next internals

- Status: Accepted
- Date: 2026-10-14

## Context
`.next/` manifests (`app-paths-manifest.json`, prerender manifests) are internal and change between minors. Tests must reflect what crawlers see.

## Decision
- `herald check` has two inputs:
  - `--url <origin>`: crawls a running server (`next start`, preview deploy) with global `fetch`, seeded from `/robots.txt` sitemaps, the sitemap(s) and `--seed` paths, then following same-origin `<a href>`s up to `--max-pages` (default 5,000) with `--concurrency` (default 8);
  - `--dir out/`: reads a static export from disk (no server).
- It never reads `.next/`. It sees server HTML only; client-only metadata changes are out of scope.
- Rules have stable IDs and default severities ([CLI](../cli.md)); output formats `pretty`, `json`, `github` (annotations), `sarif`(v0.4). Exit code 1 on any `error`.
- Polite by default: honours `robots.txt` for the `herald` user agent unless `--ignore-robots`, sends `User-Agent: next-herald-check/<version>`.

## Consequences
\+ Version-stable, works against any deploy. − Needs a running server or export; `herald check --build` runs `next build && next start` on a free port as a convenience.

## Alternatives considered
Reading `.next` manifests: rejected (unstable). Headless browser: rejected (ADR 0002, slow).
