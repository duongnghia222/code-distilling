# code-distilling

> Port high-quality implementations from reference open-source repos into your project — with discipline.

`code-distilling` is a Claude Code and Codex plugin that turns *"I want to copy this feature from that repo"* into a controlled workflow: explore the reference and write a distillation spec (what to keep verbatim, what to discard, how the seams wire into your project), and implement it.

It is a sister-plugin to [Superpowers](https://github.com/obra/superpowers) and follows the same skill-driven discipline. You do not need Superpowers installed to use it.

## Why this exists

When you find a great open-source implementation of something you need — a parser, an auth flow, a search algorithm — there are three failure modes:

1. **Paste-and-pray.** Copy the code in, rename a few symbols, hope it works. No idea what you brought along for the ride.
2. **From-scratch-with-vibes.** Read the reference, close the tab, write your own. Lose all the edge cases the original handled.
3. **Vendor-and-forget.** Add the repo as a dependency, drag in their build system, never actually understand what's running.

`code-distilling` is the disciplined fourth option: **port a curated subset of the reference into your own codebase — keeping the encoded decisions that make it good.**

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

Then ask your coding agent to port a feature, naming the reference's path. The workflow has two stages:

1. **`distillation-spec`** — traces the reference and target, explains the reference's design, and asks you the questions that shape the port: scope, what to preserve or adapt, and how semantic differences at integration seams are resolved. It then records the contract, architecture, workflow, domain heuristics, prompt/skill design, exact assets, behavioral invariants, integration differences, and acceptance cases. The spec contains the knowledge another agent needs to reproduce the feature.
2. **`distillation-implementation`** — builds directly from the approved spec and source, choosing implementation details as work develops. It verifies fidelity and target integration. Delegation is optional; there is no separate plan document or mandatory task approval.

Between them is a review gate: the agent stops with the spec, its recommendations, and any open questions, revises on your feedback, and writes no port code until you approve it. Asking for a port "end to end" does not skip the gate; only an explicit instruction to skip spec review does. A spec-only request stops at the spec.

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

Absolute paths, relative paths, and a checkout that happens to live inside your project all work — the plugin treats whatever you point it at as the reference. The agent will engage `distillation-spec` first, then walk you through the rest of the flow.

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

The skill instructions use ordinary Markdown and do not require named tools or subagents. A coding CLI, including GitHub Copilot, can follow them when its environment makes the skills available; start with `skills/using-code-distilling/SKILL.md`. Automatic discovery depends on the harness integration. See `hooks/session-start` for the existing context-output branches; portability of the instructions is not a claim of end-to-end testing on every CLI.

## What you'll see in a session

After installation, the agent loads the `using-code-distilling` bootstrap at session start. The bootstrap is silent until porting intent appears. Then:

```
User: I want to port the OAuth flow from ~/code/awesome-auth into src/auth/.

Agent: I'm using `distillation-spec` to map the reference and write the spec.
       [reads ~/code/awesome-auth/ and src/auth/, locates the OAuth files]
       Here's how their flow works and why: [core path, key mechanisms].
       First question: their session store expires entries itself; ours
       relies on a TTL. Adapt the port, adapt our store, or accept the
       difference? I recommend adapting the port because [evidence].

User: Adapt the port.

Agent: [asks the remaining design questions, one at a time]
       [writes docs/code-distilling/oauth-flow/distillation-spec.md
        — contract, design decisions, state transitions, integration, checks]
       Spec written. Please review it before I implement. Open question: [...]

User: Approved.

Agent: I'm using `distillation-implementation` to build and verify it.
       [ports the behavior, checks callback/error paths and target integration]
       The port is implemented. Here are the checks and any remaining limitations.
```

## The skills

| Skill | When it fires | What it produces |
|-------|---------------|------------------|
| `using-code-distilling` | Session start (bootstrap) | Routes to the flow on porting intent |
| `distillation-spec` | Reference feature needs analysis | `distillation-spec.md`, with linked design notes/assets when useful |
| `distillation-implementation` | Spec is approved and implementation is requested | Ported code and fidelity/integration verification |

## What survives the port

- **Exact assets:** literal values, prompt text, schemas, examples, and tables identified for exact preservation.
- **Behavioral invariants:** algorithms, state ownership, transition predicates, ordering, domain rules, and termination. Syntax and incidental structure can change.
- **Documented adaptations:** target interfaces and omitted packaging, with evidence of semantic differences and how they are resolved.

For deep research, the design may include evidence tracking, follow-up search, and citation binding. For group chat, it may include speaker eligibility, context visibility, and reply-loop prevention. The spec traces the actual reference mechanisms; it does not impose these examples on every feature.

The implementer retains discretion over local coding decisions. The spec preserves the knowledge that would otherwise be lost between sessions. Required license and attribution notices remain in copied material.

## Status

Early development.

**Acceptance target:** a session can take *"I want feature X from `<path-to-reference-repo>`"* through a design conversation and a source-grounded spec you approve to implemented and verified code, writing no port code before that approval. Commits and publishing require user authorization.

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
