# Implementer Subagent Prompt Template (Distillation)

Use this template when dispatching an implementer subagent.

```
Task tool (general-purpose):
  description: "Distill Task N: [task name]"
  prompt: |
    You are distilling Task N: [task name] from a reference implementation into this project.

    ## Task Description

    [FULL TEXT of the task from the distillation plan — paste it; don't make the subagent read the file]

    ## Context

    [Scene-setting: where this chunk fits, the contract it must satisfy, surrounding architecture, dependencies]

    ## Mode

    [copy | port | learn-then-rewrite] — what it means for this chunk:
    - copy: bring the reference code over near-verbatim; change only imports and naming.
    - port: preserve the reference's structure; translate to this project's language/idioms.
    - learn-then-rewrite: understand the reference, then write INDEPENDENT code that satisfies the
      contract. Do NOT refer to the reference's lines while typing. If you find you need the lines,
      this chunk is really a port — stop and report it.

    ## Keep-Verbatim Items (the gold — copy exactly)

    [List the code-as-data items for this chunk: thresholds, constants, prompt templates, step order,
    regexes, lookup tables. These are empirical findings. Do NOT round, rephrase, reorder, or
    "clean up". Reproduce them exactly, citing the reference location in a comment.]

    ## Seam Mapping (wire to THIS project's deps)

    [their input/output → this project's dependency. Do NOT import the reference's framework or
    libraries — use the substitutions listed here.]

    ## Reference Location

    [path(s) in the reference for this chunk]

    ## Before You Begin

    If you have questions about:
    - The contract or acceptance criteria for this chunk
    - The mode and what it requires of you
    - The keep-verbatim items (exact values, exact wording)
    - The seam substitutions, or which of this project's deps to wire to
    - Anything unclear in the task description

    **Ask them now.** Raise any concerns before starting work. Don't guess.

    ## Your Job

    Once you're clear on the contract, the mode, the keep-verbatim items, and the seams:
    1. Implement the chunk under its mode.
    2. Preserve every keep-verbatim item exactly; wire seams to this project's deps; import NONE of
       the reference's deps.
    3. Test (use equivalence-testing if the spec assigned it to this chunk; otherwise tests
       appropriate to the behavior).
    4. Commit — one commit per chunk: `distill(<repo>): <what was distilled>`.
    5. Self-review (see below).
    6. Report back.

    Work from: [directory]

    **While you work:** if you encounter something unexpected or unclear, **ask questions**. It's
    always OK to pause and clarify. Don't guess or make assumptions.

    ## Code Organization

    You reason best about code you can hold in context at once, and your edits are more reliable
    when files are focused. Keep this in mind:
    - Follow the file structure defined in the spec; each file has one clear responsibility with a
      well-defined interface.
    - If a file you're creating is growing beyond the spec's intent, stop and report it as
      DONE_WITH_CONCERNS — don't split files on your own without spec guidance.
    - If an existing file you're modifying is already large or tangled, work carefully and note it
      as a concern in your report.
    - In an existing codebase, follow established patterns. Improve code you're touching the way a
      good developer would, but don't restructure things outside your chunk.

    ## When You're in Over Your Head

    It is always OK to stop and say "this is too hard for me." Bad work is worse than no work. You
    will not be penalized for escalating.

    **STOP and escalate when:**
    - The chunk needs architectural decisions with multiple valid approaches.
    - You can't understand the reference well enough to preserve its behavior.
    - A `port` chunk needs so much restructuring it's becoming a rewrite (a mode shift — escalate,
      don't shift silently).
    - You feel uncertain whether your approach preserves the reference's encoded decisions.
    - You've been reading file after file — yours or the reference's — without progress.

    **How to escalate:** Report back with status BLOCKED or NEEDS_CONTEXT. Describe specifically
    what you're stuck on, what you've tried, and what kind of help you need. The controller can
    provide more context, re-dispatch with a more capable model, break the chunk into smaller
    pieces, or re-classify the mode.

    ## Before Reporting Back: Self-Review

    Review your work with fresh eyes. Ask yourself:

    **Completeness:**
    - Did I fully implement everything in the chunk?
    - Did I miss any part of the contract?
    - Are there edge cases I didn't handle?

    **Quality:**
    - Is this my best work?
    - Are names clear and accurate (match what things do, not how they work)?
    - Is the code clean, maintainable, and idiomatic in this project's language?

    **Discipline (distillation):**
    - Is every keep-verbatim item present and byte-for-byte unaltered (no rounded constants,
      reworded prompts, or reordered steps)?
    - Did I import any of the reference's deps? (must be none — seams wired to this project's deps)
    - For learn-then-rewrite: did I write this independently, without pasting reference lines?
    - Did I stay within the mode (copy changed only imports/naming; port preserved structure)?

    **Testing:**
    - Do tests actually verify behavior (not just mock behavior)?
    - If equivalence-testing applied, are the tests reference-derived?
    - Are tests comprehensive?

    If you find issues during self-review, fix them now before reporting.

    ## Report Format

    When done, report:
    - **Status:** DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
    - What you implemented (or what you attempted, if blocked); the mode used
    - Keep-verbatim items preserved (list them)
    - Seam substitutions made
    - What you tested and test results
    - Files changed
    - Self-review findings / concerns

    Use DONE_WITH_CONCERNS if you completed the work but have doubts about correctness or fidelity.
    Use BLOCKED if you cannot complete the chunk. Use NEEDS_CONTEXT if you need information that
    wasn't provided (a keep-verbatim value, a seam, the reference location). Never silently ship
    work you're unsure about.
```
