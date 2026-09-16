---
name: distillation-implementation
description: Use to build a reference port from an approved distillation spec, or to resume one already in progress — work the spec's task order, keep each task's status current so the work can stop and resume, and verify fidelity against the reference.
---

# Distillation Implementation

Build the port from the approved spec and the reference source. The spec is the design memory and the work plan; the code is yours.

<HARD-GATE>
Start only from work the user has explicitly approved: a spec they reviewed, a round of gap treatments they reviewed, an explicit waiver of spec review, or a small port that qualifies for direct implementation under [using-code-distilling](../using-code-distilling/SKILL.md). A request to port the feature, even "end to end", is not approval of a spec they have not seen; a request to implement or resume a spec they have reviewed is. If approval is missing, stop and use the review gate in [distillation-spec](../distillation-spec/SKILL.md). Nothing here authorizes committing, pushing, publishing, or merging.
</HARD-GATE>

## Pick up the work

Read the spec, the sources it cites at the pinned revision, and the target code around the landing site. A small port needs no spec: implement it from the reference, and stop to ask if a design question appears. If the spec is missing behavior you need, fill that gap with [distillation-spec](../distillation-spec/SKILL.md) first.

On resume, read the task table first, then check it against the actual diff and the checks — a status is a claim, the code is the evidence. Preserve changes the user made since the last session; do not revert work you cannot account for.

## Work the task list

The spec's task table is the todo list, and the spec file is where status lives — a scratch list dies with the session, the table is what the next one reads.

- Mark a task `doing` when you start it, `done` when its check passes, `blocked` with the reason when it cannot proceed. Record the check that proved `done`.
- Finish a task and its check before starting the next, and leave the tree working at every task boundary. A session can end at any point; it should never end mid-task with the status saying otherwise.
- Follow the recommended order unless the work forces a change. Resequencing, splitting, or adding a task is fine — write it into the table with its reason.
- When a discovery invalidates a spec assumption, update the spec's evidence and decision as part of that task. When it changes the contract or scope, stop and ask, and continue the tasks it does not touch meanwhile.

## Implement faithfully

Keep the source and the spec open while editing. Write each asset marked *re-express* fresh in the target's own conventions, then check it against the source at the pinned revision for what the spec says must still hold — comparing runtime values, not literals, where escaping differs between languages. Preserve the invariants the spec names — ordering, state ownership, transition predicates, retry and stop conditions, domain heuristics — even when files, classes, and constructs change around them. Resolve each seam the way the spec decided; renaming a call is not resolving it.

Use native target patterns for incidental structure. Do not import from a checkout outside the project, and do not carry over a dependency merely because the reference uses one. For prompt-driven features, land prompts, context assembly, tool schemas, output parsing, transitions, and stop rules together — a lookalike API over a generic algorithm is not the port. Keep required license and attribution notices wherever reference material survives, and let comments explain invariants rather than syntax.

Resolve ordinary coding choices yourself, and never quietly change the contract to make a check pass. Implement here by default; delegate only a bounded task with no overlapping edits, where that is authorized and useful, and review the combined result yourself.

## Verify

Give each task a check that would fail on a wrong port, then verify the feature as a whole:

- Re-expressed assets still do what the spec says they must: the instruction given, the shape enforced, the strings matched, the value produced.
- Every item in the spec's *Watch out* list is specifically checked.
- Seams behave, not just compile: ordering, filtering, units, retries, state lifetime, cancellation.
- For agent workflows, replay controlled model and tool outputs to test routing, parsing, transitions, and termination deterministically; keep that separate from any live quality evaluation.
- Discarded packaging has not leaked into the target, and no essential behavior left with it.
- The target's tests, build, and type checks pass, and the integrated feature path runs where feasible.

A green build is not evidence of behavioral parity.

## Finish

The port is done when every task is `done`, `declined`, or `deferred` with a recorded reason — not when the first working slice runs. Update the spec with material discoveries, deliberate deviations, and anything left undone, so it describes what the target actually has.

Report what was ported, the spec path (for a small port, the reference path and revision), tasks completed, checks and their results, and every unresolved difference or verification limit. Do not claim parity for behavior you could not check.
