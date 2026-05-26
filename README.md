# code-distilling

> Port high-quality implementations from reference open-source repos into your project — with discipline.

`code-distilling` is a Claude Code and Codex plugin that turns *"I want to copy this feature from that repo"* into a controlled workflow: analyze the reference, decide per-chunk what to **copy** / **port** / **learn-then-rewrite**, design a target API, plan the work, execute it with equivalence tests derived from the reference, and finish with full attribution and license compliance.

It is a sister-plugin to [Superpowers](https://github.com/obra/superpowers) and follows the same skill-driven discipline. You do not need Superpowers installed to use it.

## Why this exists

When you find a great open-source implementation of something you need — a parser, an auth flow, a search algorithm — there are three failure modes:

1. **Paste-and-pray.** Copy the code in, rename a few symbols, hope it works. No license trail, no tests, no idea what you brought along for the ride.
2. **From-scratch-with-vibes.** Read the reference, close the tab, write your own. Lose all the edge cases the original handled.
3. **Vendor-and-forget.** Add the repo as a dependency, drag in their build system, never actually understand what's running.

`code-distilling` is the disciplined fourth option: **port a curated subset of the reference into your own codebase, with the reference's tests as your acceptance criteria, and ship it with the attribution your license obligations require.**

## How it works

You drop reference repos into a `ref-code/` folder at the root of your project:

```
my-project/
  src/
  test/
  ref-code/
    awesome-auth/        # full clone of the reference repo
    cool-search-lib/
  docs/
```

Then ask Claude or Codex to port a feature. The plugin walks you through five skills:

1. **`analyzing-reference`** — deep-reads the reference repo, locates the feature, produces a reference map (files, dependencies, tests, entry points, hidden coupling, license).
2. **`distillation-design`** — interactive design step. Decides per chunk: **copy** (same language, clean fit), **port** (translate idioms or language), or **learn-then-rewrite** (use the reference as teacher only). Produces a distillation spec the user approves.
3. **`distillation-plan`** — source-to-target file map, task list with paired test/implementation steps, attribution headers.
4. **`distillation-execution`** — dispatches a fresh subagent per task, each following `equivalence-tdd` (port the reference's test first, watch it fail, port the implementation, watch it pass, commit). Two-stage review per task (spec compliance, then code quality).
5. **`attribution-and-license`** — license-compatibility gate before work starts, per-file attribution headers, repo-level `ATTRIBUTION.md`, attribution-aware commit messages.

The skills auto-trigger when the agent sees porting intent or a `ref-code/` directory. You don't need to type any slash commands.

## Quickstart

```bash
# 1. In your project, drop a reference repo into ref-code/
mkdir -p ref-code
git clone https://github.com/<org>/<reference-repo> ref-code/<reference-repo>

# 2. Open your coding agent (Claude Code, Codex, etc.)

# 3. Ask:
#    "I want to port the OAuth flow from ref-code/<reference-repo>
#     into src/auth/."
```

The agent will engage `analyzing-reference` first, then walk you through the rest of the flow.

## Installation

### Claude Code

```bash
/plugin install duongnghia222/code-distilling
```

Or clone this repo and add the plugin path manually:

```bash
git clone https://github.com/duongnghia222/code-distilling
/plugin install ./code-distilling
```

### Codex

This repo ships a `.codex-plugin/` manifest. Install it through your harness's plugin mechanism pointing at this repo.

### Other harnesses

The plugin uses a standard `SessionStart` hook that emits `additionalContext` JSON. Any harness that supports session-start hooks and additional-context injection can use it. See `hooks/session-start` for the platform-detection logic.

## What you'll see in a session

After installation, the agent loads the `using-code-distilling` bootstrap at session start. The bootstrap is silent until porting intent appears. Then:

```
User: I want to port the OAuth flow from ref-code/awesome-auth into src/auth/.

Agent: I'm using `analyzing-reference` to map the reference repo.
       [reads ref-code/awesome-auth/, locates the OAuth files]
       [writes docs/distilling/awesome-auth-oauth-reference-map.md]

       Reference map written. A few clarifying questions before we design:
       - Q1: Target API shape — mirror the reference, wrap with your conventions,
             or re-shape the interface? (A/B/C)
       ...

       [after Q&A, writes docs/specs/2026-05-26-distill-awesome-auth-oauth.md]
       Spec written and committed. Please review before we move to the plan.

User: Approved.

Agent: I'm using `distillation-plan` to write the implementation plan.
       [writes docs/plans/2026-05-26-distill-awesome-auth-oauth.md]
       Plan written and committed. Invoking `distillation-execution`.

Agent: [dispatches implementer subagents per task]
       [two-stage review per task]
       [final attribution pass]
       Done — 5 chunks distilled, 17 tests passing, ATTRIBUTION.md updated.
```

## The skills

| Skill | When it fires | What it produces |
|-------|---------------|------------------|
| `using-code-distilling` | Session start (bootstrap) | Triggers downstream skills on porting intent |
| `analyzing-reference` | First, when porting intent is detected | Reference map document |
| `distillation-design` | After the reference map exists | Distillation spec (modes, target API, test plan, attribution plan) |
| `distillation-plan` | After the spec is approved | Task-by-task implementation plan |
| `distillation-execution` | After the plan is approved | Subagent-driven execution of every task |
| `equivalence-tdd` | Inside execution, per task | Rigid test-first discipline for porting |
| `attribution-and-license` | Three times: compatibility check, per-file headers, final pass | License compliance + `ATTRIBUTION.md` + commit trailers |

## Three modes for every chunk

- **copy** — same language, license clean, idiomatic for target. Minimal changes (rename imports/types).
- **port** — different language, OR same language with materially different idioms, OR target-incompatible patterns (browser globals in a Node project).
- **learn-then-rewrite** — the chunk is heavily entangled with reference-specific infrastructure that doesn't exist in the target; OR the value is the algorithmic approach, not the code itself; OR cross-language translation is so heavy that "porting" is misleading.

Mode is decided per chunk during `distillation-design`, using explicit criteria documented in `skills/distillation-design/references/mode-decision-criteria.md`. The deciding criterion is recorded in the spec so reviewers can validate.

## Philosophy

- **The reference is the spec.** Tests come from the reference, not from your imagination. If the reference has no tests, you capture behaviors from a real run.
- **Modes are explicit.** Every chunk has a mode and a deciding criterion. "Just trust me" is not a criterion.
- **License compliance is non-optional.** No port without a compatibility check. No file without a header. No commit without a `Source:` trailer.
- **Subagents per task.** Fresh context per task keeps the implementer focused. Two-stage review (spec compliance, then code quality) prevents over- or under-building.
- **Process over guessing.** Skipping the analysis or the spec produces ports nobody can audit. The flow scales — short specs for small ports — but you still run it.

## Status

Early development (v0.1.0). The architecture is documented in [`docs/specs/2026-05-26-code-distilling-plugin-design.md`](docs/specs/2026-05-26-code-distilling-plugin-design.md).

**v1 is considered ready when:** a single session can take *"I want feature X from `ref-code/<repo>`"* through to committed, tested, attributed code in the user's project — without manual intervention beyond approving the spec and the plan.

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md). The short version:

- One problem per PR.
- Fill in the entire `.github/PULL_REQUEST_TEMPLATE.md`.
- Skills are not prose — they are agent behavior. Changes need evidence.
- Test on at least one harness and report results.

## License

MIT. See [`LICENSE`](LICENSE).

## Acknowledgements

`code-distilling` is heavily inspired by [Superpowers](https://github.com/obra/superpowers) by Jesse Vincent and the team at Prime Radiant. The skill structure, the bootstrap-via-hook mechanism, the two-stage review pattern, and the writing voice all draw on Superpowers' design choices. If you don't already use Superpowers for general agent-driven development, you probably should.
