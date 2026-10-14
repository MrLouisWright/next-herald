# Changelog

All notable changes to this project will be documented in this file. The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project will adhere to [Semantic Versioning](https://semver.org/) from v1.0 (0.x releases may break between minors).

## [Unreleased]

### Added
- Project planning and documentation: README, `docs/` (planned API), ROADMAP with acceptance criteria, ADRs 0001–0011, research and critical review, community health files, issue/PR templates, wiki pages (and the `/wiki` mirror).
- Labels, milestones v0.1–post-1.0, roadmap issues and the public project board.
- SEO/AEO review applied (2026-10-14): [AEO stance](docs/aeo.md), [ADR 0012](docs/adr/0012-ai-crawler-policy-and-aeo-stance.md), the review with a verification addendum ([docs/seo-aeo-review.md](docs/seo-aeo-review.md)), 27 planned `herald check` rules, `herald indexnow` (v0.5), entity graph by default (v0.2), accurate `lastmod` (v0.3), experimental markdown alternates and llms.txt (v0.5), AI referral measurement docs; issues #45–#56.

### Changed
- Planned `defineRobots` option `aiCrawlers: 'allow' | 'disallow'` replaced by `ai: { training, search?, userFetch? }` (search and user-fetch allowed by default) (#11).
- Docs: Google-Extended framing corrected (it does not control AI Overviews/AI Mode); FAQPage no longer presented as a rich-result win; no HowTo.
- `graph()` and `person` moved from v0.4 (#33) to v0.2 (#49).
- README positioning and comparison: Nuxt SEO and Astro added; "no ranking or AI-citation claims".

No package has been published yet.
