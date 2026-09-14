---
name: using-code-distilling
description: Use when the user wants to port, distill, copy, borrow, or bring a feature from a reference repository into their project, or to catch an existing port up with its reference or close its remaining gaps.
---

# Using Code Distilling

Port the decisions that make a reference feature work: algorithms, architecture, prompts, workflow, domain rules, and edge cases. Re-express them in the target project's conventions while preserving the behavior that matters.

Use this workflow for reference-based feature adoption, including keeping an existing port current with its reference, not unrelated debugging, refactoring, or original development. Establish the reference and target from the request and workspace; ask for a missing reference rather than inventing one.

## Small ports: implement directly

Use the full flow only when the port needs it. A port is small when all of these hold:

- The reference behavior is a bounded unit you can read completely: a function, helper, regex, constant table, validation rule, small component, or localized fix. Workflows, multi-component features, stateful or concurrent logic, and prompt-driven or agent behavior are not small.
- It lands in one place in the target through existing interfaces, without new dependencies or architectural change.
- There is no design question: scope is clear, and no integration difference changes observable behavior.

For a small port, skip the design conversation, spec document, and review gate:

1. Read the reference unit, the dependencies that determine its behavior (defaults, constants, tests), and where it lands in the target.
2. Say in one line that this is a small port you are implementing directly, then do it without waiting.
3. Preserve exact values and invariants, adapt incidental structure to target conventions, and keep required license and attribution notices.
4. Run a check that would catch a wrong port, then report the reference path and revision, what was preserved or adapted, the checks, and any limits.

If a design question appears — a scope choice, a behavior-changing seam difference, a new dependency, or more code than it looked — stop and ask before continuing. If the port is no longer small, switch to the full flow below. If the user asks for a spec or a review, use the full flow regardless of size.

## Spec → review → implementation

1. Read [distillation-spec](../distillation-spec/SKILL.md). Inspect the reference and target, explain the design to the user and ask the questions that shape the port, then write `docs/code-distilling/<capability>/distillation-spec.md`: the behavioral contract and the design knowledge needed to reproduce it.
2. Stop for review. Present the spec with your recommendations and open questions, and wait for the user's explicit approval. On feedback, revise and present it again.
3. Read [distillation-implementation](../distillation-implementation/SKILL.md). Implement directly from the approved spec and the source, verify fidelity and integration, and report the result.

<HARD-GATE>
Do not write port code or start implementation until the user has reviewed and explicitly approved the spec. A request to port the feature, even "end to end", is not approval of a spec the user has not seen. Only an explicit instruction to skip spec review waives this gate. This applies to every port that is not small under the criteria above; a port with any design question is not small.
</HARD-GATE>

There is no separate plan stage, task document, or requirement to prewrite implementation code. The spec records durable decisions and evidence; the implementer chooses files, sequence, and local tactics as work develops. A working checklist is optional.

If the user requested only analysis or a spec, deliver that and stop. Ask whenever a choice changes scope or observable behavior; routine implementation decisions are yours. While waiting for an answer, continue investigating, but do not edit target code.

## Existing ports: close the gap

When a port already exists and the reference has changed since it was distilled, or the port still falls short of its spec or reference, read [distillation-gap](../distillation-gap/SKILL.md) instead of writing a new spec from scratch. It inventories gaps against the recorded spec and reference revision, takes decisions that change the spec through the same review gate, and closes approved gaps with distillation-implementation.

## Scale to the feature

When a port goes through the spec, size the spec to it. A bounded feature may need a paragraph with source, invariants, exact values, target integration, and checks. A workflow feature needs enough architecture and domain context that another agent can reproduce its decisions without the original conversation. Add linked design notes or exact assets only when useful; do not manufacture empty sections.

For AI features, inspect prompts and the code that assembles context, selects tools or speakers, routes results, retries, and stops. A similar UI or a copied system prompt alone does not establish feature fidelity.

## Harness portability

Use the host's skill-loading mechanism if available; otherwise read the linked `SKILL.md` files directly. Paths are relative to their containing file. The workflow requires source inspection, file editing, and suitable verification, not a particular Skill, Task, Todo, or review tool. Work in the current session by default. Delegation is optional when available and authorized.

Existing `distillation-plan.md` artifacts may contain useful decisions. Reconcile those with the current spec and code when resuming an old port; do not require or regenerate a plan.
