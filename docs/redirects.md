# Redirect manager (v0.3)

> **Status: planned / pre-release.** This page documents the intended API of next-herald. Nothing is published yet; names may change before v1.0. See [ROADMAP](../ROADMAP.md).

One typed list of redirects feeds `next.config` `redirects()` or your own `proxy.ts`, and `herald check` verifies what is served ([#58](https://github.com/MrLouisWright/next-herald/issues/58)).

```ts
// redirects.ts
import { defineRedirects } from 'next-herald/redirects';
import herald from '@/herald.config';

export default defineRedirects(herald, [
  { source: '/old-blog/:slug', destination: '/blog/:slug' },            // permanent by default (308)
  { source: '/pricing-2025', destination: '/pricing' },
  { source: '/sale', destination: '/offers/autumn', permanent: false },  // intentionally temporary
]);
```

```ts
// next.config.ts
import type { NextConfig } from 'next';
import redirects from './redirects';

const nextConfig: NextConfig = {
  redirects: async () => redirects.toNextConfig(),
};
export default nextConfig;
```

For many rules or rules from a CMS, match in your own proxy instead. `match(pathname)` is pure; the request is read by your code, not the library:

```ts
// proxy.ts
import { NextResponse, type NextRequest } from 'next/server';
import redirects from './redirects';

export function proxy(request: NextRequest) {
  const hit = redirects.match(request.nextUrl.pathname);
  if (hit) return NextResponse.redirect(new URL(hit.destination, request.url), hit.status);
}
```

## Validation
At load: `source` starts with `/`, no duplicate sources, destinations absolute or root-relative, `permanent` defaults to `true`. Errors name the rule, e.g. `herald: redirects[2].source must start with "/".` `herald build` reports chains (A → B → C) inside the rule set and fails on loops.

## Check rules
| ID | Severity | What it reports |
|---|---|---|
| `redirect-chain` | warn | More than one hop before a 200 |
| `redirect-loop` | error | Redirects that never reach a final response |
| `redirect-temporary` | warn | 302/307 where a permanent redirect is intended (rule declared `permanent`, or the URL is in a sitemap or linked internally) |

See [CLI](cli.md#rules).

## Static export
`output: 'export'` does not support `redirects` or proxy. Use `redirects.toNextConfig()` to generate your host's redirect file in a build script.
