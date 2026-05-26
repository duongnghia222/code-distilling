# Attribution Header Templates

Each ported file carries an attribution header at the top, in the target language's primary comment style. Templates below — replace `<…>` placeholders with values from the reference map and the per-file attribution data.

## Fields

| Field | Source |
|-------|--------|
| Source repo URL | Reference map (resolved from `ref-code/<repo>` to its public URL if known; otherwise the local path) |
| Source path | Reference map; file or files this target was derived from |
| Source commit | Reference map (full SHA recorded by `analyzing-reference`) |
| Source license | Reference map (SPDX) |
| Distilled date | Today's date in YYYY-MM-DD |
| Mode line (optional) | `Implementation is independent; design influenced by the source.` — only for learn-then-rewrite |

## Templates by language

### TypeScript / JavaScript / Java / Go / Rust / C / C++ / Swift / Kotlin

```
/*
 * Source: <repo URL or ref-code/<repo>>
 * Source path: <path inside the source repo>
 * Source commit: <full SHA>
 * Source license: <SPDX>
 * Distilled into this project on <YYYY-MM-DD>
 */
```

For learn-then-rewrite, add a final line inside the block:

```
 * Implementation is independent; design influenced by the source.
```

### Python

```
"""
Source: <repo URL or ref-code/<repo>>
Source path: <path inside the source repo>
Source commit: <full SHA>
Source license: <SPDX>
Distilled into this project on <YYYY-MM-DD>
"""
```

For learn-then-rewrite, add at the bottom of the docstring:

```
Implementation is independent; design influenced by the source.
```

### Ruby / Shell / YAML / TOML / Dockerfile

```
# Source: <repo URL or ref-code/<repo>>
# Source path: <path inside the source repo>
# Source commit: <full SHA>
# Source license: <SPDX>
# Distilled into this project on <YYYY-MM-DD>
```

### HTML / Markdown / XML

```
<!--
  Source: <repo URL or ref-code/<repo>>
  Source path: <path inside the source repo>
  Source commit: <full SHA>
  Source license: <SPDX>
  Distilled into this project on <YYYY-MM-DD>
-->
```

### CSS / Less / Sass

```
/*
 * Source: <repo URL or ref-code/<repo>>
 * Source path: <path inside the source repo>
 * Source commit: <full SHA>
 * Source license: <SPDX>
 * Distilled into this project on <YYYY-MM-DD>
 */
```

### SQL / Lua

```
-- Source: <repo URL or ref-code/<repo>>
-- Source path: <path inside the source repo>
-- Source commit: <full SHA>
-- Source license: <SPDX>
-- Distilled into this project on <YYYY-MM-DD>
```

### Languages not listed

Use the language's primary multi-line comment if it has one, otherwise its single-line comment marker on each row. Keep the field order from the templates above.

## Multi-source files

If a target file was derived from more than one source file (typical for `port` mode when restructuring), repeat the `Source path:` line — one per source file — and use a single `Source commit:` (they should all be from the same commit in the same repo).

```
/*
 * Source: <repo URL>
 * Source path: src/auth/oauth.ts
 * Source path: src/auth/token.ts
 * Source commit: <full SHA>
 * Source license: <SPDX>
 * Distilled into this project on <YYYY-MM-DD>
 */
```

## What NOT to put in the header

- Marketing prose ("an excellent OAuth implementation by …"). Save credit prose for `ATTRIBUTION.md`.
- TODOs, FIXMEs, or implementation notes. Use regular comments below the header.
- The reference's full license text. The license text is preserved at `licenses/<repo>-<spdx>.txt` instead.
