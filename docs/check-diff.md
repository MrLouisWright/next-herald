# `herald check --diff` (v0.4)

> **Status: planned / pre-release.** This page documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](../ROADMAP.md).

Compare SEO output against a base build or branch and report only what changed, for PR comments ([#61](https://github.com/MrLouisWright/next-herald/issues/61)).

```bash
herald check --build --snapshot .herald/snapshot.json               # on main: save a snapshot
herald check --build --diff .herald/snapshot.json --format markdown  # on a PR: compare with a file
herald check --build --diff origin/main --format sarif               # or with a git ref
```

- **Snapshot:** deterministic JSON per URL (status, title, description, canonical, robots, hreflang, JSON-LD types) plus the sitemap URL set.
- **Git ref:** the base is built in a temporary `git worktree` with `node:child_process` and removed afterwards. No dependencies ([ADR 0002](adr/0002-zero-dependencies.md)).
- **Output:** `--format markdown` (PR-comment summary, per-URL details collapsed, length capped), `sarif` (2.1.0, [#34](https://github.com/MrLouisWright/next-herald/issues/34)), plus `pretty`, `json`, `github`.
- **Exit code:** 1 only if a diff finding is `error`; normal rules are not reported in diff mode.

## Diff rules
| ID | Severity | What it reports |
|---|---|---|
| `diff-noindex-added` | error | URL indexable in base is `noindex` now |
| `diff-canonical-changed` | warn | Canonical target changed |
| `diff-sitemap-url-removed` | warn | URL in base sitemap missing now |
| `diff-status-changed` | warn | HTTP status changed (e.g. 200 → 404) |
| `diff-title-changed` | info | Title changed |
| `diff-description-changed` | info | Description changed |

## GitHub Actions
```yaml
- run: npx herald check --build --diff origin/${{ github.base_ref }} --format markdown > herald-diff.md
```
Post `herald-diff.md` with your PR-comment step of choice. See [CLI](cli.md).
