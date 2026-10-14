# Contributing to next-herald

Thanks for your interest! next-herald is **pre-alpha**: there is no library code yet. The most valuable contributions right now are:

1. **Design feedback** on the planned API ([docs](docs/README.md)) and [ADRs](docs/adr/README.md) — open an issue with the design feedback template.
2. **Real-world cases** — SEO bugs you hit with Next.js 16 metadata, sitemaps or hreflang; they become `herald check` rules or fixtures.
3. **Picking up a roadmap issue** in [v0.1](https://github.com/MrLouisWright/next-herald/milestone/1) once scaffolding lands.

## Ground rules
- Be kind; we follow the [Code of Conduct](CODE_OF_CONDUCT.md).
- Discuss significant changes in an issue first. Changes to core principles need an ADR (`docs/adr/template.md`).
- **No dependencies.** PRs must not add `dependencies`, `optionalDependencies` or peers beyond `next`/`react` ([ADR 0002](docs/adr/0002-zero-dependencies.md)). devDependencies are fine.
- **No request APIs** (`headers()`, `cookies()`, `connection()`) in library code ([ADR 0006](docs/adr/0006-cache-components-safety.md)).
- Docs and code stay consistent: API changes update `docs/api-reference.md` and every affected guide; new check rules update the table in `docs/cli.md`.

## Development (planned, from v0.1)
```bash
pnpm install
pnpm test          # vitest
pnpm typecheck     # tsc + type tests
pnpm lint
pnpm e2e           # playwright against examples/
pnpm changeset
```
Repo layout (planned): `packages/next-herald` (the one published package), `examples/*`, `test/fixtures/`, `docs/`, `wiki/` (mirror of the GitHub wiki).

**Scope freeze:** no new features are accepted before v0.1 ships ([ROADMAP](ROADMAP.md)).

## Pull requests
- One logical change per PR; link the issue (`Closes #123`).
- Add tests and a changeset. Conventional commit titles (`feat(sitemap): …`, `docs: …`) are appreciated.
