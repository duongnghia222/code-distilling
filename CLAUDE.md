# CLAUDE.md

Guidance for Claude Code and other agents working **on this repository** (developing/maintaining the plugin itself). For using the plugin to port features, see `README.md`.

## What this repo is

`code-distilling` is a **Claude Code + Codex plugin** — not an application. It ships a set of skills that turn "copy this feature from that repo" into a two-stage flow (spec → implementation). There is no build step and no runtime; the deliverable is Markdown skills, hooks, and plugin manifests.

**Zero runtime dependencies by design.** Do not add third-party service/tool dependencies except when adding support for a new harness.

## Skills are behavior, not prose

The files under `skills/` are **agent instructions that shape how other agents behave** — treat them as code, not documentation.

- Don't reword carefully-tuned content (Red Flags tables, rationalization lists, Iron Law statements, "your human partner" phrasing) without evidence a change improves behavior.
- A skill change is only "done" after adversarial testing across sessions against a real reference repo — verifying the right skills fire, the Red Flags trip, and the checklist items get done. See `CONTRIBUTING.md`.

## Layout

```
.claude-plugin/     plugin.json (Claude manifest) + marketplace.json
.codex-plugin/      plugin.json (Codex manifest)
skills/             four skills with supporting references and optional handoffs
  using-code-distilling/        session-start bootstrap; routes on porting intent
  distillation-spec/            contract, architecture, behavioral design, integration, checks
  distillation-implementation/  direct implementation + fidelity and integration review
  distillation-gap/             upstream catch-up and remaining-gap closure for existing ports
hooks/              SessionStart hook that injects the bootstrap (session-start, run-hook.cmd)
scripts/            bump-version.sh
.version-bump.json  declares which files carry the version + audit excludes
```

## Versioning — always use the script

Two manifests carry the version (`.claude-plugin/plugin.json`, `.codex-plugin/plugin.json`) and **must stay in sync**. Never hand-edit them. Use:

```bash
scripts/bump-version.sh <X.Y.Z>   # bump all declared files, then audit for stragglers
scripts/bump-version.sh --check   # show current versions, detect drift
scripts/bump-version.sh --audit   # check + scan repo for undeclared version strings
```

The list of version-bearing files lives in `.version-bump.json`; add any new one there rather than bumping it by hand.

## Commit convention

Use **Conventional Commits**: `type(scope): summary` — e.g. `feat(...)`, `fix(...)`, `refactor(...)`, `chore(...)`, `docs(...)`. Scope with the area touched (`plugin`, `skills`, a capability name). Keep one logical change per commit; split unrelated changes.

> Note: this is the repo's *own* commit style. It is distinct from — but now matches — the commit convention the skills instruct downstream agents to use when distilling into a user's project (`feat(<feature>): ...`).

Only commit or push when the user asks. End commit messages with the `Co-Authored-By` trailer.

## Contribution bar is high

`CONTRIBUTING.md` is strict, and its "If you are an AI agent" section applies to you: verify a real problem motivates any change, search open/closed PRs for prior art, keep changes in-scope (domain/language/tool-specific work belongs in a separate plugin), and get explicit human approval on the complete diff before opening a PR.
