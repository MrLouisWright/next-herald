# 0002. Zero dependencies beyond `next` and `react`

- Status: Accepted
- Date: 2026-10-14

## Context
SEO kits tend to pull in HTML parsers, crawlers (Puppeteer/Playwright), glob libraries, XML builders, CLI frameworks and schema type packages. Each one adds supply-chain risk and upgrade work for a solo maintainer. `next-sitemap` 4.2.3 has four runtime dependencies, including `@next/env@^13`. Same rule as [next-tongue ADR 0012](https://github.com/MrLouisWright/next-tongue/blob/main/docs/adr/0012-zero-dependencies.md).

## Decision
- The published `next-herald` package has **no `dependencies`** and no optional or bundled dependencies. Its only `peerDependencies` are `next` and `react`.
- Only Node built-ins and web platform APIs are used:
  - CLI: `node:util` `parseArgs`, `fs`, `path`, `url`, `zlib` (gzipped sitemaps), global `fetch`, `AbortController`; no CLI framework;
  - crawling: own tolerant `<head>` scanner (meta, link, title, JSON-LD scripts) — not a full HTML parser, with documented limits;
  - XML: own small sitemap/robots writer and reader for the subset we emit and check;
  - image size: PNG/JPEG/WebP/GIF header readers (a few dozen lines each);
  - schema.org types: own curated, hand-maintained type subset ([ADR 0005](0005-own-schema-types.md)); no `schema-dts`.
- Repo devDependencies (test runners, linters) are allowed but never appear in the published dependency fields.
- **CI dependency budget check** ([#3](https://github.com/MrLouisWright/next-herald/issues/3)): fails if `dependencies`, `optionalDependencies` or `bundleDependencies` are non-empty, or `peerDependencies` contains anything besides `next` and `react`. It runs on the packed tarball. A test asserts that source imports only `node:` built-ins, `next`, `next/*`, `react` and internal modules.

## Consequences
- \+ `npm install next-herald` installs nothing else.
- − We own the head scanner, XML writer and image readers; each has a fuzz/fixture corpus.
- − `herald check` sees server-rendered HTML only. It does not execute client JS (documented).

## Alternatives considered
`schema-dts` as a dependency or optional peer (rejected; types stay structurally compatible instead); `htmlparser2`/`parse5` (rejected); Puppeteer/Playwright crawling (rejected, see [research](../research.md#rejected-ideas)).
