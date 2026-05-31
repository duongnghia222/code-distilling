# code-distilling

> Port high-quality implementations from reference open-source repos into your project — with discipline.

`code-distilling` is a Claude Code and Codex plugin that turns *"I want to copy this feature from that repo"* into a controlled workflow: analyze the reference, write a distillation spec (decide per-chunk what to **copy** / **port** / **learn-then-rewrite**, what to keep verbatim, how the seams wire into your project), implement it, and verify the result against the reference.

It is a sister-plugin to [Superpowers](https://github.com/obra/superpowers) and follows the same skill-driven discipline. You do not need Superpowers installed to use it.

## Why this exists

When you find a great open-source implementation of something you need — a parser, an auth flow, a search algorithm — there are three failure modes:

1. **Paste-and-pray.** Copy the code in, rename a few symbols, hope it works. No idea what you brought along for the ride.
2. **From-scratch-with-vibes.** Read the reference, close the tab, write your own. Lose all the edge cases the original handled.
3. **Vendor-and-forget.** Add the repo as a dependency, drag in their build system, never actually understand what's running.

`code-distilling` is the disciplined fourth option: **port a curated subset of the reference into your own codebase — keeping the encoded decisions that make it good, and verifying the result against the reference.**

The core idea: **the code is not the asset; the decisions encoded in the code are the asset.** A tuned threshold, the order of two steps, an edge-case branch, a prompt template — those are months of someone's tuning. Distillation keeps that gold and drops the packaging (their framework, config, logging, abstractions for their scale).

## How it works

You point the plugin at a reference repo by path — any path on disk works. Clone it wherever you keep code; you don't have to copy it into your project:

```
~/code/
  my-project/            # your project — no ref-code/ folder needed
    src/
    test/
    docs/
  awesome-auth/          # reference repo, anywhere
  cool-search-lib/
```

Then ask Claude or Codex to port a feature, naming the reference's path. The plugin walks you through five stages, with human approval gates after the analysis, the spec, and the plan:

1. **`reference-analysis`** — deep-reads the reference, locates the one capability, explains its design, and agrees with you on exactly what to copy. Produces `reference-analysis.md`.
2. **`distillation-spec`** — writes the distillation spec: the behavioral contract, the keep-verbatim list (tuned constants, prompts, step order), the discard list, the seam → your-deps mapping, and a per-chunk mode (**copy** / **port** / **learn-then-rewrite**). You approve it.
3. **`distillation-plan`** — turns the spec into a bite-sized, file-mapped implementation plan: a source→target file map and per-task steps carrying the mode, keep-verbatim items, and seam substitutions, with complete code in every step. You approve it.
4. **`distillation-implementation`** — executes the plan, task by task: a fresh subagent per logic-heavy task with two-stage review (spec compliance, then code quality); simple tasks implemented directly. Runs continuously.
5. **`gap-report`** — a fresh-eyes subagent compares the implementation against the spec and the reference (completeness, fidelity, no leaked deps, contract) and writes a gap report before the port is called done.

The skills auto-trigger when the agent sees porting intent and a reference repo path. You don't need to type any slash commands.

## Quickstart

```bash
# 1. Clone the reference repo anywhere you like — outside or inside your project.
git clone https://github.com/<org>/<reference-repo> ~/code/<reference-repo>

# 2. Open your coding agent (Claude Code, Codex, etc.) in your project.

# 3. Ask, naming the reference path:
#    "I want to port the OAuth flow from ~/code/<reference-repo>
#     into src/auth/."
```

Absolute paths, relative paths, and a checkout that happens to live inside your project all work — the plugin treats whatever you point it at as the reference. The agent will engage `reference-analysis` first, then walk you through the rest of the flow.

## Installation

### Claude Code

This repo is a Claude Code plugin marketplace. Add it, then install the plugin:

```bash
/plugin marketplace add duongnghia222/code-distilling
/plugin install code-distilling@code-distilling
```

Or test locally by cloning first:

```bash
git clone https://github.com/duongnghia222/code-distilling
/plugin marketplace add ./code-distilling
/plugin install code-distilling@code-distilling
```

Update later with `/plugin marketplace update code-distilling`.

### Codex

This repo ships a `.codex-plugin/` manifest. Install it through your harness's plugin mechanism pointing at this repo.

### Other harnesses

The plugin uses a standard `SessionStart` hook that emits `additionalContext` JSON. Any harness that supports session-start hooks and additional-context injection can use it. See `hooks/session-start` for the platform-detection logic.

## What you'll see in a session

After installation, the agent loads the `using-code-distilling` bootstrap at session start. The bootstrap is silent until porting intent appears. Then:

```
User: I want to port the OAuth flow from ~/code/awesome-auth into src/auth/.

Agent: I'm using `reference-analysis` to map the reference.
       [reads ~/code/awesome-auth/, locates the OAuth files]
       Here's their design, and here's what I'd copy vs leave behind...
       [writes docs/code-distilling/oauth-flow/reference-analysis.md]
       Please confirm the capability and the copy list before we spec.

User: Looks right.

Agent: I'm using `distillation-spec` to write the spec.
       [writes docs/code-distilling/oauth-flow/distillation-spec.md
        — contract, keep-verbatim list, seam mapping, per-chunk modes]
       Please review the keep/discard split before we plan.

User: Approved.

Agent: I'm using `distillation-plan` to write the implementation plan.
       [writes docs/code-distilling/oauth-flow/distillation-plan.md
        — source→target file map, bite-sized tasks with code]
       Please review the task breakdown before we implement.

User: Approved.

Agent: I'm using `distillation-implementation` to build it.
       [dispatches a fresh subagent per logic-heavy chunk; two-stage review each]
       [implements simple copy-chunks directly]

       I'm using `gap-report` to verify.
       [fresh-eyes subagent compares impl ↔ spec ↔ reference]
       [writes docs/code-distilling/oauth-flow/gap-report.md]
       Verdict: fully distilled — 5 chunks, no gaps.
```

## The skills

| Skill | When it fires | What it produces |
|-------|---------------|------------------|
| `using-code-distilling` | Session start (bootstrap) | Routes to the flow on porting intent |
| `reference-analysis` | Stage 1 — porting intent detected | `reference-analysis.md` (the capability, their design, the copy list) |
| `distillation-spec` | Stage 2 — after the analysis is approved | `distillation-spec.md` (contract, keep-verbatim, discard, seams, per-chunk modes) |
| `distillation-plan` | Stage 3 — after the spec is approved | `distillation-plan.md` (source→target file map, bite-sized tasks with code) |
| `distillation-implementation` | Stage 4 — after the plan is approved | The ported code; subagent-driven with two-stage review |
| `gap-report` | Stage 5 — after implementation | `gap-report.md` (completeness, fidelity, no-leakage, contract) |
| `equivalence-testing` | Opt-in, for testable chunks | Reference-derived RED→GREEN tests |

## Three modes for every chunk

- **copy** — same language, idiomatic for target. Minimal changes (rename imports/types).
- **port** — different language, OR same language with materially different idioms, OR target-incompatible patterns (browser globals in a Node project), OR a library substitution.
- **learn-then-rewrite** — the chunk is heavily entangled with reference-specific infrastructure that doesn't exist in the target; OR the value is the algorithmic approach, not the code itself; OR cross-language translation is so heavy that "porting" is misleading.

Mode is decided per chunk during `distillation-spec`, using explicit criteria documented in `skills/distillation-spec/references/mode-decision-criteria.md`. The deciding criterion is recorded in the spec so reviewers can validate.

## Philosophy

- **The reference is the source of truth.** The encoded decisions worth copying — tuned constants, step order, edge cases, prompts — are kept verbatim, and the result is verified against the reference. Where a chunk is testable, the tests are derived from the reference, not invented.
- **Keep data verbatim, rewrite logic freely.** Control flow and structure get re-expressed in your idioms; the code-as-data gold is copied exactly.
- **Modes are explicit.** Every chunk has a mode and a deciding criterion. "Just trust me" is not a criterion.
- **Subagents per chunk.** Fresh context per logic-heavy chunk keeps the implementer focused. Two-stage review (spec compliance, then code quality) prevents over- or under-building.
- **Verify before done.** A fresh-eyes gap report is the default safety net against a port that runs but is subtly wrong. Executable equivalence tests are the optional addition where the capability supports them.
- **Process over guessing.** Skipping the analysis or the spec produces ports nobody can audit. The flow scales — short docs for small ports — but you still run it.

## Status

Early development.

**v1 is considered ready when:** a single session can take *"I want feature X from `<path-to-reference-repo>`"* through to committed, verified code in the user's project — without manual intervention beyond approving the analysis, the spec, the plan, and the gap report.

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
