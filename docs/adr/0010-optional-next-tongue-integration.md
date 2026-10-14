# 0010. next-tongue integration is optional and structural

- Status: Accepted
- Date: 2026-10-14

## Context
hreflang needs locales, a default locale and a way to build each locale's URL. [next-tongue](https://github.com/MrLouisWright/next-tongue) knows these, but most users of next-herald won't use it (next-intl, Paraglide, hand-rolled).

## Decision
- Core API takes a plain `I18nSource`: `{ locales, defaultLocale, href(locale, pathname) }`. hreflang builders validate BCP 47 tags and add `x-default`.
- `next-herald/i18n` exports `fromTongue(tongueConfig, getPathname)` that adapts next-tongue structurally. **next-herald never imports next-tongue** and does not list it as a peer.
- Recipes document next-intl and Paraglide adapters (a few lines each).

## Consequences
\+ No coupling; zero deps preserved. − Adapter drift if next-tongue changes; covered by a contract test in CI once next-tongue publishes.

## Alternatives considered
A peer/optional peer on next-tongue: rejected.
