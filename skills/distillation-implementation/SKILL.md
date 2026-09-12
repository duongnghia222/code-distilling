---
name: distillation-implementation
description: Use when implementing or resuming a reference-based feature port from a distillation spec, preserving its behavioral and design decisions in the target project.
---

# Distillation Implementation

Build directly from the distillation spec and reference source. Use your judgment to choose implementation order, file boundaries, and local tactics. The spec is the durable contract and design memory; no separate implementation plan is required.

## Establish context

Read the spec, relevant linked design notes/assets, reference locations, and target code. If the spec is absent or missing behavior-critical knowledge, use [distillation-spec](../distillation-spec/SKILL.md) to fill that gap first. On resume, inspect the current diff and checks before assuming what is complete. Preserve existing user changes.

Start only from a spec the user has explicitly approved, or after they explicitly waived spec review. A request to port the feature, even "end to end", is not approval of a spec they have not seen; a request to implement or resume a spec they have reviewed is. If approval is missing, stop and use the review gate in [distillation-spec](../distillation-spec/SKILL.md). Do not infer permission to commit, push, publish, or merge from this skill.

## Implement and learn

Choose a coherent slice that exercises the feature's essential path and target integration, then extend it through the remaining contract. For small ports, implement the whole feature directly. Keep a short working checklist only if useful; do not prewrite complete code in a document or require fixed-duration tasks.

Keep source and spec available while editing. Preserve exact assets at their required fidelity, and preserve behavioral invariants even when classes, files, or language constructs change. Adapt target interfaces explicitly; do not import a checkout outside the project or carry over dependencies merely because the reference uses them. Dependencies justified by the spec and target conventions are acceptable.

For prompt-driven features, implement context construction, tool/output contracts, controller transitions, and stopping rules together with prompt assets. For domain-heavy features, carry over relevant heuristics and boundary behavior. A lookalike API with a generic replacement algorithm does not satisfy the port.

Use native target patterns for incidental structure. If investigation invalidates a spec assumption, update its evidence and adaptation notes. Resolve routine implementation details autonomously; ask when the resolution would change requested behavior or expand scope. Continue unaffected work. Never silently change the contract to make a failing check pass.

Keep detailed provenance in the spec and retain required license/attribution notices in copied material. Comments should explain non-obvious behavior and invariants.

## Verify fidelity and integration

Select checks that can expose a wrong port, proportional to the change:

- Compare exact constants, templates, schemas, and other preserved assets, accounting for source-language representation.
- Exercise contract cases and important alternate/failure paths against the reference where runnable; otherwise use source-grounded fixtures and state the limitation.
- Check semantic seams, including ordering, filtering, units, retries, state lifetime, and cancellation where relevant.
- For agent workflows, replay controlled model/tool outputs to verify decision rules and transitions. Distinguish those checks from any live quality evaluation.
- Run relevant target tests/build/type checks and an integrated feature path where feasible. Check maintainability and unintended scope changes as well as fidelity.

Use [spec-reviewer-prompt.md](spec-reviewer-prompt.md) and [code-quality-reviewer-prompt.md](code-quality-reviewer-prompt.md) as review lenses when useful, in the current session or with an authorized reviewer. Fix material findings and rerun affected checks. A successful build alone is not evidence of behavioral parity.

## Optional delegation

Default to implementing in the current session. If delegation is available, authorized, and useful for a bounded independent slice, use [implementer-prompt.md](implementer-prompt.md). Supply relevant spec decisions, source locations, target interfaces, acceptance cases, and the edit boundary. Pass accessible file paths or include content if the worker cannot read those paths. Avoid overlapping edits; review the combined integration. Do not require a particular tool name, model tier, or agent topology.

## Finish

Complete the authorized capability, not merely the first working slice. Update the spec with material discoveries and deliberate deviations so it remains useful to another agent. Report what was ported, the spec path, checks and results, and unresolved differences or verification limits. Do not claim parity for behavior you could not check.
