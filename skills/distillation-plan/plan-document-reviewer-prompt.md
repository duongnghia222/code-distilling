# Plan Document Reviewer Prompt Template (Distillation)

Use this template when dispatching a plan document reviewer subagent.

**Purpose:** verify the plan is complete, matches the spec, preserves the gold, and is buildable.

```
Task tool (general-purpose):
  description: "Review distillation plan document"
  prompt: |
    You are a plan document reviewer. Verify this distillation plan is complete and ready for implementation.

    **Plan to review:** [PLAN_FILE_PATH]
    **Spec for reference:** [SPEC_FILE_PATH]

    ## What to Check

    | Category | What to Look For |
    |----------|------------------|
    | Completeness | TODOs, placeholders, incomplete tasks, missing steps |
    | Spec alignment | Every chunk in the spec has a task; no major scope creep |
    | Keep-verbatim coverage | Every keep-verbatim item from the spec appears, verbatim, in a task — constants, prompts, step order, regexes. A missing one is a dropped trick. |
    | Seam / mode discipline | Each task's mode matches the spec; seam substitutions are named; no task introduces a reference dependency |
    | Task decomposition | Tasks have clear boundaries; steps are actionable with real code |
    | Buildability | Could a zero-context engineer follow this without getting stuck? |

    ## Calibration

    Only flag issues that would cause real problems during implementation: a missing chunk, a
    keep-verbatim item that's named but not shown (so it'll get dropped or invented), a leaked
    reference dep, contradictory steps, placeholder content, or tasks too vague to act on.
    Minor wording and stylistic preferences are not blocking.

    ## Output Format

    ## Plan Review

    **Status:** Approved | Issues Found

    **Issues (if any):**
    - [Task X, Step Y]: [specific issue] — [why it matters for implementation]

    **Recommendations (advisory, do not block approval):**
    - [suggestions for improvement]
```
