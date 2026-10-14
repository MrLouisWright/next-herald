# hreflang & i18n

> **Status: planned / pre-release.** This page documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](../ROADMAP.md).

hreflang annotations tell search engines about [localized versions](https://developers.google.com/search/docs/specialty/international/localized-versions). Rules: every version lists **all** versions including itself, links must be **reciprocal**, URLs absolute, codes are ISO 639-1 language (+ optional ISO 3166-1 region or script), plus an optional `x-default`.

## Configure an `I18nSource`
```ts
// herald.config.ts
export default defineHerald({
  siteUrl: 'https://example.com',
  i18n: {
    locales: ['en', 'de', 'fr-CA'],
    defaultLocale: 'en',
    href: (locale, pathname) => (locale === 'en' ? pathname : `/${locale}${pathname}`),
    xDefault: 'default-locale',   // or a path like '/' (language picker), or false
  },
});
```
`pageMetadata(herald, { path, locale })` then sets `alternates.canonical` to the locale's own URL and `alternates.languages` for every locale; `defineSitemap` adds the same alternates to each entry. Pages missing in a locale: pass `locales: ['en', 'de']` per page so only existing versions are listed (otherwise reciprocity breaks).

## With next-tongue (optional)
next-herald does not import [next-tongue](https://github.com/MrLouisWright/next-tongue) ([ADR 0010](adr/0010-optional-next-tongue-integration.md)). The adapter is structural:
```ts
import { fromTongue } from 'next-herald/i18n';
import tongue from './tongue.config';
import { getPathname } from 'next-tongue/navigation';
export default defineHerald({ siteUrl: 'https://example.com', i18n: fromTongue(tongue, getPathname) });
```
It respects next-tongue's `prefix` mode and localized pathnames, because `getPathname` does the mapping. Use the locale from route `params` (`await params`) — consistent with next-tongue's params-only design and Cache Components.

## With next-intl or Paraglide
Write `href` with the library's path helper (next-intl `getPathname`, Paraglide `localizeHref`). Recipes in the [FAQ](faq.md#next-intl-or-paraglide).

## Checks
`hreflang-reciprocal` (error), `hreflang-self` (error), `hreflang-code` (error), `hreflang-x-default` (info), `hreflang-canonical` (warn: canonical points to another locale), `hreflang-target-status` (error: alternate is non-200 or noindex). See [CLI](cli.md).
