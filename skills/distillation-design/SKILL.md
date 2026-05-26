---
name: distillation-design
description: Use after analyzing-reference to interactively design a distillation - decides per-chunk mode (copy/port/learn-then-rewrite), target API, attribution plan, equivalence test plan, then produces a distillation spec
---

# Distillation Design

Interactive design step. The brainstorming analog, specialized for porting. Reads a reference map, asks the user clarifying questions one at a time, decides modes, and produces a distillation spec the plan skill can consume.

**Announce at start:** "I'm using `distillation-design` to design the distillation."

<HARD-GATE>
Do NOT invoke `distillation-plan` or any execution skill until the user has approved the written distillation spec.
</HARD-GATE>

## Inputs

- A reference map at `docs/distilling/<repo>-<feature-slug>-reference-map.md` (produced by `analyzing-reference`).
- The user's project (for understanding target conventions: language, test framework, naming, license).

If the reference map is missing or stale, invoke `analyzing-reference` first.

## Outputs

A distillation spec at `docs/specs/YYYY-MM-DD-distill-<repo>-<feature-slug>.md` in the user's project, committed to git, approved by the user.

## Checklist

Create one `TaskCreate` entry per item.

1. **Read the reference map end-to-end.** Note surprises: large dep graph, unusual licenses, heavy hidden coupling.
2. **Run the license-compatibility check** by invoking `attribution-and-license`. STOP if INCOMPATIBLE without override.
3. **Ask clarifying questions** (see below) one at a time.
4. **Assign a mode per chunk.** Use the criteria in `references/mode-decision-criteria.md`. Record the criterion that triggered each decision.
5. **Define the target API.** How does the distilled feature look from inside the user's project?
6. **Define the equivalence test plan.** Which tests we port, adapt, write fresh, or fall back to spot-check.
7. **Define the attribution plan** in terms `attribution-and-license` understands (which files, which commit, override needed?).
8. **Write the distillation spec** to `docs/specs/YYYY-MM-DD-distill-<repo>-<feature-slug>.md`.
9. **Spec self-review** (see below).
10. **Commit the spec** with message: `spec: distill <repo>/<feature>`.
11. **User review gate.** Ask the user to review and approve. Wait. If changes requested, fix and re-review.
12. **Hand off to `distillation-plan`.**

## Clarifying questions (one at a time)

Ask in this order. Prefer multiple-choice. Skip a question if the reference map already answers it unambiguously.

1. **Target API shape:** mirror the reference / wrap with our conventions / re-shape the interface entirely.
2. **Scope:** take everything in the reference map / take a subset (which chunks?). Mark `can-stub` deps as in-scope only if needed by the chosen subset.
3. **Target language:** same as reference / different (specify). If different, surface that this is a cross-language port and `references/cross-language-notes.md` will inform mode decisions and adaptations.
4. **Test strategy:** port reference tests / fresh equivalence tests captured from running the reference / spot-check only (forced if no tests exist or learn-then-rewrite mode dominates).
5. **External library substitutions:** for each external library in the reference map, ask: same library available in target / use an equivalent (which?) / replace with bespoke code.
6. **Hidden-coupling handling:** for each item in the reference map's hidden coupling section, ask: preserve / replace with target-project equivalent / explicitly remove from scope.

If a question has an obvious default given the reference map and target project, propose it and confirm rather than open-ending it.

## Mode assignment

For each chunk (typically one file, or one logical group of files), assign one mode:

- **copy** — same language, license clean for copy, idiomatic for target. Bring the code over with minimal changes (rename imports/types to match target).
- **port** — different language, OR same language but materially different idioms (e.g., callback-style → async/await), OR same language but the chunk uses target-incompatible patterns (e.g., browser globals in a Node project).
- **learn-then-rewrite** — the chunk is heavily entangled with reference-specific infrastructure (DI containers, framework lifecycles, event buses) that don't exist in the target; OR the value is in the algorithmic *approach* rather than the code; OR cross-language translation is so heavy that "porting" is misleading.

See `references/mode-decision-criteria.md` for explicit criteria.

Record the chosen mode and the deciding criterion for every chunk. The plan skill needs both.

## Cross-language adaptations

When target language differs from source, load `references/cross-language-notes.md` and use it to:

- Map type systems (TS structural types → Python typing.Protocol, Go interfaces, etc.).
- Map idioms (Promises ↔ async/await ↔ futures ↔ goroutines+channels).
- Identify library equivalents (axios ↔ requests, lodash utilities ↔ stdlib).

Record the chosen translations per chunk in the spec.

## Spec structure

Write the spec to `docs/specs/YYYY-MM-DD-distill-<repo>-<feature-slug>.md` with these sections:

```markdown
# Distillation Spec: <repo>/<feature>

**Status:** Draft → awaiting user review.
**Date:** YYYY-MM-DD
**Reference map:** docs/distilling/<repo>-<feature-slug>-reference-map.md
**Reference commit:** <SHA>
**Reference license:** <SPDX>
**Target license:** <SPDX>
**License compatibility:** COMPATIBLE | COMPATIBLE-WITH-CONSTRAINTS | OVERRIDDEN
**Source language:** <lang>  →  **Target language:** <lang>

## 1. Goal
One paragraph: what feature, what value it brings to the user's project.

## 2. Scope
What we take, what we leave. Reference each chunk by source path.

## 3. Mode assignments

| Source chunk | Target file(s) | Mode | Deciding criterion | Adaptation notes |
| ------------ | -------------- | ---- | ------------------ | ---------------- |
| ...          | ...            | copy / port / learn-then-rewrite | ... | ... |

## 4. Target API
The public surface of the distilled feature inside the user's project.
Signatures, types, examples.

## 5. External library plan

| Reference library | Target choice | Notes |
| ----------------- | ------------- | ----- |

## 6. Hidden coupling resolutions

| Coupling item from ref map | Resolution |
| -------------------------- | ---------- |

## 7. Equivalence test plan
Test framework in target. For each chunk: port-tests / fresh-equivalence-tests / spot-check.
Fallbacks explicitly stated for chunks without reference tests.

## 8. Attribution plan
- Per-file headers: yes for every distilled file (including ported tests).
- ATTRIBUTION.md update planned.
- Source license copy to `licenses/<repo>-<spdx>.txt`: yes.
- Override (if INCOMPATIBLE was overridden): record both licenses, user's reason, date.

## 9. Out of scope
Things explicitly NOT being distilled in this run, even if present in the reference map.

## 10. Success criteria
- All ported tests pass against the distilled code.
- All distilled files have attribution headers.
- ATTRIBUTION.md and licenses/ updated.
- License compatibility check passed (or override recorded).
- Hidden coupling items each have a resolution; none left unaddressed.
```

## Spec self-review

After writing the spec, scan with fresh eyes:

1. **Placeholder scan:** any TBD/TODO/vague entries? Fix inline.
2. **Internal consistency:** does Section 3's mode column match Section 7's test plan (e.g., a chunk marked `learn-then-rewrite` shouldn't say "port reference tests" without a note)?
3. **Scope coverage:** every chunk in the reference map is either in Section 3 or Section 9. Nothing is silently dropped.
4. **Hidden coupling:** every item from the reference map is resolved in Section 6.
5. **License compatibility:** result recorded explicitly. If overridden, the override section is present.

Fix any issues inline. No re-review loop needed.

## User review gate

After self-review, write to the user:

> "Distillation spec written and committed to `<path>`. Please review it and approve before we move to writing the plan."

Wait. If changes requested, edit, re-run self-review, re-commit, ask again. Only proceed to `distillation-plan` after explicit approval.

## What you do NOT do

- You do **not** map files to tasks. That's `distillation-plan`.
- You do **not** write code or tests. That's `distillation-execution`.
- You do **not** check or update `ATTRIBUTION.md`. The headers come from per-file work in execution; the final pass is also in execution.

## Anti-patterns

| Symptom | Fix |
|---------|-----|
| Modes assigned without a deciding criterion column | Always record the criterion. Otherwise reviewers can't validate. |
| Cross-language target without consulting cross-language-notes | Load the file. Idiom mismatches kill ports. |
| External libraries listed in the reference map are silently kept in the spec | Each one needs an explicit target choice. |
| Hidden coupling section blank | Reference map listed items; this spec must resolve each. |
| Spec approved without license check | `attribution-and-license` runs first. No exceptions. |
