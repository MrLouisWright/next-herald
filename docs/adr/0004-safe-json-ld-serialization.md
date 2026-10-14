# 0004. JSON-LD is serialized safely by default

- Status: Accepted
- Date: 2026-10-14

## Context
The common pattern `dangerouslySetInnerHTML={{ __html: JSON.stringify(data) }}` is an XSS sink: a string containing `</script>` closes the tag. Next's [JSON-LD guide](https://nextjs.org/docs/app/guides/json-ld) warns about this and suggests replacing `<`. CMS and user content often ends up in `name`, `description` or `review` fields.

## Decision
- `<JsonLd>` and `serializeJsonLd()` are the only ways the library emits JSON-LD. They:
  - escape `<` → `\u003c`, `>` → `\u003e`, `&` → `\u0026`, U+2028 → `\u2028`, U+2029 → `\u2029` (valid JSON, inert in HTML);
  - throw in dev / drop with a warning in prod on functions, symbols, `bigint`, cycles and non-finite numbers; strip `undefined`;
  - add `@context: "https://schema.org"` when missing; support `@graph`.
- The component renders `<script type="application/ld+json">` with an optional `nonce` prop (CSP). It is a Server Component with no client JS.
- An XSS corpus (script-closing variants, HTML comments, `<!--`, CDATA, Unicode line separators) runs in CI ([#9](https://github.com/MrLouisWright/next-herald/issues/9)).

## Consequences
\+ Safe by default; no raw-HTML escape hatch. − Output differs byte-wise from `JSON.stringify` (still equal after `JSON.parse`).

## Alternatives considered
Escaping only `<`: rejected (`>`/`&` are cheap to cover; U+2028/2029 break older JS parsers when the JSON is reused in inline scripts).
