# 0009. Config loading and the route manifest

- Status: Accepted
- Date: 2026-10-14

## Context
The app imports config through Next's bundler, but the CLI runs in plain Node with no TypeScript loader (ADR 0002). Generating a sitemap "from the app/ route manifest" needs the list of routes without reading `.next/` (ADR 0008).

## Decision
- Config lives in `herald.config.ts` via `defineHerald()` and is imported by app code normally. The CLI loads it with Node's built-in type stripping (Node ≥ 22.18 or ≥ 23.6; erasable syntax only — no enums/namespaces). On older Node, `herald.config.mjs`/`.json` are accepted; the CLI prints a clear message otherwise.
- `herald routes` (also run by `herald build`) scans `app/` with `fs`: route groups `(x)` removed, parallel `@slot` and intercepting `(.)` segments ignored, private `_folders` skipped, `page.(tsx|ts|jsx|js|mdx)` (respecting `pageExtensions` passed in config) detected. It writes `herald-routes.gen.ts`: a typed list of static paths and dynamic patterns (`/blog/[slug]`).
- `defineSitemap({ routes, params })` requires a `params` enumerator for every dynamic pattern (type error otherwise); static routes are included automatically. `exclude` globs are matched with an own small matcher.
- Route scanning is a convenience: a hand-written route list works the same.

## Consequences
\+ No Next internals; a type error tells you when a new dynamic route lacks params. − The generated file must be refreshed (`herald build` in `prebuild`, `herald routes --watch` via `fs.watch` in dev).

## Alternatives considered
Calling `generateStaticParams` automatically: rejected (needs executing app modules outside Next). A TS loader dependency (tsx/jiti): rejected (ADR 0002).
