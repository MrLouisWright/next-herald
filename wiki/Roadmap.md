# Roadmap

> **Status: pre-alpha.** Dates are not promised. Each milestone ships when its acceptance criteria are met. Tracked in [GitHub milestones](https://github.com/MrLouisWright/next-herald/milestones), issues and the [project board](https://github.com/users/MrLouisWright/projects/8).

> **Scope freeze (2026-10-14):** no new features are added to any milestone before v0.1 ships. v0.1 is intentionally small so it can ship fast.

**Constraints for every milestone:** zero dependencies beyond the `next` and `react` peers ([ADR 0002](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0002-zero-dependencies.md), [#3](https://github.com/MrLouisWright/next-herald/issues/3)); no request APIs in library code ([ADR 0006](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0006-cache-components-safety.md), [#7](https://github.com/MrLouisWright/next-herald/issues/7)); Next.js 16 App Router only ([ADR 0003](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0003-nextjs-16-only.md)).

## v0.1 — Core builders (ship fast)
Goal: Ship a small, useful alpha fast: everything a typical site needs in `layout.tsx`, `page.tsx`, `sitemap.ts` and `robots.ts`, with the zero-dependency and Cache Components guarantees enforced in CI from day one.

Scope: Config, metadata builders, `<JsonLd>` + 7 types, basic sitemap/robots, CI with dependency budget and canary. Small on purpose.

**Issues and acceptance criteria**

- **[#1](https://github.com/MrLouisWright/next-herald/issues/1) Repo scaffolding: pnpm workspace, TypeScript, Vitest, lint, CI, changesets**
  - [ ] `packages/next-herald` with ESM build, `exports` map for every entry point in docs/README.md, `sideEffects: false`.
  - [ ] CI runs lint, typecheck, unit tests on Node 20.9 and 22 LTS.
  - [ ] Changesets configured; npm provenance planned for publish (no publish in this issue).
- **[#2](https://github.com/MrLouisWright/next-herald/issues/2) defineHerald config with validation**
  - [ ] `siteUrl` validated (absolute, https or http://localhost, no path/query/hash).
  - [ ] `title.template` must contain `%s`; errors name the option path.
  - [ ] `indexing: 'auto'` resolves from `HERALD_ENV`/`VERCEL_ENV` at build time.
  - [ ] Config object is frozen; unit tests cover every option.
- **[#3](https://github.com/MrLouisWright/next-herald/issues/3) CI dependency budget check: no deps beyond next/react peers**
  - [ ] CI job fails if the published package.json has any `dependencies`, `optionalDependencies` or `bundleDependencies`, or `peerDependencies` other than `next` and `react`.
  - [ ] The check runs on the packed tarball (`npm pack`), not just the workspace file.
  - [ ] A test asserts source files import only `node:` built-ins, `next`, `next/*`, `react` and package-internal modules.
  - [ ] Failing output names the offending dependency and links the ADR.
- **[#4](https://github.com/MrLouisWright/next-herald/issues/4) Supported Next versions matrix and nightly next@canary job**
  - [ ] E2E runs on latest Next 16 minor and the oldest supported minor, Turbopack (default) and `--webpack`.
  - [ ] Nightly canary job opens/updates an issue labelled `upstream` on failure.
  - [ ] Peer range `next >=16.0.0 <17`, `react >=19` documented and tested.
- **[#5](https://github.com/MrLouisWright/next-herald/issues/5) Metadata builders: createMetadata and pageMetadata**
  - [ ] `createMetadata` sets `metadataBase`, title template, OG/Twitter defaults and `robots` from `indexing`.
  - [ ] `pageMetadata` sets absolute canonical and `og:url` from `path`, normalizes trailing slash, copies title/description into OG and Twitter.
  - [ ] Type tests: `path` required; `type: 'article'` enables article fields.
  - [ ] Rendered via a real Next app in e2e: tags match expectations.
- **[#6](https://github.com/MrLouisWright/next-herald/issues/6) mergeMetadata: deep merge of openGraph, twitter, robots, alternates**
  - [ ] Deep merges `openGraph`, `twitter`, `robots`, `alternates`, `other`; arrays replaced.
  - [ ] `undefined` never overrides a defined value; `null` explicitly clears.
  - [ ] Unit tests for every merge case in the docs table.
- **[#7](https://github.com/MrLouisWright/next-herald/issues/7) Cache Components safety: request-API guard and cacheComponents example**
  - [ ] CI fails if `src/` imports `headers`, `cookies`, `connection` or `draftMode` from `next/headers`/`next/server`.
  - [ ] `examples/cache-components` (`cacheComponents: true`) builds; every route is reported static/partially static in the build output.
  - [ ] Runtime entry points contain no `node:` imports (Edge-compatible) — test.
- **[#8](https://github.com/MrLouisWright/next-herald/issues/8) `<JsonLd>` component and serializeJsonLd**
  - [ ] Escapes `<`, `>`, `&`, U+2028, U+2029; `JSON.parse(output)` deep-equals input.
  - [ ] Throws in dev on functions, symbols, bigint, cycles, non-finite numbers; strips `undefined`.
  - [ ] Adds `@context` when missing; supports arrays and `@graph`; `nonce` prop.
  - [ ] Server Component; zero client JS (asserted in e2e).
- **[#9](https://github.com/MrLouisWright/next-herald/issues/9) JSON-LD XSS corpus in CI**
  - [ ] Corpus includes `</script>` case/whitespace variants, `<!--`, `<![CDATA[`, U+2028/2029, nested strings in keys.
  - [ ] For each payload, the rendered page's DOM contains exactly one script element and no injected nodes (Playwright).
  - [ ] Corpus documented and extendable via PR.
- **[#10](https://github.com/MrLouisWright/next-herald/issues/10) Schema types and builders batch 1**
  - [ ] Builders: organization, webSite (+searchAction), webPage, breadcrumbList (+breadcrumbsFromPath), article/blogPosting, product+offer+aggregateRating+review, faqPage.
  - [ ] Google-required properties are required in types (type tests).
  - [ ] schema-dts `WithContext<T>` values are accepted by `<JsonLd>` (type test with schema-dts as devDependency only).
  - [ ] Each builder's output validated against fixtures captured from the Schema.org validator.
- **[#11](https://github.com/MrLouisWright/next-herald/issues/11) defineSitemap (explicit routes) and defineRobots**
  - [ ] Returns native `MetadataRoute.Sitemap`; absolute, deduped, sorted URLs; `exclude` globs.
  - [ ] Dynamic patterns without a `params` enumerator are a type error.
  - [ ] `defineRobots` adds sitemap URLs, locks down when `indexing` is off, validates paths per RFC 9309.
  - [ ] Native `images`/`videos` fields passed through and rendered correctly by Next (e2e).
- **[#12](https://github.com/MrLouisWright/next-herald/issues/12) examples/basic and e2e harness**
  - [ ] `examples/basic` uses every v0.1 API.
  - [ ] Playwright e2e builds with `next build` and asserts head tags, JSON-LD, `/sitemap.xml`, `/robots.txt`.
  - [ ] Runs in CI under 5 minutes.
- **[#13](https://github.com/MrLouisWright/next-herald/issues/13) Docs-as-tests: type-check every code sample in docs/**
  - [ ] Script extracts ts/tsx code blocks from `docs/` and README and type-checks them against the package.
  - [ ] Blocks can opt out with an explicit `<!-- notest -->` marker and reason.
  - [ ] Runs in CI; wiki mirror regenerated from `docs/`.

**Exit criteria:** `examples/basic` and `examples/cache-components` pass e2e on the latest Next 16 and canary; dependency budget green; docs samples type-check. Then publish `0.1.0` (manual maintainer step).

## v0.2 — herald check core
Goal: The differentiator: a dependency-free `herald check` that catches the most common technical SEO mistakes in CI.

Scope: Zero-dep crawler and head scanner, title/description/canonical/noindex/sitemap/robots rules, herald routes.

**Issues and acceptance criteria**

- **[#14](https://github.com/MrLouisWright/next-herald/issues/14) herald CLI skeleton and config loading**
  - [ ] `node:util` `parseArgs`; `--help` for every command.
  - [ ] Loads `herald.config.ts` via Node type stripping (≥ 22.18), else `.mjs`/`.json` with a clear message.
  - [ ] No dependencies (enforced by #3).
- **[#15](https://github.com/MrLouisWright/next-herald/issues/15) Head scanner (own tolerant HTML tokenizer) with fuzz corpus**
  - [ ] Extracts title, meta, link, html lang, JSON-LD scripts and same-origin `<a href>`.
  - [ ] Corpus of real Next 16 output plus malformed HTML; results match a reference parser (devDependency in tests only) on ≥ 99% of corpus fields.
  - [ ] Fuzzing finds no crash or hang (time limit per document).
- **[#16](https://github.com/MrLouisWright/next-herald/issues/16) Crawler for --url: robots, sitemap seeding, concurrency, limits**
  - [ ] Seeds from `/`, robots.txt sitemaps (incl. indexes), `--seed`; follows same-origin links.
  - [ ] `--max-pages`, `--concurrency`, `--timeout`, response size caps; manual redirect tracking.
  - [ ] Honours robots.txt for its UA unless `--ignore-robots`.
  - [ ] `--build` runs `next build` + `next start` on a free port and shuts it down.
- **[#17](https://github.com/MrLouisWright/next-herald/issues/17) herald check --dir for static export output**
  - [ ] Maps URLs to `out/` files (`trailingSlash` both ways).
  - [ ] Same rule results as `--url` against `npx serve out` on the fixture.
- **[#18](https://github.com/MrLouisWright/next-herald/issues/18) Rules: title and description**
  - [ ] Implements title-missing, title-duplicate, title-length, description-missing, description-duplicate, description-length.
  - [ ] Duplicates grouped by canonical; noindex pages excluded.
  - [ ] Fixture tests per rule (positive and negative).
- **[#19](https://github.com/MrLouisWright/next-herald/issues/19) Rules: canonicals**
  - [ ] Implements canonical-missing, canonical-multiple, canonical-relative, canonical-broken, canonical-host, canonical-noindex.
  - [ ] `--site-url` lets a preview crawl compare against the production host.
- **[#20](https://github.com/MrLouisWright/next-herald/issues/20) Rules: noindex leaks and sitemap/robots consistency**
  - [ ] Implements noindex-leak (meta + X-Robots-Tag), sitemap-noindex, sitemap-url-status, sitemap-non-canonical, sitemap-missing, sitemap-invalid, sitemap-orphan, robots-missing, robots-blocks-sitemap-url, robots-blocks-noindex, robots-sitemap-host.
  - [ ] robots.txt matching per RFC 9309 (longest match, `*`, `$`) with the RFC examples as tests.
- **[#21](https://github.com/MrLouisWright/next-herald/issues/21) Reporters: pretty, json, github annotations; exit codes**
  - [ ] Exit 1 on any error, 2 on crawl failure, else 0.
  - [ ] `--format github` emits workflow annotations; `json` schema documented and versioned.
  - [ ] Severity overrides via config and `--rule`.
- **[#22](https://github.com/MrLouisWright/next-herald/issues/22) herald routes: scan app/ and generate typed routes**
  - [ ] Handles route groups, `@slot`, intercepting routes, `_private`, `src/app`, `pageExtensions`.
  - [ ] Writes `herald-routes.gen.ts`; `defineSitemap` infers required `params` keys from it (type tests).
  - [ ] `--watch` uses `fs.watch`; no orphaned watchers after exit.

**Exit criteria:** `herald check --build` on the examples reports zero errors, and each rule has a failing fixture that it catches.

## v0.3 — Large sites & i18n
Goal: Sites with >50k URLs, news publishers and multilingual sites.

Scope: Sitemap index splitting, news sitemaps, hreflang, next-tongue adapter, hreflang rules, static export.

**Issues and acceptance criteria**

- **[#23](https://github.com/MrLouisWright/next-herald/issues/23) Sitemap index splitting at 50k URLs / 50 MB**
  - [ ] `splitSitemap` + `sitemapIndex` produce valid chunks and index for a 120k-URL fixture.
  - [ ] Byte limit enforced (splits early if a chunk would exceed 50 MB).
  - [ ] Only one page of the source is loaded per chunk request.
  - [ ] sitemap-size and sitemap-index-broken rules implemented.
- **[#24](https://github.com/MrLouisWright/next-herald/issues/24) News sitemap route and image/video helpers**
  - [ ] `newsSitemap` drops items older than 2 days and caps at 1,000 URLs.
  - [ ] `sitemapVideo` enforces required video fields; output validated against Google's documented examples.
- **[#25](https://github.com/MrLouisWright/next-herald/issues/25) hreflang alternates from I18nSource**
  - [ ] `pageMetadata({ locale, locales })` and `defineSitemap` emit identical alternate sets including self and `x-default`.
  - [ ] Invalid codes rejected at config load.
  - [ ] Reciprocity holds for every page of `examples/i18n` (checked by #27).
- **[#26](https://github.com/MrLouisWright/next-herald/issues/26) fromTongue adapter and next-intl/Paraglide recipes**
  - [ ] `fromTongue` has no import of next-tongue (structural types).
  - [ ] Contract test against next-tongue once it publishes (skipped until then).
  - [ ] Recipes for next-intl and Paraglide verified in a scratch app and noted with versions.
- **[#27](https://github.com/MrLouisWright/next-herald/issues/27) Rules: hreflang and html lang**
  - [ ] Implements hreflang-code, hreflang-self, hreflang-reciprocal, hreflang-target-status, hreflang-canonical, hreflang-x-default, html-lang.
  - [ ] Sitemap alternates and HTML alternates are both checked and compared.
- **[#28](https://github.com/MrLouisWright/next-herald/issues/28) Static export compatibility e2e**
  - [ ] `examples/static-export` builds with `output: 'export'` including sitemap index via `force-static` route handlers, or the fallback `herald build --emit-xml` is implemented and documented.
  - [ ] `herald check --dir out` passes.

## v0.4 — OG images & structured data checks
Goal: Social previews and structured data validation.

Scope: OG templates, OG and JSON-LD rules, schema types batch 2, SARIF.

**Issues and acceptance criteria**

- **[#29](https://github.com/MrLouisWright/next-herald/issues/29) OG image templates and ogImage**
  - [ ] `SimpleCard`, `ArticleCard`, `ProductCard` with typed props; only Satori-supported CSS.
  - [ ] No fonts bundled; `fonts` option documented.
  - [ ] Works in Node and Edge runtimes.
- **[#30](https://github.com/MrLouisWright/next-herald/issues/30) OG template PNG snapshot tests**
  - [ ] Snapshots per template for LTR, long titles, and a CJK font.
  - [ ] Pixel diff threshold documented; runs in CI.
- **[#31](https://github.com/MrLouisWright/next-herald/issues/31) Rules: OG images**
  - [ ] Implements og-image-missing, og-image-status, og-image-dimensions, og-image-bytes, og-image-type, og-image-alt.
  - [ ] PNG/JPEG/WebP/GIF header readers with fixtures; Range requests where supported.
- **[#32](https://github.com/MrLouisWright/next-herald/issues/32) Rules: JSON-LD validity**
  - [ ] Implements jsonld-parse, jsonld-type, jsonld-required, jsonld-recommended for all built-in types.
  - [ ] Docs state these are a subset of Google's checks.
- **[#33](https://github.com/MrLouisWright/next-herald/issues/33) Schema types and builders batch 2**
  - [ ] newsArticle, person, localBusiness, event, videoObject, softwareApplication, recipe, jobPosting with required-field type tests.
  - [ ] `graph()` with `@id` references.
- **[#34](https://github.com/MrLouisWright/next-herald/issues/34) SARIF output and GitHub Action recipe**
  - [ ] `--format sarif` validates against SARIF 2.1.0 schema in tests.
  - [ ] Docs recipe uploads to GitHub code scanning.

## v0.5 — Adoption
Goal: Make adoption easy and back the README with published numbers.

Scope: herald init, verified migration guides, published size and perf report.

**Issues and acceptance criteria**

- **[#35](https://github.com/MrLouisWright/next-herald/issues/35) herald init scaffolder**
  - [ ] Creates config, sitemap, robots; `--large` and `--i18n` variants; never overwrites without `--force`.
  - [ ] Scaffolded app passes `herald check --build`.
- **[#36](https://github.com/MrLouisWright/next-herald/issues/36) Verified migration guide from next-seo**
  - [ ] Guide applied to a sample next-seo 7 App Router app; `herald check` before/after shows no regressions.
  - [ ] Every mapping row verified.
- **[#37](https://github.com/MrLouisWright/next-herald/issues/37) Verified migration guide from next-sitemap**
  - [ ] Guide applied to a sample next-sitemap app; URL sets identical (scripted diff).
  - [ ] Every mapping row verified.
- **[#38](https://github.com/MrLouisWright/next-herald/issues/38) Published size and check performance report**
  - [ ] Script measures client JS added (expected 0) and server bundle size per entry point.
  - [ ] `herald check` time and memory on a 10k-page fixture published with hardware notes.
  - [ ] Reproducible via one command.

## v0.9 — Beta (API freeze)
Goal: Freeze the API and rule IDs; harden.

Scope: API and rule-ID freeze, security review, upstream issue audit.

**Issues and acceptance criteria**

- **[#39](https://github.com/MrLouisWright/next-herald/issues/39) API and rule-ID freeze review**
  - [ ] Every public export reviewed and documented; deprecated names removed.
  - [ ] Rule IDs and default severities frozen and documented as stable.
- **[#40](https://github.com/MrLouisWright/next-herald/issues/40) Security review against the threat model**
  - [ ] Each threat row has a linked test.
  - [ ] Crawler SSRF limits (same-origin, size caps, off-origin redirects) tested.
- **[#41](https://github.com/MrLouisWright/next-herald/issues/41) Audit upstream Next.js metadata/sitemap issues**
  - [ ] Each listed issue re-checked on the latest Next; docs and troubleshooting updated.
  - [ ] Workarounds removed when fixed upstream.

## v1.0 — Stable
Goal: Stable release.

Scope: Semver guarantees, complete docs.

**Issues and acceptance criteria**

- **[#42](https://github.com/MrLouisWright/next-herald/issues/42) v1.0 release**
  - [ ] All v0.x acceptance criteria met; docs complete; semver policy published.
  - [ ] npm publish with provenance (separate, explicit maintainer action).

## Post-1.0 — Codemods
Goal: Automated migration.

Scope: next-seo and next-sitemap codemods.

**Issues and acceptance criteria**

- **[#43](https://github.com/MrLouisWright/next-herald/issues/43) Codemod: next-seo to next-herald**
  - [ ] Converts JSON-LD components and `DefaultSeo` config; reports what it can't convert.
  - [ ] Zero dependencies (own TSX transform scope documented) or shipped as a separate package — decided by ADR first.
- **[#44](https://github.com/MrLouisWright/next-herald/issues/44) Codemod: next-sitemap config to next-herald**
  - [ ] Converts `next-sitemap.config.js` into `herald.config.ts`, `app/sitemap.ts`, `app/robots.ts`.
  - [ ] Removes the postbuild script with a confirmation prompt.

## Not planned
See [rejected ideas](https://github.com/MrLouisWright/next-herald/blob/main/docs/research.md#rejected-ideas): head components, postbuild files in `public/`, headless browser crawling, request-derived URLs, Pages Router.
