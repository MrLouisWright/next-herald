# Architecture

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

```
herald.config.ts ──► defineHerald (validated, frozen)
        │
        ├─► next-herald/metadata ─► Metadata objects ─► Next renders <head>  (ADR 0001)
        ├─► next-herald/jsonld  ─► <script type=application/ld+json> (safe serializer, ADR 0004)
        ├─► next-herald/sitemap ─► MetadataRoute.Sitemap | XML Response (index, news; ADR 0007)
        │         ▲ herald-routes.gen.ts ◄── herald routes (fs scan of app/, ADR 0009)
        ├─► next-herald/robots  ─► MetadataRoute.Robots
        └─► next-herald/og      ─► next/og ImageResponse (ADR 0011)

herald check ─► fetch (server) | fs (out/) ─► head scanner ─► page model ─► rules ─► reporters
```

## Runtime
- Every entry point is tree-shakable ESM, `sideEffects: false`. No module-level state besides frozen config.
- No request APIs ([ADR 0006](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0006-cache-components-safety.md)). Works in the Node and Edge runtimes (no `node:` imports in runtime entry points; enforced by test).
- XML writer: escapes `& < > " '`, rejects control characters, percent-encodes non-ASCII in `loc` per RFC 3986.

## CLI
- `node:util` `parseArgs`; config loading by dynamic `import()` (type stripping on Node ≥ 22.18).
- **Head scanner**: a streaming tokenizer that reads until `</head>` (or the first `<body>` content, plus body `<script type="application/ld+json">` and `<a href>` for link discovery). It handles comments, CDATA, unquoted attributes and entity decoding for the attributes we read. It is not an HTML5-spec parser; fuzzed against a corpus of real Next output and malformed HTML.
- **Crawler**: bounded queue (`--concurrency`), per-request timeout via `AbortController`, redirects tracked manually (`redirect: 'manual'`), `robots.txt` parsed per RFC 9309, gzip handled by `fetch`/`zlib`.
- **Page model** → **rules** (pure functions over pages + site graph) → **reporters** (`pretty`, `json`, `github`, `sarif`).
- Image header readers (PNG IHDR, JPEG SOFn, WebP VP8/VP8L/VP8X, GIF) use `Range: bytes=0-65535` when supported.

## Repo layout (planned)
`packages/next-herald` (single published package), `examples/basic`, `examples/cache-components`, `examples/i18n`, `examples/large-sitemap`, `examples/static-export`, `test/fixtures` (HTML corpus, XSS corpus, sitemap corpus), `docs/`.
