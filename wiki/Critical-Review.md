# Critical review (October 2026)

Two review passes over the plan on 2026-10-14, focused on consistency, accuracy and feasibility under Next 16 + Turbopack.

## Pass 1 — feasibility
| # | Concern | Resolution |
|---|---|---|
| F1 | A TS config cannot be loaded by a zero-dep CLI on Node 20 | Node type stripping (≥ 22.18) or `.mjs`/`.json` fallback ([ADR 0009](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0009-cli-config-and-route-scan.md)) |
| F2 | "Sitemap from the route manifest" would need `.next` internals | `herald routes` scans `app/` with `fs`; `params` enumerators required and typed |
| F3 | Next has no sitemap index; `sitemap.ts` + route handler conflict (#87314) | Large-site mode uses `app/sitemaps/sitemap.ts` + `app/sitemap.xml/route.ts` ([ADR 0007](https://github.com/MrLouisWright/next-herald/blob/main/docs/adr/0007-sitemap-index-via-route-handlers.md)) |
| F4 | Turbopack: no bundler plugin needed? | Correct — next-herald has no `next.config` wrapper or loader; only plain imports and generated TS. Nothing Turbopack-specific |
| F5 | Route handlers under `output: 'export'` | `force-static` GET, verified by e2e (#28); fallback `herald build --emit-xml` documented |
| F6 | HTML parsing without deps | Head-only scanner with fuzz corpus; explicit limits |
| F7 | Edge runtime for OG/sitemaps | Runtime entry points avoid `node:` imports (test) |

## Pass 2 — consistency and accuracy
| # | Finding | Fix |
|---|---|---|
| C1 | README draft claimed next-seo is "Pages Router only"; v7 supports App Router JSON-LD | Comparison table corrected |
| C2 | #98684 initially described as Cache Components-specific | It's a segment-cache prefetch race; wording fixed everywhere |
| C3 | Rule IDs drifted between cli.md and issues | cli.md is the single source; issues reference it |
| C4 | `priority`/`changefreq` presented as SEO levers | Marked as ignored by Google |
| C5 | OG size rule "must be 1200×630" too strict | ≥ 1200×630, aspect within 10% |
| C6 | Milestones vs ROADMAP mismatch | Issue milestones re-checked against ROADMAP |

## Residual risks
- Upstream metadata/sitemap regressions in 16.x minors (canary CI mitigates).
- Head scanner false positives on exotic HTML.
- v0.1 scope creep: scope freeze until v0.1 ships.
