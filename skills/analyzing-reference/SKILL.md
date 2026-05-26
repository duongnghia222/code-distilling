---
name: analyzing-reference
description: Use when the user wants to port a feature from a reference repo in ref-code/ - produces a structured reference map (files, deps, tests, entry points, license) that drives every downstream skill
---

# Analyzing a Reference Repository

Deep-read a reference repo at `ref-code/<repo>/` to produce a **reference map** — a structured document the design and plan skills consume.

This is the first skill in the porting flow. Skipping it means downstream skills make decisions without evidence.

**Announce at start:** "I'm using the `analyzing-reference` skill to map the reference repo."

## Inputs

You must have, before invoking:

1. **Path to a reference repo:** a directory under `ref-code/`, e.g. `ref-code/awesome-auth/`.
2. **Feature description:** a sentence or two naming what the user wants to port (e.g., "the OAuth login and token refresh flow", "the in-memory LRU cache").

If either is missing, ask the user — one question at a time, multiple-choice when possible.

## Output

A reference map written to `docs/distilling/<repo>-<feature-slug>-reference-map.md` in the user's project. This is a **working artifact**, not yet committed (the `distillation-design` skill commits it together with the spec). The map has these sections:

```markdown
# Reference Map: <repo> — <feature>

**Reference repo path:** ref-code/<repo>/
**Source commit hash:** <hash from `git -C ref-code/<repo> rev-parse HEAD`>
**License:** <SPDX identifier from LICENSE file>
**License file:** ref-code/<repo>/LICENSE
**Primary language:** <language>
**Test framework:** <e.g., jest, pytest, go test>

## Feature scope (user's words)
<one paragraph>

## Feature files
- ref-code/<repo>/path/to/file1.ts — <one-line summary>
- ref-code/<repo>/path/to/file2.ts — <one-line summary>

## Transitive deps inside the repo
| File | Used by | Must take or can stub? | Why |
| ---- | ------- | ---------------------- | --- |
| ... | ... | must / can-stub | ... |

## External libraries
| Library | Version constraint | Used for | Equivalent in target? |
| ------- | ------------------ | -------- | --------------------- |
| ... | ... | ... | (filled in during design) |

## Tests for the feature
- ref-code/<repo>/test/file1.test.ts — covers <what>
- ref-code/<repo>/test/file2.test.ts — covers <what>

## Entry points
How callers invoke the feature from outside:
- function/class/method/endpoint, with signature

## Hidden coupling
Globals, env vars, file paths, side effects, time/random sources, network calls, OS-specific behavior. Each item names the file and line.

## License & attribution data
- License: <SPDX>
- Copyright holders: <names from LICENSE or package metadata>
- Author of feature files (best-effort from git log of feature files): <names>
- Recorded commit hash for attribution: <hash>

## Open questions for design
- <anything you couldn't resolve and need user input on>
```

## Checklist

Create one `TaskCreate` entry per item and complete in order:

1. **Confirm inputs** — path exists, feature description present.
2. **Snapshot reference metadata** — top-level files: README, LICENSE, package manifest, test config. Record license SPDX and source commit hash.
3. **Locate the feature** — search files by user's keywords + manifest entry points; deep-read candidates; confirm with user if multiple plausible matches.
4. **Walk transitive deps inside the repo** — for every feature file, trace imports recursively. Stop at the repo boundary. Mark each dep `must-take` (used by feature logic) or `can-stub` (only used for unrelated paths).
5. **List external libraries** — distinct from stdlib. Note version constraints.
6. **Find tests** — locate test files exercising feature files. Identify test framework.
7. **Identify entry points** — how callers invoke the feature.
8. **Surface hidden coupling** — globals, env vars, file IO, network, time, random, OS-specific calls inside the feature subgraph.
9. **Collect license & attribution data** — copyright holders, author names if discoverable.
10. **Write the reference map** to `docs/distilling/<repo>-<feature-slug>-reference-map.md`.
11. **Self-review** — placeholder scan, internal consistency, open questions listed explicitly.
12. **Hand off** — announce: "Reference map written to `<path>`. Invoking `distillation-design` next."

## How to do the work

### Locating the feature

If the user's description is fuzzy (e.g., "the search part"), pick the most plausible entry points from the README or manifest's `main`/`bin`/`exports`, follow them, and confirm with the user before committing to a scope. **One question at a time.**

### Walking transitive deps

Use language-appropriate import resolution:

- TypeScript/JavaScript: imports/requires, traverse with the project's `tsconfig`/`package.json` paths in mind.
- Python: `import` and `from ... import ...`, follow into the repo's package directories.
- Go: package imports, follow within the module.
- Rust: `use` statements, follow within the crate.
- Other: follow the language's import mechanism. If unfamiliar, ask the user to point you at the entry file.

Stop tracing as soon as you cross the repo boundary into external libraries — those go in the External libraries section, not transitive deps.

### Marking deps `must-take` vs `can-stub`

- **must-take:** the feature's behavior on its normal paths depends on this file. Without it, ported tests will fail.
- **can-stub:** the file is imported but only used in error paths, debug paths, or features outside the user's scope. A trivial stub or an empty default keeps the port working.

When in doubt, mark `must-take`. The design skill can downgrade later with evidence.

### Hidden coupling

Look for:

- Module-level mutable state (`let cache = ...` at top level, class-level statics).
- `process.env`, `os.environ`, runtime feature flags.
- Reads/writes to specific files or directories.
- Network calls (HTTP, sockets).
- `Date.now()`, `time.time()`, `random()`, `crypto.randomBytes()`.
- Threading, asyncio, event-loop assumptions.
- Platform-specific syscalls or path conventions.

Each item is an item the design step has to decide how to handle.

## What you do NOT do

- You do **not** decide what to copy / port / rewrite. That's `distillation-design`.
- You do **not** check license compatibility. That's `attribution-and-license` (invoked from `distillation-design`).
- You do **not** modify any file under `ref-code/`. The reference is read-only.
- You do **not** write code in the target project. You produce a document.

## When the reference is poor

If the reference has no tests, no clear feature boundary, broken builds, or a license you cannot identify: stop and report this to the user before writing the map. They may choose a different reference, a different feature, or accept the limitations explicitly.

## Anti-patterns

| Symptom | Fix |
|---------|-----|
| Reference map lists only the entry file, no transitive deps | Walk imports fully. Missing deps cause port failures. |
| Hidden coupling section is empty | Almost no real feature has zero hidden coupling. Look harder. |
| License section says "TBD" | Read `LICENSE`. If missing, report to user — do not guess. |
| Tests section says "the repo has tests" without listing them | Name the files, name the framework. Otherwise the design step has nothing to port. |
| Map includes opinions on how to port | Out of scope. Map describes what's there; design decides what to do. |
