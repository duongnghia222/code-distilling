# code-distilling Plugin — Design

**Status:** Approved architecture, ready for per-skill brainstorming and planning.
**Date:** 2026-05-26
**Owner:** nghia.duong@ringkas.homes

---

## 1. Goal

`code-distilling` is a standalone, public Claude Code / Codex plugin that helps a developer port high-quality implementations from a reference open-source repo into their own project — preserving what makes the reference good while adapting it to the target project's stack and conventions.

The plugin shapes a disciplined "brainstorm → design → plan → implement" workflow (mirroring the structure of the `superpowers` plugin) but every step is specialized for the porting use case: the input is always a reference repo on disk, the success criterion is always behavioral equivalence verified by ported tests, and attribution and license compliance are first-class concerns.

The plugin does not depend on `superpowers` at runtime. Skills, conventions, and the bootstrap mechanism are reimplemented inside this plugin so it can be installed standalone.

## 2. Non-goals

- **Scouting reference repos.** The plugin does not search GitHub or evaluate candidate repos. The user is responsible for choosing a reference and placing it on disk.
- **Network access to remote repos.** All reference code is local — see the `ref-code/` convention below.
- **General-purpose coding workflows.** The plugin does not replace `superpowers`-style general skills for non-porting tasks. A user porting some features and writing others from scratch can install both plugins.
- **Auto-detecting feature boundaries inside reference repos.** The user describes what they want in terms the analyzer can locate; the analyzer maps it, but the plugin does not infer "what's interesting" in a repo on its own.

## 3. Core conventions

### 3.1 `ref-code/` directory

Reference repos live in a single directory at the user's project root:

```
my-project/
  src/
  ref-code/
    awesome-auth/        # full clone of reference repo 1
    cool-search-lib/     # full clone of reference repo 2
  docs/
```

The plugin operates against this directory. The user clones or copies reference repos into `ref-code/` before invoking the plugin. Each subdirectory of `ref-code/` is a complete reference repo (its own git history, tests, license, etc.).

### 3.2 Distillation modes

Every chunk of code being distilled is assigned one of three modes during design:

- **copy** — same language, compatible style/license, well-isolated. Bring the code over with minimal changes (naming, import paths). Test by running the reference's tests against the copy.
- **port** — same language but different idioms/style/types, OR a different language. Translate while preserving algorithmic structure. Test by porting the reference's tests and running them against the port.
- **learn-then-rewrite** — implementation is too entangled with reference-specific infrastructure, or the value is in the *approach* rather than the *code*. Use the reference as teacher; rewrite from understanding. Tests are still derived from the reference's behavior where useful, but the implementation is independent.

The `distillation-design` skill decides the mode per chunk (typically per file or per module), with explicit criteria. The decision is recorded in the distillation spec and drives execution.

### 3.3 Equivalence as the success criterion

Wherever possible, the reference's tests are ported to the target project's test framework and become the acceptance criteria for the distillation. A distilled chunk is "done" when its ported tests pass.

This is enforced by the `equivalence-tdd` skill: tests first (ported from reference), verify they fail against an unimplemented or stub target, port the implementation, verify tests pass, commit.

When the reference has no usable tests for the targeted feature, the design skill specifies a fallback: capture representative input/output cases from running the reference, encode those as tests, then port. If neither is feasible (e.g., the reference is documentation or architecture only), the chunk is forced into `learn-then-rewrite` mode and equivalence is established by spot-check, with that limitation explicitly recorded in the spec.

### 3.4 Attribution and license as first-class

Every distilled file or module carries an attribution header pointing at the source repo, source path, source commit hash, and source license. A repo-level `ATTRIBUTION.md` summarizes all reference repos drawn from. Commit messages for distilled code follow a convention that names the source.

License compatibility is gated: the plugin refuses to proceed if the reference license is incompatible with the user's project license, unless the user explicitly overrides and accepts responsibility (recorded in the spec).

The `attribution-and-license` skill owns all of this.

### 3.5 Borrowed superpowers conventions

The plugin adopts these conventions from superpowers:

- "Use when …" YAML frontmatter `description` on every skill, written to auto-trigger appropriately.
- `TaskCreate` per checklist item at the start of each skill.
- "Spec self-review" loop after writing any spec or plan: scan for placeholders, internal contradictions, scope, ambiguity. Fix inline.
- Explicit user-review gate after every spec or plan before moving on.
- One clarifying question at a time during interactive design.
- Subagent-driven execution as the recommended execution mode, with fresh subagents per task and two-stage review (spec compliance, then code quality).
- `docs/specs/YYYY-MM-DD-<topic>-design.md` for specs and `docs/plans/YYYY-MM-DD-<topic>-plan.md` for plans inside the user's project. The plugin's own design and plan documents also follow this convention inside the plugin repo.

## 4. End-to-end user flow

```
User: "Port the OAuth flow from ref-code/awesome-auth into my project."

[using-code-distilling auto-fires; recognizes porting intent and ref-code/ presence]
        ↓
[analyzing-reference]
  Deep-reads ref-code/awesome-auth. Locates the OAuth feature. Produces a
  "reference map": files implementing the feature, transitive deps inside the
  repo, external libraries, tests covering the feature, entry points, hidden
  coupling, license file location and contents.
        ↓
[distillation-design]                       (brainstorming analog)
  Interactive. Decides per chunk: copy / port / learn-then-rewrite.
  Defines scope (take/leave), target API in the user's project, equivalence
  test plan, attribution plan. Delegates license-compatibility check to
  attribution-and-license. Output: distillation spec at docs/specs/...
        ↓
[distillation-plan]                         (writing-plans analog)
  Source→target file map. Bite-sized task list. Each implementation task is
  paired with an equivalence-test task (test first, then impl). Output:
  distillation plan at docs/plans/...
        ↓
[distillation-execution]                    (subagent-driven-development analog)
  Per task: dispatch implementer subagent that follows equivalence-tdd
  (port reference test → verify it fails → port impl → verify it passes →
  commit). Two-stage review per task: spec compliance, then code quality.
  At the end: run the final attribution-and-license pass to write
  ATTRIBUTION.md and verify all attribution headers are present.
```

Two sub-skills are referenced from inside the core flow:

- **`equivalence-tdd`** — the rigid TDD discipline used by `distillation-execution`. Tests are derived from the reference, not invented.
- **`attribution-and-license`** — the legal/credit discipline used by `distillation-design` (compatibility check, attribution plan) and `distillation-execution` (final attribution pass).

## 5. Skill profiles

### 5.1 `using-code-distilling` (bootstrap)

**Purpose.** Establish skill discipline at session start. Same role as `using-superpowers` in superpowers: ensures any of the other skills auto-trigger at the right moment.

**Triggers.** Loads at session start via the plugin's `SessionStart` hook (Claude Code) or session bootstrap (Codex). Once loaded, it instructs the agent to check for relevant skills before any response, including clarifying questions.

**Content.** Mirrors `using-superpowers`'s structure: an `<EXTREMELY-IMPORTANT>` block requiring skill invocation when applicable, a list of red-flag rationalizations, a skill-priority order ("process skills first, implementation skills second"), and a brief "how to access skills" section adapted for Claude Code and Codex.

**Differences from `using-superpowers`.** Adds two distilling-specific triggers in its red-flags table: "the user mentioned a reference repo or ref-code/" and "we're porting from somewhere else" both must invoke the porting flow. Removes superpowers-specific examples that don't apply.

**Output.** Skill discipline is in effect for the rest of the session.

### 5.2 `analyzing-reference`

**Purpose.** Given a reference repo at `ref-code/<repo>/` and a description of the feature the user wants, produce a structured "reference map" that subsequent skills can act on.

**Triggers.** Invoked at the start of any distilling workflow, either auto-fired by bootstrap when the user mentions a reference, or invoked manually with `Skill code-distilling:analyzing-reference`.

**Inputs.**
- Path to a `ref-code/<repo>/` directory.
- Natural-language description of the feature to distill (e.g., "the OAuth login and token refresh flow").

**Process.**
1. Read the reference repo's top-level files: README, LICENSE, package manifest, test config.
2. Locate the feature: file search by names/keywords from the user's description, then deep-read.
3. Build the reference map:
   - **Feature files:** primary implementation files.
   - **Transitive deps inside the repo:** every file in the repo that a feature file imports/uses, recursively. Marked as "must take" or "can stub".
   - **External libraries:** non-stdlib dependencies the feature pulls in. With version constraints if visible.
   - **Tests:** test files exercising the feature, with the test framework identified.
   - **Entry points:** how the feature is invoked from the outside (functions, classes, API endpoints).
   - **Hidden coupling:** globals, env vars, file paths, side effects the feature relies on.
   - **License & attribution data:** license type, copyright holders, commit hash to record.
4. Write the reference map to a working file under `docs/distilling/<repo>-<feature>-reference-map.md` in the user's project (not committed automatically — it's a working artifact).

**Output.** Reference map document. Handed to `distillation-design`.

### 5.3 `distillation-design`

**Purpose.** Interactive design step. Mirrors `superpowers:brainstorming` in shape. Decides what to distill, how, and into what shape inside the target project. Produces the distillation spec.

**Triggers.** After `analyzing-reference` completes, or when the user requests redesign for an existing reference map.

**Process.**
1. Confirm the reference map with the user. Surface anything surprising (hidden coupling, license concerns, oversized scope).
2. Run the license-compatibility check by invoking `attribution-and-license`. If incompatible, stop unless the user explicitly overrides.
3. Ask clarifying questions, one at a time, multiple-choice when possible:
   - Target API: how should this feature look in the user's project? (mirror the reference / wrap with our conventions / re-shape the interface)
   - Scope: take everything in the reference map / take a subset (which?)
   - Cross-language: is the target language different? (if so, surface a cross-language adaptation pass)
   - Test strategy: port reference tests / fresh equivalence tests / spot-check (forced if no tests exist)
4. Assign a **mode per chunk** (copy / port / learn-then-rewrite) using explicit criteria documented in the skill: same-language and clean-license → favor copy; different language or different idioms → port; entangled with reference infrastructure or value-is-in-the-approach → learn-then-rewrite.
5. Define the **target API** the distilled feature exposes to the rest of the user's project.
6. Define the **attribution plan**: which files get headers, which commit hash to record, what goes in `ATTRIBUTION.md`. Detail is delegated to `attribution-and-license`.
7. Define the **equivalence test plan**: which tests we port, which we adapt, which we write fresh, and the fallback for chunks with no tests.
8. Write the distillation spec to `docs/specs/YYYY-MM-DD-distill-<repo>-<feature>.md` in the user's project. Run the spec self-review loop. Commit. Hand to the user for review.

**Output.** Approved distillation spec. Hands to `distillation-plan`.

### 5.4 `distillation-plan`

**Purpose.** Translate the distillation spec into a task-by-task, file-level plan ready for execution. Mirrors `superpowers:writing-plans` in shape.

**Triggers.** After the user approves a distillation spec.

**Process.**
1. Read the spec end-to-end.
2. Produce the **source→target file map**: each entry pairs a reference-repo file (or a logical chunk if cross-language splitting differs, or a referenced module for learn-then-rewrite chunks where the target file has no 1:1 source) with its target path in the user's project, the assigned mode, and a brief adaptation note (naming, types, library substitutions, or design-influence summary for learn-then-rewrite).
3. Produce the **task list**. Each chunk yields a sequence:
   - **Equivalence-test task:** port (or write) the test that defines the chunk's behavior. Step list follows `equivalence-tdd`: write/port the test, run it, verify it fails, commit.
   - **Implementation task:** port the implementation with adaptations specified in the spec. Step list: write/port the code, run the test, verify it passes, commit.
   - **Attribution task:** add the attribution header (delegated step pointing into `attribution-and-license`).
4. Order tasks so dependencies are honored (e.g., a utility chunk is distilled before the chunk that uses it).
5. Include a **final attribution task**: run `attribution-and-license`'s repo-level pass to generate `ATTRIBUTION.md` and verify every distilled file has a header.
6. Write the plan to `docs/plans/YYYY-MM-DD-distill-<repo>-<feature>.md`. Run plan self-review (placeholder scan, spec coverage, type consistency). Commit. Hand to the user.

**Output.** Approved distillation plan. Hands to `distillation-execution`.

### 5.5 `distillation-execution`

**Purpose.** Execute the plan. Mirrors `superpowers:subagent-driven-development`. Dispatches a fresh implementer subagent per task with a tightly-scoped context.

**Triggers.** After the user approves a distillation plan and selects subagent-driven execution.

**Process.**
1. Read the plan once. Extract every task with full text and required context (source file paths, reference-map excerpts the task needs, target paths, mode, adaptation notes).
2. Create one `TaskCreate` entry per plan task.
3. For each task in order:
   a. Dispatch the implementer subagent with the task text and the curated context. Implementer follows `equivalence-tdd` (test first, verify fail, impl, verify pass, commit).
   b. Dispatch the spec-compliance reviewer subagent. Re-dispatch the implementer for any gaps until the reviewer approves.
   c. Dispatch the code-quality reviewer subagent. Re-dispatch the implementer for any issues until the reviewer approves.
   d. Mark the task complete.
4. After all tasks: dispatch a final code reviewer for the entire distillation.
5. Run the final `attribution-and-license` pass.

**Status handling, model selection, and subagent prompts** mirror the conventions in `superpowers:subagent-driven-development`, adapted with prompts that mention the distilling discipline.

**Output.** Completed distillation: ported code in the target project, ported tests passing, attribution headers and `ATTRIBUTION.md` in place, all committed.

### 5.6 `equivalence-tdd`

**Purpose.** The rigid TDD discipline for ports. Tests are derived from the reference, not invented from requirements.

**Type.** Rigid. Follow exactly.

**Triggers.** Invoked by `distillation-execution`'s implementer subagent for every implementation task. May also be invoked manually if a developer is porting one chunk by hand.

**Discipline.** For each chunk:
1. **Port or write the test first.** Same-language: copy the reference's test, adapt imports and any naming changes. Cross-language: translate the test to the target language and test framework, preserving assertions. No-reference-tests fallback: capture representative input/output from the reference and encode as a test.
2. **Run the test against an unimplemented or stub target.** It must fail. If it passes accidentally, the test is wrong — investigate.
3. **Port or write the implementation.** Apply adaptations from the spec (naming, types, library substitutions).
4. **Run the test.** It must pass. If it doesn't, debug: is the port wrong, or is the test importing something not yet ported?
5. **Commit.** The test and implementation go in the same commit, with a message naming the source.

**Anti-patterns** (called out explicitly so the agent doesn't drift):
- Skipping step 2 ("the test is obviously correct, no need to run it failing first") — banned. Step 2 is the only thing that confirms the test is exercising the right code path.
- Adapting the test to make it pass without verifying the reference's test would have caught the same bug — banned.
- Writing fresh tests "while we're at it" — out of scope. Equivalence first; new behavior is a separate workstream.

### 5.7 `attribution-and-license`

**Purpose.** Make license compliance and credit non-optional. Centralizes the rules so other skills don't reinvent them.

**Type.** Rigid for compliance steps; flexible for header wording.

**Triggers.**
- `distillation-design` invokes the **compatibility check** (target license vs. reference license) before approving any distillation.
- `distillation-plan` consults it when producing per-file attribution tasks.
- `distillation-execution` invokes the **final pass** at the end of a distillation run to write `ATTRIBUTION.md` and verify all headers are in place.

**Compatibility check.**
- Read the reference repo's `LICENSE` file.
- Compare to the target project's license (read from `LICENSE` at project root, or asked from the user if absent).
- Use a built-in compatibility matrix for common cases (MIT, Apache-2.0, BSD-3, GPLv2/3, AGPL, MPL, proprietary). Output: compatible / incompatible-with-explicit-override-required / unknown-ask-user.
- If incompatible, refuse to proceed unless the user explicitly overrides. The override is recorded in the distillation spec.

**Per-file attribution header.** A short comment block at the top of every distilled file in the project's primary comment style, containing: source repo URL, source path inside the repo, source commit hash recorded by `analyzing-reference`, source license name, "Distilled into this project on YYYY-MM-DD". For chunks produced in **learn-then-rewrite** mode, the header notes that the implementation is independent and attributes the design influence instead; the source path field names the reference module that inspired the design rather than a 1:1 source file.

**`ATTRIBUTION.md`.** Generated at the project root. One section per reference repo drawn from. Lists every distilled file or chunk, source paths, commit hashes, license, and a paragraph crediting the original authors as named in the reference repo's metadata.

**Commit-message convention.** Commits that introduce distilled code start with `distill(<repo>): <what>` and include a trailer `Source: <repo>@<commit-hash>:<path>`.

## 6. Repo layout

```
code-distilling/
  .claude-plugin/                # Claude Code plugin manifest (mirrors superpowers' shape)
    plugin.json
    hooks.json
  .codex-plugin/                 # Codex plugin manifest
    plugin.json
    hooks.json
  skills/
    using-code-distilling/
      SKILL.md
      references/
        copilot-tools.md         # tool-name mapping for Copilot CLI, if applicable
    analyzing-reference/
      SKILL.md
    distillation-design/
      SKILL.md
      references/
        mode-decision-criteria.md
        cross-language-notes.md
    distillation-plan/
      SKILL.md
    distillation-execution/
      SKILL.md
      implementer-prompt.md
      spec-reviewer-prompt.md
      code-quality-reviewer-prompt.md
    equivalence-tdd/
      SKILL.md
    attribution-and-license/
      SKILL.md
      references/
        license-compatibility-matrix.md
        header-templates.md
  docs/
    specs/
      2026-05-26-code-distilling-plugin-design.md   # this document
    plans/
      (per-skill plans, generated as the plugin is built)
  README.md
  LICENSE
```

Cross-language adaptation lives as a reference file under `skills/distillation-design/references/cross-language-notes.md` (type-system mappings, idiom mappings, common library equivalents). It is loaded by `distillation-design` and `distillation-execution` when the target language differs from the reference. If the file grows past about 500 lines or starts shaping behavior independently, it is promoted to its own skill in a future revision.

## 7. Harness support

Claude Code and Codex are both supported at v1, mirroring how superpowers ships both `.claude-plugin/` and `.codex-plugin/` manifests. Each manifest registers the plugin and a `SessionStart` hook that loads `using-code-distilling`. Skill files themselves are shared between the two manifests; only the manifest format differs.

Additional harness adapters (Copilot CLI, Gemini CLI) are out of scope for v1 and tracked in section 9.

## 8. v1 build sequence

Each skill gets its own brainstorm → spec → plan → implement cycle. The recommended order:

1. **`using-code-distilling`** — bootstrap. Without this, no other skill auto-triggers. Smallest skill, fastest first win.
2. **`analyzing-reference`** — the input producer for every other skill. Designed and built before design/plan/execution so those have a real reference map to consume in testing.
3. **`attribution-and-license`** — built early because both `distillation-design` and `distillation-plan` reference it. Self-contained, no dependency on the design/plan skills.
4. **`distillation-design`** — depends on analyzing-reference and attribution-and-license.
5. **`equivalence-tdd`** — built before distillation-plan so the plan skill can reference its discipline by name in task templates.
6. **`distillation-plan`** — depends on distillation-design (consumes spec) and equivalence-tdd (referenced in task templates).
7. **`distillation-execution`** — depends on everything above. Built last.

After step 7, the plugin is end-to-end usable for a real distillation. v1 release is gated on a working end-to-end run against at least one real reference repo, with the result reviewed by the maintainer.

## 9. Out of scope for v1

- **Reference scouting.** No skill that searches GitHub or evaluates candidate repos. The user supplies the reference.
- **Promotion of cross-language to its own skill.** Cross-language adaptation is reference content inside `distillation-design` until it visibly outgrows that.
- **Copilot CLI and Gemini CLI manifests.** Claude Code and Codex only at v1. Skills are written portably enough that adding adapters later does not require rewriting skill content.
- **An automated equivalence harness.** Tests are run by the target project's existing test framework. The plugin does not provide its own test runner or diff harness.
- **A `debugging-port-divergence` skill.** If a port misbehaves, users fall back to general debugging tools. Promoted to its own skill only if real usage shows a recurring failure pattern that needs codified guidance.

## 10. Success criteria

The plugin is considered v1-ready when:
- A user can run a single session that takes "I want feature X from `ref-code/<repo>`" through to committed, tested, attributed code in their project, using only this plugin.
- License-incompatibility is correctly blocked, with an override path recorded in the spec.
- Cross-language porting works on at least one real example (e.g., a TypeScript utility ported to Python with tests).
- All seven skills auto-trigger appropriately from a fresh session in Claude Code, verified by a recorded transcript.
