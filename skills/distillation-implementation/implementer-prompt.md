# Implementer Subagent Prompt Template (Distillation)

Heavy-copied from superpowers' `subagent-driven-development`, with distillation discipline added. Use when dispatching an implementer subagent for a logic-heavy or risky chunk.

```
Task tool (general-purpose):
  description: "Distill Task N: [task name]"
  prompt: |
    You are distilling Task N: [task name] from a reference implementation into this project.

    ## Task Description

    [FULL TEXT of the task from the distillation plan — paste it; don't make the subagent read the file]

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

    If anything about the contract, the mode, the keep-verbatim items, or the seam substitutions is
    unclear, ask now. Don't guess.

    ## Your Job

    1. Implement the chunk under its mode.
    2. Preserve every keep-verbatim item exactly; wire seams to this project's deps; import NONE of
       the reference's deps.
    3. Test (use equivalence-testing if the spec assigned it to this chunk; otherwise tests
       appropriate to the behavior).
    4. Commit — one commit per chunk: `distill(<repo>): <what was distilled>`.
    5. Self-review (below).
    6. Report back.

    Work from: [directory]

    **While you work:** if you hit something unexpected or unclear, ask. It's always OK to pause.

    ## Code Organization

    You reason best about code you can hold in context at once. Keep files focused:
    - Follow the file structure in the spec; one clear responsibility per file.
    - If a file is growing beyond the spec's intent, stop and report DONE_WITH_CONCERNS — don't
      split files on your own.
    - In an existing codebase, follow established patterns. Don't restructure outside your chunk.

    ## When You're in Over Your Head

    It is always OK to stop and say "this is too hard." Bad work is worse than no work.

    STOP and escalate (BLOCKED / NEEDS_CONTEXT) when:
    - The chunk needs architectural decisions with multiple valid approaches.
    - You can't understand the reference well enough to preserve its behavior.
    - A `port` chunk needs so much restructuring it's becoming a rewrite (mode shift — escalate,
      don't shift silently).
    - You've been reading file after file without progress.

    ## Before Reporting Back: Self-Review

    - **Completeness:** implemented everything in the chunk? edge cases handled?
    - **Quality:** best work? names clear? idiomatic in this project's language?
    - **Discipline (distillation):**
      - Is every keep-verbatim item present and byte-for-byte unaltered?
      - Did I import any of the reference's deps? (must be none)
      - For learn-then-rewrite: did I write this without pasting reference lines?
    - **Testing:** do tests verify behavior, not mocks? reference-derived if equivalence-testing applied?

    Fix issues now before reporting.

    ## Report Format

    - **Status:** DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
    - What you implemented; the mode used
    - Keep-verbatim items preserved (list them)
    - Seam substitutions made
    - What you tested and results
    - Files changed
    - Self-review findings / concerns

    Use DONE_WITH_CONCERNS if you finished but have doubts. Never silently ship work you're unsure about.
```
