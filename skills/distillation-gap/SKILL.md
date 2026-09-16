---
name: distillation-gap
description: Use to find and log the gaps between a feature and its reference — a port that has fallen behind an upstream that moved on, a clone where both sides have changed, or two versions of the same feature that were never formally ported — then decide each gap with the user and close the approved ones.
---

# Distillation Gap

Log the distance between what the target has and what the reference has, decide each gap with the user, and close the approved ones. This runs against anything that has a reference: a port made from a spec, a fork or clone that has drifted, or a feature someone built separately from the same idea. Most of the time there is no spec — the ledger is then the only record, and it is the deliverable even when nothing gets closed, because it is what the next round starts from. The goal is a target that is faithful where it means to be and deliberately different everywhere else, not one that mirrors every upstream commit.

<HARD-GATE>
Do not change target code for a gap until the user has reviewed its treatment in the ledger and explicitly approved it, whenever that treatment changes behavior, scope, or a decision the target already embodies — adopting, adapting, or declining a reference change; adding, changing, or removing a requirement; revising a seam resolution; or reversing a deliberate local change. Where a spec exists, they review the updated spec alongside the ledger. A request to "sync", "catch up", or "close the gaps" is not approval of treatments the user has not seen. Only an explicit instruction to skip review, or a small catch-up as defined under "Converge and review", waives this gate.
</HARD-GATE>

## Name the situation first

It decides what you compare and what counts as evidence:

- **The port went stale.** Upstream moved, the target did not. Compare the recorded base revision to the revision the user names, or the reference's current default branch.
- **Both sides moved.** The clone drifted locally while upstream changed. Review both the upstream range and the target's own history since it diverged, and sort every gap into upstream-only, target-only, or *both changed the same thing* — a collision you reconcile explicitly, because upstream may have solved the same problem a different way.
- **No shared history.** Two implementations of the same capability that were never formally ported, or a target with no spec and no recoverable base. Compare design and behavior directly, capability by capability, in both directions.

Say which one you are in and what evidence you have. If the base revision is missing, infer it — the history that introduces or removes the assets the target carries, the date the target's version landed, matching code — and record the inference with its confidence. If no base can be established, compare the target's design against the current reference directly and say so; never present a guessed diff as a complete range review.

## Establish the baseline

The baseline is whatever records what the target meant to be: a spec at `docs/code-distilling/<capability>/distillation-spec.md` with its notes and any earlier ledger, if one exists. Usually none does — then the baseline is the target code itself plus what the user can tell you about which differences are deliberate, and you ask rather than assume. Either way, read the current target code and its checks: it may have moved through fixes, refactors, or drift, so work from what is there and preserve it.

Do not reconstruct a spec merely to have something to compare against. Write one with [distillation-spec](../distillation-spec/SKILL.md) only when the user wants the target governed by it from here on, and then pass its review gate.

Prefer reading revisions — fetching and diffing refs — over moving the checked-out tree, and never discard local modifications in the reference checkout without permission.

## Find the gaps

**Upstream.** Review the range for everything that can affect the capability, not only the paths you already know about: renamed files and moved symbols, new code wired into the feature's path, configuration defaults, prompts, templates, schemas, parsers, library and model versions, tests, and fixtures. Changed test expectations are strong evidence of intended behavior change. Group changes by intent, not by file — a prompt revision and the parser, tool schema, or model change that came with it are one change, and adopting half breaks both. Commit messages, pull requests, and changelogs are material to analyze, then confirm in code; do not triage a range by commit titles. If the range is too large to read fully, state how you bounded the review and what remains unreviewed.

Trace each group against what the target intends — the spec where there is one, otherwise the behavior the target clearly relies on: a behavior-neutral refactor changes nothing but citations; a changed asset or preserved behavior means the target is stale on something it meant to keep; a changed seam means re-checking whether the target's adaptation still reproduces the reference contract; changed packaging usually means nothing, unless it now carries behavior the capability needs; behavior added, removed, or reverted is a scope decision. For prompt-driven features, use the lenses in [behavioral-design.md](../distillation-spec/references/behavioral-design.md).

**Target.** Classify every local change the target has accumulated: a deliberate adaptation — recorded in the spec, or one the user confirms — which is not a gap; a local fix or feature upstream lacks, which is a decision about whether to keep it and a deviation to record either way; or drift, an invariant broken, an asset edited, a heuristic dropped, which is a gap. Where a local change and an upstream change touch the same thing, reconcile them explicitly; never overwrite a deliberate local change with an upstream one silently.

**Target against its own intent.** Behavior it was supposed to have and does not, assets that no longer do what they must, invariants broken, seams resolved in name only, and checks never run or unable to fail on a wrong implementation — plus any limits recorded when the work last finished. Run the checks instead of trusting an earlier report.

**Unrecorded design.** Reference behavior the target depends on that nothing records, often surfaced by a divergence found in use. Capture it to [distillation-spec](../distillation-spec/SKILL.md)'s evidence standard — in the spec if there is one, in the ledger otherwise — before implementing against it.

Decisions already made — non-goals, declined changes, accepted differences, deferrals, whether recorded in a spec, an earlier ledger, or by the user just now — are not gaps. Reopen one only when new evidence changes its basis, and then as a question.

## Log the ledger

Write or extend `docs/code-distilling/<capability>/distillation-gaps.md`. Open each round with its date, the situation, the range or comparison used, and the target state examined. Per gap:

- ID and kind: upstream, target drift, collision, missing behavior, verification, or unrecorded design.
- Evidence: revisions, paths, symbols, commits, the requirement it violates, the failing or missing check.
- The decisions it affects and the target locations it touches.
- Recommended treatment — adopt exactly, adopt behavior, adapt, decline, defer, or citations only — and its observable effect.
- The user's decision, the status, and the check that shows it closed.

Scale entries to the gap: a reviewed range with no capability changes is one line. The ledger is what stops the next round re-reviewing a range or re-raising a declined change.

## Converge and review

Explain the material gaps plainly. Ask only where the decision changes scope or observable behavior, one at a time, most consequential first, with concrete options, your recommendation, and the evidence — framing a changed seam as **adapt the port**, **adapt the target**, or **accept** the difference. Gaps that only implement or verify what the user has already approved need no question. Record every answer where the target's decisions live — the spec if there is one, the ledger otherwise — including declined upstream changes, then present the round and stop:

> "Gap ledger written to `<path>`. Please review — are these the right gaps, and are the proposed treatments correct before I change the target?"

In the same message, summarize what you propose to adopt, decline, and defer, the behavior changes the target will show, and the open questions with your recommended resolutions. Wait for explicit approval; answers to some questions are not approval of the round.

Two exceptions. If every gap only implements or verifies what the user already approved and they asked to close them, that approval covers the work — share the ledger and proceed. And a small catch-up may skip the ledger and review when the user asked to catch up, the changes are few and localized, and each is an unambiguous adoption with no design question, such as an upstream bug fix or a retuned value the target means to keep exactly: say in one line what you are adopting, implement it with a check, update the citations and recorded revision, and report. Declining, deferring, or adapting anything, or touching scope or a seam resolution, still goes through review.

## Close the gaps

Turn the approved treatments into tasks with dependencies and status — appended to the spec's task table where there is one, kept in the ledger with the same columns where there is not — so a round can stop and resume like any other work, then implement them with [distillation-implementation](../distillation-implementation/SKILL.md).

Re-express reference changes through the target's own adaptations; do not apply the upstream diff as a patch, because those adaptations change what the same edit means. Land grouped changes together. Give each closed gap a check that fails before the change and passes after it where feasible — upstream's changed tests and fixtures are good sources — and rerun the target's existing checks, since adopting one change can break a preserved invariant elsewhere. Never close a gap by weakening a check or narrowing the contract without a recorded decision.

## Finish

Update the ledger statuses. Record the reference revision reconciled through — in the spec if there is one, in the ledger otherwise — advancing it to the end of the reviewed range once every gap from that range is closed, declined, or deferred with a recorded decision. It means "reconciled through", not "identical to".

Report the range or comparison reviewed, gaps closed with their checks, declined and deferred items, the behavior changes the target now shows, and what remains open. Do not claim the target is current with changes you did not review, or a gap closed that you could not check.
