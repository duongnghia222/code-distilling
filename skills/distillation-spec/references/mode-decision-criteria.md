# Mode Decision Criteria

Apply these in order. The first one that fires assigns the mode for the chunk. Record which criterion fired in the spec's chunk table (adaptation-notes column).

## learn-then-rewrite — apply if ANY of:

1. The chunk is structurally entangled with reference-specific infrastructure that does not exist in the target: DI containers, framework lifecycle hooks, event buses, decorator-heavy registries, framework-provided base classes. Untangling would change more code than rewriting.
2. The value of the chunk is in the algorithmic or architectural *approach* rather than the code itself (e.g., the reference demonstrates a clever data layout but the implementation is dense and target-specific).
3. The cross-language gap is so wide that "port" is misleading (e.g., heavily-templated C++ → Python, or Rust macro-driven code → JavaScript).
4. The chunk is small and idiomatic-target rewrites are clearly cleaner than translations.

## port — apply if ANY of (and learn-then-rewrite did not fire):

1. Target language differs from source language.
2. Same language, but materially different idioms (callback chains ↔ async/await, sync ↔ async, class-based ↔ functional).
3. Same language, but the chunk uses target-incompatible primitives (browser globals in a Node-only target, Node-only modules in a browser target).
4. Same language, but the chunk's style or naming conventions clash with the target project's conventions enough that adaptation is non-trivial.
5. The chunk imports a library not available in the target stack and a substitution is being made.

## copy — apply if ALL of (and neither of the above fired):

1. Target language equals source language.
2. Idioms match the target project's style.
3. No incompatible primitives or library substitutions required.
4. The chunk is self-contained or its deps are also being copied.

## Examples

| Situation | Mode | Why |
|-----------|------|-----|
| TypeScript utility function, target is also TypeScript, matches project conventions | copy | All copy criteria pass. |
| Python repo's `requests`-based HTTP client, target is Node.js | port | Cross-language. |
| Java Spring Boot service class, target is plain Node.js Express | learn-then-rewrite | Framework entanglement (Spring lifecycle). |
| React class component, target is React but functional/hooks style | port | Same language, different idioms. |
| TypeScript algorithm, target is TypeScript, but reference uses `lodash` and target avoids lodash | port | Same language, library substitution. |
| Rust async runtime helper, target is JavaScript event-loop code | learn-then-rewrite | Cross-language gap too wide for a meaningful port. |

## Conflicting signals

If two criteria from different modes fire for the same chunk, prefer the more conservative mode (learn-then-rewrite > port > copy). Record both fired criteria in the chunk table's adaptation-notes column.

## When to revisit

If during execution a chunk marked `port` turns out to need so much restructuring that it crosses into rewrite territory, pause and escalate to the user — do not silently shift modes mid-port. Mode shifts are recorded as a spec amendment.
