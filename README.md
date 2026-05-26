# code-distilling

Port high-quality implementations from reference open-source repos into your project, with discipline.

`code-distilling` is a Claude Code and Codex plugin that turns "I want to copy this feature from that repo" into a controlled workflow: analyze the reference, decide per-chunk what to copy / port / learn-then-rewrite, design a target API, plan the work, execute it with equivalence tests derived from the reference, and finish with full attribution and license compliance.

## How it works

You drop reference repos into a `ref-code/` folder at the root of your project:

```
my-project/
  src/
  ref-code/
    awesome-auth/        # full clone of the reference repo
    cool-search-lib/
  docs/
```

Then ask Claude or Codex to port a feature. The plugin walks you through:

1. **`analyzing-reference`** — deep-reads the reference repo, locates the feature, produces a reference map (files, dependencies, tests, entry points, license).
2. **`distillation-design`** — interactive design step. Decides per chunk: **copy** (same language, clean fit), **port** (translate idioms or language), or **learn-then-rewrite** (use the reference as teacher). Produces a distillation spec.
3. **`distillation-plan`** — source-to-target file map, task list with equivalence-test steps.
4. **`distillation-execution`** — dispatches subagents per task, each following `equivalence-tdd` (port the reference's test first, verify it fails, port the implementation, verify it passes).
5. **`attribution-and-license`** — license-compatibility check before work starts, per-file attribution headers, repo-level `ATTRIBUTION.md`, attribution-aware commit messages.

## Installation

This plugin ships manifests for both Claude Code (`.claude-plugin/`) and Codex (`.codex-plugin/`). Install via your harness's plugin mechanism pointing at this repo.

The `SessionStart` hook auto-loads the `using-code-distilling` bootstrap skill at session start, which causes the rest of the skills to fire when porting work is detected.

## Status

Early development. The architecture is documented in [`docs/specs/2026-05-26-code-distilling-plugin-design.md`](docs/specs/2026-05-26-code-distilling-plugin-design.md). v1 is considered ready when a single session can take "I want feature X from `ref-code/<repo>`" through to committed, tested, attributed code in the user's project.

## License

MIT. See [`LICENSE`](LICENSE).
