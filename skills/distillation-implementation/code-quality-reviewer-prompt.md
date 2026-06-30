# Code Quality Reviewer Prompt Template (Distillation)

Use this template when dispatching a code quality reviewer subagent.

**Purpose:** verify the distilled chunk is well-built — idiomatic in the target, clean, and free of leaked reference cruft.

**Only dispatch after the spec-compliance review passes.**

```
Task tool (general-purpose):
  description: "Code quality review for Chunk N"
  prompt: |
    You are reviewing the quality of a distilled chunk. Review the diff between [BASE_SHA] and [HEAD_SHA].

    ## Context

    [chunk summary from the implementer's report; the chunk's mode]

    ## Review For:

    **Standard code quality:**
    - Correctness and maintainability — does the code do what it should, and is it easy to change?
    - Names match what things do (not how they work); readable control flow.
    - Errors handled, not swallowed; no dead, duplicated, or commented-out code.

    **Idiomatic in this project's language/stack:**
    - Does it read like native target code, or like transliterated source-language code?
    - Names match what things do; target conventions followed (casing, error handling, async style).

    **Structure:**
    - Does each file have one clear responsibility with a well-defined interface?
    - Can units be understood independently?
    - Does it follow the file structure from the spec?
    - Did this change create new large files or significantly grow existing ones? (Judge only what
      this change contributed; don't flag pre-existing sizes.)

    **Distillation cleanliness:**
    - Did accidental complexity from the reference leak in (its config, logging, telemetry,
      abstractions-for-their-scale)?
    - Are keep-verbatim constants/prompts isolated and clearly labeled, not scattered magic numbers?

    ## Report
    - **Strengths**
    - **Issues** — Critical / Important / Minor, each with file:line
    - **Assessment** — approved | changes needed
```
