# Security

> **Status: planned / pre-release.** This wiki page mirrors [`docs/`](https://github.com/MrLouisWright/next-herald/blob/main/docs/README.md) and documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](https://github.com/MrLouisWright/next-herald/blob/main/ROADMAP.md).

Reporting and supported versions: [SECURITY.md](https://github.com/MrLouisWright/next-herald/blob/main/SECURITY.md).

## Threat model

| Asset | Trust boundary | Threat | Mitigation |
|---|---|---|---|
| Rendered HTML | CMS/user data → `<JsonLd>` | XSS via `</script>`, `<!--`, U+2028 | Safe serializer ([ADR 0004](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0004-safe-json-ld-serialization.md)); no raw-HTML API; XSS corpus in CI ([#9](https://github.com/MrLouisWright/next-herald/issues/9)) |
| Metadata | CMS data → `Metadata` | Tag injection | Next escapes attribute values; next-herald adds no raw tags |
| Sitemaps/XML | Data → XML | XML injection, invalid control chars | Escaping writer, control chars rejected, URLs RFC 3986 encoded |
| Canonical/hreflang | Request → URLs | Host-header poisoning of canonical | `siteUrl` from config only; request headers never read ([ADR 0006](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0006-cache-components-safety.md)) |
| Indexing | Env config | Preview/staging indexed; production `noindex` | `indexing: 'auto'`; `noindex-leak` rule |
| CI machine | `herald check` → network | SSRF / crawling third parties, huge responses | Same-origin only by default; `--max-pages`, per-response size cap (10 MB HTML, 64 KB image headers), timeouts; no redirects followed off-origin |
| Supply chain | npm | Malicious transitive deps | Zero dependencies ([ADR 0002](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0002-zero-dependencies.md)); npm provenance on publish |
| OG rendering | Data → Satori | Remote image fetch from user data | Docs: only pass trusted image URLs; templates never fetch URLs themselves |
