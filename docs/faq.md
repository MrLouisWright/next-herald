# FAQ

> **Status: planned / pre-release.** This page documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](../ROADMAP.md).

### Why not just use native `generateMetadata`?
You should — next-herald returns native objects. It adds config-wide defaults, deep merging across segments, canonical/hreflang generation, safe JSON-LD, sitemap index/news, and a CI check. If you need none of that, native APIs are enough.

### Is next-herald faster / does it improve rankings?
No such claim. It helps you avoid technical SEO mistakes; rankings depend on much more. Bundle-size numbers will be published with reproducible scripts (v0.5).

### Why zero dependencies?
Supply-chain surface and maintenance ([ADR 0002](adr/0002-zero-dependencies.md)). The trade-off: we own a small head scanner and XML writer.

### Can I use schema-dts types?
Yes. `<JsonLd>` accepts any `@type` object, including schema-dts `WithContext<T>` values ([ADR 0005](adr/0005-own-schema-types.md)).

### Does it support the Pages Router or Next 15?
No ([ADR 0003](adr/0003-nextjs-16-only.md)). Keep next-seo / next-sitemap there.

### Do I need next-tongue?
No. Any i18n setup works through `I18nSource` ([hreflang & i18n](i18n-hreflang.md)).

### next-intl or Paraglide?
<a id="next-intl-or-paraglide"></a>
```ts
// next-intl (routing from i18n/routing.ts)
import { getPathname } from '@/i18n/navigation';
i18n: { locales: routing.locales, defaultLocale: routing.defaultLocale, href: (locale, pathname) => getPathname({ locale, href: pathname }) }
// Paraglide JS 2
import { locales, baseLocale, localizeHref } from '@/paraglide/runtime';
i18n: { locales, defaultLocale: baseLocale, href: (locale, pathname) => localizeHref(pathname, { locale }) }
```
Verify signatures against your library version; these are recipes, not tested adapters.

### Does `herald check` run JavaScript?
No. It reads server HTML, like many crawlers' first pass ([ADR 0008](adr/0008-check-crawls-http.md)).

### Why doesn't it read `.next` manifests to find pages?
They are internal and change between minors. It crawls from sitemaps and links, and `herald routes` scans `app/`.

### Is `priority`/`changefreq` useful?
Google ignores them; they are supported for other consumers. `lastModified` should be a real content date.
