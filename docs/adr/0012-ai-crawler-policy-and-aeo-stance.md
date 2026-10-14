# 0012. AI crawler policy by purpose, and an evidence-based AEO stance

- Status: Accepted
- Date: 2026-10-14

## Context
The plan had one toggle, `aiCrawlers: 'allow' | 'disallow'`. AI vendors run separate bots for model **training**, **search indexing** and **user-initiated fetches**, and document different effects: OpenAI says sites that opt out of OAI-SearchBot "will not be shown in ChatGPT search answers"; Anthropic documents ClaudeBot, Claude-SearchBot and Claude-User; Perplexity documents PerplexityBot and Perplexity-User. One toggle makes "don't train on my content" silently also mean "don't cite me". Google-Extended is widely misread as an AI Overviews opt-out; Google says AI Overviews and AI Mode are controlled by Googlebot and snippet controls. Meanwhile "AEO/GEO" marketing promotes files and scores no engine documents. See the [review](../seo-aeo-review.md).

## Decision
1. `defineRobots(config, { ai: { training, search?, userFetch? } })`. `search` and `userFetch` default to `'allow'`. Omitting `ai` adds no AI-specific groups. Backed by a bundled, dated user-agent table with a source URL per entry, updated in minors; `robots-ai-list-stale` flags an old table ([#11](https://github.com/MrLouisWright/next-herald/issues/11), [#45](https://github.com/MrLouisWright/next-herald/issues/45)).
2. Docs state that Google-Extended controls Gemini/Vertex training and grounding, **not** AI Overviews/AI Mode, and point to `nosnippet`, `data-nosnippet`, `max-snippet`, `noindex`.
3. AEO work is limited to things engines document: crawlable server HTML, search-bot access, accurate dates, entity markup consistent with visible content, IndexNow. Each is a builder default or a `herald check` rule.
4. Unproven techniques (llms.txt, markdown alternates) ship only as **experimental opt-ins** in v0.5, documented as having no proven visibility benefit. Accept-header negotiation is a user `proxy.ts` recipe, never library code ([ADR 0006](0006-cache-components-safety.md)).
5. **No ranking or AI-citation claims** anywhere in docs or marketing. No "GEO score".

## Consequences
\+ Users can block training without leaving AI search; claims stay verifiable. − The UA table needs upkeep (dated, staleness rule). − Experimental entry points add surface we may remove before v0.9.

## Alternatives considered
- Keep a single toggle: rejected (removes sites from AI search answers by accident).
- Block all known AI bots by default: rejected (same, and opinionated).
- Content-Signal passthrough: deferred ([AEO stance](../aeo.md#considered-not-scheduled)).
- Read `Accept`/`User-Agent` in library code: rejected (ADR 0006).
