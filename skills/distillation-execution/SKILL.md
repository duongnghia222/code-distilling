---
name: distillation-execution
description: Use to execute an approved distillation plan - dispatches a fresh implementer subagent per task following equivalence-tdd, with two-stage review (spec compliance first, then code quality), and a final attribution-and-license pass
---

# Distillation Execution

Execute the distillation plan by dispatching a fresh implementer subagent per task, with tightly-scoped context. Run two-stage review after each task: **spec compliance first, then code quality**. Finish with the `attribution-and-license` final pass.

**Why subagents:** delegating tasks to specialized subagents with isolated context keeps them focused. You curate exactly what context they need; they should not inherit your session history. This preserves your context for coordination and review work.

**Core principle:** Fresh subagent per task + two-stage review (spec then quality) = high-quality ports, fast iteration.

**Continuous execution:** Do not pause to check in with your human partner between tasks. Execute every task from the plan without stopping. The only reasons to stop: BLOCKED status you cannot resolve, ambiguity that genuinely prevents progress, or all tasks complete. "Should I continue?" prompts and progress summaries waste time — they asked you to execute the plan, so execute it.

**Announce at start:** "I'm using `distillation-execution` to execute the plan with subagent-driven dispatch."

## When to Use

```dot
digraph when_execute {
    "Plan approved?" [shape=diamond];
    "Tasks mostly independent?" [shape=diamond];
    "Stay in this session?" [shape=diamond];
    "distillation-execution" [shape=box];
    "Manual execution" [shape=box];
    "Run distillation-plan first" [shape=box];

    "Plan approved?" -> "Tasks mostly independent?" [label="yes"];
    "Plan approved?" -> "Run distillation-plan first" [label="no"];
    "Tasks mostly independent?" -> "Stay in this session?" [label="yes"];
    "Tasks mostly independent?" -> "Manual execution" [label="no — tightly coupled"];
    "Stay in this session?" -> "distillation-execution" [label="yes"];
    "Stay in this session?" -> "Manual execution" [label="no"];
}
```

## Branch Safety

**Never start execution on `main` / `master` without explicit user consent.** If the current branch is the project's main branch, ask the user to confirm or to switch to a feature branch (e.g., `distill/<repo>-<feature-slug>`) first. **No exceptions.**

## The Process

```dot
digraph process {
    rankdir=TB;

    "Read plan once, extract every task with full text + curated context" [shape=box];
    "Run license compatibility recheck" [shape=box];
    "Create TaskCreate entries per plan task" [shape=box];

    subgraph cluster_per_task {
        label="Per Task";
        "Dispatch implementer subagent (./implementer-prompt.md)" [shape=box];
        "Implementer asks questions?" [shape=diamond];
        "Answer questions, re-dispatch" [shape=box];
        "Implementer commits + self-reviews" [shape=box];
        "Dispatch spec compliance reviewer (./spec-reviewer-prompt.md)" [shape=box];
        "Spec reviewer approves?" [shape=diamond];
        "Implementer fixes spec gaps" [shape=box];
        "Dispatch code quality reviewer (./code-quality-reviewer-prompt.md)" [shape=box];
        "Code reviewer approves?" [shape=diamond];
        "Implementer fixes quality issues" [shape=box];
        "Mark task complete" [shape=box];
    }

    "More tasks remain?" [shape=diamond];
    "Dispatch final code reviewer (whole branch)" [shape=box];
    "Run attribution-and-license final pass" [shape=box];
    "Done" [shape=doublecircle];

    "Read plan once, extract every task with full text + curated context" -> "Run license compatibility recheck";
    "Run license compatibility recheck" -> "Create TaskCreate entries per plan task";
    "Create TaskCreate entries per plan task" -> "More tasks remain?";
    "More tasks remain?" -> "Dispatch implementer subagent (./implementer-prompt.md)" [label="yes"];
    "Dispatch implementer subagent (./implementer-prompt.md)" -> "Implementer asks questions?";
    "Implementer asks questions?" -> "Answer questions, re-dispatch" [label="yes"];
    "Answer questions, re-dispatch" -> "Dispatch implementer subagent (./implementer-prompt.md)";
    "Implementer asks questions?" -> "Implementer commits + self-reviews" [label="no"];
    "Implementer commits + self-reviews" -> "Dispatch spec compliance reviewer (./spec-reviewer-prompt.md)";
    "Dispatch spec compliance reviewer (./spec-reviewer-prompt.md)" -> "Spec reviewer approves?";
    "Spec reviewer approves?" -> "Implementer fixes spec gaps" [label="no"];
    "Implementer fixes spec gaps" -> "Dispatch spec compliance reviewer (./spec-reviewer-prompt.md)";
    "Spec reviewer approves?" -> "Dispatch code quality reviewer (./code-quality-reviewer-prompt.md)" [label="yes"];
    "Dispatch code quality reviewer (./code-quality-reviewer-prompt.md)" -> "Code reviewer approves?";
    "Code reviewer approves?" -> "Implementer fixes quality issues" [label="no"];
    "Implementer fixes quality issues" -> "Dispatch code quality reviewer (./code-quality-reviewer-prompt.md)";
    "Code reviewer approves?" -> "Mark task complete" [label="yes"];
    "Mark task complete" -> "More tasks remain?";
    "More tasks remain?" -> "Dispatch final code reviewer (whole branch)" [label="no"];
    "Dispatch final code reviewer (whole branch)" -> "Run attribution-and-license final pass";
    "Run attribution-and-license final pass" -> "Done";
}
```

## Steps

1. **Read the plan once.** Extract every task with its full text and required context: source paths to read inside `ref-code/`, target paths, mode, adaptation notes from the spec, the test source (path or captured cases).
2. **License recheck.** Confirm the spec's compatibility result is still valid (target license hasn't changed; reference commit hash matches the reference map). If anything drifted, halt and escalate.
3. **Create `TaskCreate` entries** — one per plan task (test tasks and impl tasks both).
4. **For each task in order:**
   a. Dispatch the implementer subagent using `implementer-prompt.md`. Provide the task text, the curated context, and a hard requirement to follow `equivalence-tdd`.
   b. If the implementer asks questions, answer clearly and completely before letting them proceed.
   c. After the implementer commits, dispatch the spec compliance reviewer using `spec-reviewer-prompt.md`. Re-dispatch the implementer with the reviewer's findings until the reviewer approves.
   d. Dispatch the code quality reviewer using `code-quality-reviewer-prompt.md`. Re-dispatch the implementer with the reviewer's findings until the reviewer approves.
   e. Mark the task complete.
5. **After all tasks:** dispatch the final code reviewer for the entire distillation diff against the branch base.
6. **Run the `attribution-and-license` final pass.** This generates/updates `ATTRIBUTION.md`, copies source licenses to `licenses/`, verifies every distilled file has a header, and verifies every commit has a `Source:` or `Source-influence:` trailer.
7. **Announce completion** with a summary: chunks distilled, modes used, tests passing, attribution status.

## Curated Subagent Context

Implementer subagents should receive, per task:

- The full task text from the plan (**verbatim** — they do not read the plan file).
- The spec's row for this chunk (modes table + adaptation notes).
- The reference-map excerpt naming the file's transitive deps and hidden coupling.
- The exact source file content from `ref-code/<repo>/<path>` (for copy/port; **omit for learn-then-rewrite** to enforce independence).
- The attribution header template filled with values (so they paste, not invent).
- The exact test command to run and what success/failure looks like.

Do **not** dump the entire spec, plan, or reference map. Curate.

## Model Selection

Use the least powerful model that can handle each role to conserve cost and increase speed.

| Task | Suggested model |
|------|-----------------|
| copy chunk, ≤2 files, no library substitution | cheap |
| port chunk, same language, no library substitution | cheap or standard |
| port chunk, cross-language or with substitution | standard |
| learn-then-rewrite | most capable |
| spec compliance reviewer | standard |
| code quality reviewer | standard or most capable |
| final code reviewer (whole diff) | most capable |

**Task complexity signals:**

- Touches 1–2 files with a complete spec row → cheap model.
- Multiple files, cross-language, or library substitution → standard.
- learn-then-rewrite, reviewer, or final review → most capable.

## Handling Implementer Status

Implementer subagents return one of four statuses. Handle each appropriately.

**DONE** — proceed to spec compliance review.

**DONE_WITH_CONCERNS** — read the concerns before proceeding.
- Concerns about correctness or scope → address before review.
- Observations ("this file is getting large") → note and proceed to review.

**NEEDS_CONTEXT** — the implementer needs information that wasn't provided. Provide it and re-dispatch.

**BLOCKED** — the implementer cannot complete the task. Diagnose:
1. Context problem → provide more context, re-dispatch same model.
2. Reasoning required → re-dispatch with more capable model.
3. Task too large → break into smaller tasks (amend the plan first).
4. Plan is wrong → escalate to the user.

**Never** ignore an escalation. **Never** force the same model to retry without changing something.

## Prompt Templates

- `./implementer-prompt.md` — dispatch implementer subagent
- `./spec-reviewer-prompt.md` — dispatch spec compliance reviewer subagent
- `./code-quality-reviewer-prompt.md` — dispatch code quality reviewer subagent

## Red Flags - BANNED

**Never:**

- Start execution on `main` / `master` without explicit user consent.
- Skip either review stage (spec compliance OR code quality).
- Proceed with unfixed issues from either reviewer.
- Dispatch multiple implementer subagents in parallel (conflicts on shared files).
- Make the subagent read the plan file (provide curated full text instead).
- Skip the scene-setting context for a task (subagent needs to know where it fits).
- Ignore subagent questions before they implement.
- Accept "close enough" on spec compliance.
- **Start code quality review before spec compliance is approved** (wrong order).
- Move to the next task while either review has open issues.
- Skip the final `attribution-and-license` pass.
- Let the implementer pull lines from the reference for a `learn-then-rewrite` chunk.

**If the subagent asks questions:** answer clearly and completely. Don't rush them.

**If a reviewer finds issues:** the same implementer subagent fixes them. The reviewer reviews again. Repeat until approved. Don't skip the re-review.

**If a subagent fails the task:** dispatch a fresh implementer with specific instructions. Don't try to fix manually (context pollution).

## Example Workflow

```
You: I'm using distillation-execution to execute the plan with subagent-driven dispatch.

[Read plan once: docs/plans/2026-05-26-distill-awesome-auth-oauth.md]
[Extract all 5 chunks → 10 tasks (test + impl) + Final Task F]
[License recheck: COMPATIBLE — proceed]
[Create TaskCreate with all tasks]

Task 1.t: Port test for src/cache/lru.ts
[Dispatch implementer with task text + curated context]
Implementer: DONE. Commit abc1234.

[Dispatch spec compliance reviewer]
Spec reviewer: APPROVED.

[Dispatch code quality reviewer]
Code reviewer: APPROVED.

[Mark Task 1.t complete]

Task 1.i: Port implementation for src/cache/lru.ts
[Dispatch implementer with task text + source code + header template]
Implementer: DONE. Commit def5678.

[Dispatch spec compliance reviewer]
Spec reviewer: ISSUES — adaptation notes say "rename LRU to LruCache" but the file still uses `LRU`.

[Re-dispatch implementer with the issue]
Implementer: Fixed. Commit ghi9012.

[Re-dispatch spec reviewer]
Spec reviewer: APPROVED.

[Dispatch code quality reviewer]
Code reviewer: ISSUES (Important) — magic number 1024 should be a constant.

[Re-dispatch implementer]
Implementer: Extracted DEFAULT_CAPACITY. Commit jkl3456.

[Re-dispatch code reviewer]
Code reviewer: APPROVED.

[Mark Task 1.i complete]

... [Tasks 2 through 5 similarly] ...

[After all tasks]
[Dispatch final code reviewer for whole branch diff]
Final reviewer: APPROVED.

[Run attribution-and-license final pass]
- ATTRIBUTION.md updated with 5 distilled files
- licenses/awesome-auth-MIT.txt copied
- All commits verified to have Source: trailer

Done!
```

## Completion

Execution is done when:

- Every plan task is complete and both reviewers approved it.
- The final code reviewer has approved the whole diff.
- The `attribution-and-license` final pass has committed.
- A short summary has been posted to the user with: chunks distilled, modes used, tests passing, attribution status, any concerns to follow up on.

## Required Sub-Skills

- `code-distilling:equivalence-tdd` — every implementer subagent follows this.
- `code-distilling:attribution-and-license` — invoked at start (recheck) and end (final pass).

## Key Principles

- **Fresh subagent per task.** No context pollution between tasks.
- **Two-stage review, in order.** Spec compliance gates code quality.
- **Continuous execution.** No "should I continue?" prompts.
- **Curate context.** Don't dump the plan; quote what's needed.
- **Cheapest model that works.** Reviewers and learn-then-rewrite get the most capable.
