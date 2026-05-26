---
name: attribution-and-license
description: Use whenever code is being copied, ported, or learn-then-rewritten from a reference repo - enforces license compatibility, per-file attribution headers, ATTRIBUTION.md, and commit-message conventions
---

# Attribution and License

Make license compliance and credit non-optional. This skill is invoked from three places in the porting flow:

- `distillation-design` calls the **compatibility check** before approving any distillation.
- `distillation-plan` consults the **header rules** when producing per-file attribution tasks.
- `distillation-execution` runs the **final pass** at the end of a distillation run.

**Announce at start:** "I'm using `attribution-and-license` to <check compatibility | plan headers | finalize attribution>."

**Type:** Rigid for compliance steps. Flexible for header wording.

## 1. Compatibility check

**When:** Called by `distillation-design` after the reference map exists, before mode decisions are finalized.

**Inputs:**
- Reference license: `ref-code/<repo>/LICENSE` (read by `analyzing-reference`).
- Target license: target project's `LICENSE` at project root; if absent, ask the user.

**Process:**

1. Identify both licenses by SPDX identifier. If you cannot identify one, ask the user — do not guess.
2. Look up the pair in `references/license-compatibility-matrix.md`.
3. Report one of:
   - **COMPATIBLE** — proceed.
   - **INCOMPATIBLE** — refuse to proceed. State which license combination is the problem and why. Offer the user three paths: (a) pick a different reference, (b) change the target project's license, (c) explicitly override and accept legal responsibility (recorded in the spec).
   - **UNKNOWN** — combination is not in the matrix. Ask the user to research and decide. Do not invent an answer.

4. If the user chooses to override an INCOMPATIBLE pairing, record the override in the distillation spec under a "License override" section with: both licenses, the user's stated reason, and the date.

**Do not** proceed with any porting work until the check returns COMPATIBLE or the user has explicitly overridden.

## 2. Per-file attribution headers

**When:** Generated as part of each ported file's commit, by the implementer subagent inside `distillation-execution`.

**Template:** See `references/header-templates.md` for language-specific comment styles. The header contains, in this order:

```
Source: <reference repo URL or path under ref-code/>
Source path: <path inside the reference repo, e.g. src/auth/oauth.ts>
Source commit: <full commit SHA recorded by analyzing-reference>
Source license: <SPDX>
Distilled into this project on <YYYY-MM-DD>
```

**Mode-specific variations:**

- **copy:** header as above. `Source path` is the single file copied.
- **port:** header as above. If multiple source files were merged into one target file, list all source paths.
- **learn-then-rewrite:** add a final line `Implementation is independent; design influenced by the source.` `Source path` names the reference module that inspired the design (best-fit, not 1:1).

**Header placement:** at the very top of the file, before any imports, in the target language's primary comment style. The header is the first thing a reader sees.

**Files that get headers:** every file in the target project that contains distilled code, including ported test files. Files that are entirely original (unrelated to the distillation) do not.

## 3. ATTRIBUTION.md

**When:** Generated or updated by the final pass in `distillation-execution`, after all tasks complete.

**Location:** `ATTRIBUTION.md` at the project root.

**Structure:**

```markdown
# Attribution

This project distills code from the following open-source repositories.

## <Reference repo name>

- **Source:** <repo URL or local path>
- **License:** <SPDX>
- **Commit:** <full SHA>
- **Distilled on:** <YYYY-MM-DD>
- **Original authors:** <names from LICENSE / package metadata / git log>

### Distilled into this project

| Target file | Source path | Mode |
| ----------- | ----------- | ---- |
| src/auth/oauth.ts | src/auth/oauth.ts | copy |
| src/auth/refresh.ts | src/auth/refresh.ts + src/auth/token.ts | port |
| src/auth/strategy.ts | (design influence) src/auth/ | learn-then-rewrite |

A copy of the source license is preserved at `licenses/<repo>-<spdx>.txt`.

## <Next reference repo>

...
```

The skill also copies the verbatim text of each reference's `LICENSE` file into `licenses/<repo>-<spdx>.txt` so the project ships the source license alongside its attribution claim.

## 4. Commit-message convention

Commits introducing distilled code follow this format:

```
distill(<repo>): <what was distilled>

<optional body with adaptation notes>

Source: <repo>@<short-commit-sha>:<source path>
License: <SPDX>
```

For multi-file commits, use multiple `Source:` lines.

For learn-then-rewrite commits, use `Source-influence:` instead of `Source:`.

## Final pass checklist

Run all of these at the end of a distillation, as the last task in `distillation-execution`. Create `TaskCreate` entries.

1. **Verify every distilled file has a header.** Grep for the header marker across changed files in the distillation branch. Any file missing one is fixed.
2. **Generate or update `ATTRIBUTION.md`.** Add a new section per reference repo not already listed; under each, list every distilled file added in this run.
3. **Copy source `LICENSE` files** to `licenses/<repo>-<spdx>.txt`. Skip if already present.
4. **Verify commit messages.** Scan commits introduced in this distillation. Any without `Source:` / `Source-influence:` trailers are fixed (amend or follow-up commit).
5. **Commit the attribution changes.** `chore(attribution): finalize attribution for <repo> distillation`.

## Anti-patterns

| Symptom | Fix |
|---------|-----|
| "License is probably MIT, let's go" | Read the file. If unreadable, ask. Never guess. |
| Header is buried mid-file | Move to the top. Readers should see it without scrolling. |
| `ATTRIBUTION.md` lists the repo but not the files | List every file. Auditability matters. |
| `Source:` trailer missing on a commit | Amend (if local) or follow-up commit with the trailer. Don't skip. |
| Override of INCOMPATIBLE without recording in spec | Stop and record. The user assumed responsibility — make that visible. |
