# HTTP Header Value Handler — ESM + TypeScript Support ⚙️

https://github.com/duway18/http-header-value-handler-es/releases

[![Releases](https://img.shields.io/badge/releases-%20Download-blue?logo=github&style=flat-square)](https://github.com/duway18/http-header-value-handler-es/releases)

A compact ECMAScript module to parse, format, and validate HTTP header values. Works with modern ESM setups and with TypeScript types.

Table of contents
- Features
- Why use this module
- Install
- Quick start (ESM)
- TypeScript usage
- API reference
- Common examples
- CLI (release asset)
- Tests
- Contributing
- License
- Changelog

Features 🔧
- Parse header values to structured objects.
- Build header values from objects.
- Normalize and validate values per RFC 7230 token and quoted-string rules.
- Handle comma-separated lists, params, quality values, and structured headers.
- Tiny footprint. Pure ECMAScript. No runtime dependencies.

Why use this module 🧩
- HTTP header text often carries structured data. This module turns that text into structured values you can inspect and transform.
- It follows common spec rules for tokens, quoted-strings, and separators.
- It focuses on practical cases: Accept, Content-Type params, Cache-Control directives, custom header lists.

Install 📦
- For ESM projects:
  - npm: npm install http-header-value-handler-es
  - yarn: yarn add http-header-value-handler-es

Quick start — ESM (JavaScript)
- Import the module using ESM import syntax.
```js
import { parseHeaderValue, buildHeaderValue } from 'http-header-value-handler-es';

const accept = 'text/html, application/json;q=0.9, */*;q=0.1';
const parsed = parseHeaderValue(accept);
// parsed -> [{ value: 'text/html', params: {} }, { value: 'application/json', params: { q: '0.9' } }, ...]

const rebuilt = buildHeaderValue(parsed);
// rebuilt -> 'text/html, application/json;q=0.9, */*;q=0.1'
```

TypeScript usage 🧾
- The package ships types. Use import and let the compiler infer types.
```ts
import { parseHeaderValue } from 'http-header-value-handler-es';

type HeaderItem = {
  value: string;
  params: Record<string, string>;
};

const items: HeaderItem[] = parseHeaderValue('gzip, deflate, br;q=0.8');
```

API reference 🗂️
- parseHeaderValue(headerText: string, options?: ParseOptions): HeaderItem[]
  - Parse a header value string into an array of structured items.
  - Handles comma-separated lists, parameters, and quoted-strings.
  - Options:
    - allowEmpty (boolean) — include empty items (default: false)
    - preserveCase (boolean) — keep parameter key case (default: false)
  - HeaderItem:
    - value: string
    - params: Record<string, string>

- buildHeaderValue(items: HeaderItem[], options?: BuildOptions): string
  - Build a header value string from parsed items.
  - Options:
    - sortParams (boolean) — sort params by key (default: false)
    - quoteValues (boolean) — force quoting of params when needed (default: true)

- normalizeHeaderValue(headerText: string): string
  - Normalize spacing and token quoting to a canonical form.

- validateHeaderValue(headerText: string): ValidationResult
  - Validate header syntax.
  - Returns { valid: boolean, errors: string[] }

- escapeToken(token: string): string
  - Escape characters not allowed in a header token by returning a quoted-string.

Examples — common header tasks 🛠️
1) Parse Accept header and pick highest q:
```js
import { parseHeaderValue } from 'http-header-value-handler-es';

const accept = 'text/html, application/json;q=0.9, */*;q=0.1';
const items = parseHeaderValue(accept);
// pick item with highest q (default q=1)
const best = items
  .map(i => ({ ...i, q: parseFloat(i.params.q ?? '1') }))
  .sort((a,b) => b.q - a.q)[0];
```

2) Parse Content-Type and extract charset:
```js
const ct = 'text/html; charset=UTF-8';
const [main] = parseHeaderValue(ct);
const charset = main.params.charset ?? 'utf-8';
```

3) Normalize Cache-Control directives:
```js
import { parseHeaderValue, buildHeaderValue } from 'http-header-value-handler-es';

const raw = 'public, max-age=60, s-maxage=120';
const parsed = parseHeaderValue(raw, { preserveCase: true });
const normalized = buildHeaderValue(parsed, { sortParams: true });
```

Parsing details and rules (brief)
- The parser follows RFC 7230 basic rules:
  - token = 1*tchar
  - quoted-string supports backslash escapes
  - params follow value as ; key=value pairs
- The module treats comma outside quotes as list separator.
- It trims linear whitespace between tokens.
- It preserves param values with quotes when required.

Examples of return shapes
- parseHeaderValue('application/json') ->
  - [{ value: 'application/json', params: {} }]
- parseHeaderValue('text/html;level=1') ->
  - [{ value: 'text/html', params: { level: '1' } }]
- parseHeaderValue('a, b; q="0.2"') ->
  - [{ value: 'a', params: {} }, { value: 'b', params: { q: '0.2' }}]

CLI and Release asset 🚀
- Download the release asset from Releases and run the bundled CLI. The release page lists build artifacts. Download the asset file (for example: http-header-value-handler-es-<version>.tgz or http-header-value-handler-es-<version>.zip). After you extract it, run the bundled CLI with Node.
- Example:
  - Visit the Releases page and download the asset:
    https://github.com/duway18/http-header-value-handler-es/releases
  - After extraction, run:
```bash
# example after extracting a release archive
node dist/cli.mjs parse --header "Accept: text/html, application/json;q=0.9"
```
- The CLI exposes commands:
  - parse — parse a header string to JSON
  - build — build header string from JSON
  - validate — check header syntax

Testing 🧪
- The repo includes unit tests. Run:
```bash
npm test
```
- Tests cover:
  - token and quoted-string parsing
  - comma-separated lists
  - parameter parsing and building
  - edge cases for whitespace and escaping

Contributing 🤝
- Open an issue for bugs or enhancements.
- Fork, create a branch, and submit a pull request.
- Write tests for new behavior. Keep changes small and focused.
- Follow the existing code style and keep modules small.

Developer notes (implementation hints)
- Use a tokenizer that scans bytes for separators (, ; =) while respecting quoted regions.
- Build a small finite-state parser for tokens and quoted-string.
- Use minimal allocations and avoid large regexes for performance.
- Provide clear TypeScript types for each API function.

Benchmarks and performance
- The module targets low overhead for header parsing in HTTP servers.
- In micro-benchmarks, the parser performs well for common header sizes (< 1KB).
- Avoid heavy allocations in hot paths. Reuse small objects when possible.

Security and validation
- The module escapes tokens that contain control characters.
- It validates quoted-string escapes and reports errors for malformed inputs.
- Use validateHeaderValue for user-supplied headers before processing.

License
- MIT

Changelog
- See Releases for tagged builds and changelog entries:
  https://github.com/duway18/http-header-value-handler-es/releases

Images and visuals
- HTTP topic image:
  ![HTTP Topic](https://raw.githubusercontent.com/github/explore/main/topics/http/http.png)

- Use the badge above to jump to the release page or download an archive for CLI tests.