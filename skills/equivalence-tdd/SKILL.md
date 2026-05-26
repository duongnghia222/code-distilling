---
name: equivalence-tdd
description: Use during execution of every port task - rigid TDD discipline for distilled code where tests are derived from the reference (not invented from requirements); port the test first, watch it fail, port the implementation, watch it pass, commit
---

# Equivalence-TDD

## Overview

The rigid TDD discipline for porting. **Tests are derived from the reference, not invented from requirements.** That's what differentiates a distillation from a from-scratch rewrite: the reference's tests (or captured behaviors, or property assertions) become the acceptance criteria for the port.

Invoked by `distillation-execution`'s implementer subagent for every implementation task. Also usable directly if a human is porting one chunk by hand.

**Core principle:** If you didn't watch the test fail against the not-yet-ported target, you don't know if it tests the right thing.

**Violating the letter of the rules is violating the spirit of the rules.**

**Type:** Rigid. Follow exactly. Do not adapt away the discipline.

**Announce at start:** "I'm using `equivalence-tdd` to port <chunk> with equivalence verification."

## The Iron Law

```
NO PORTED IMPLEMENTATION WITHOUT A PORTED TEST THAT FAILED FIRST
```

Wrote the implementation before the test? Delete it. Start over.

**No exceptions:**

- Don't keep it as "reference".
- Don't "adapt" it while writing the test.
- Don't peek at it.
- Delete means delete.

Re-implement fresh from the test. Period.

## The Cycle

```dot
digraph equivalence_tdd {
    rankdir=LR;
    test [label="PORT TEST\nfrom reference", shape=box, style=filled, fillcolor="#ffcccc"];
    verify_fail [label="Verify fails\ncorrectly", shape=diamond];
    impl [label="PORT IMPL\nor learn-then-rewrite", shape=box, style=filled, fillcolor="#ccffcc"];
    verify_pass [label="Verify passes\nclean output", shape=diamond];
    header [label="Add attribution\nheader", shape=box, style=filled, fillcolor="#ccccff"];
    commit [label="COMMIT\nwith Source: trailer", shape=ellipse];

    test -> verify_fail;
    verify_fail -> impl [label="yes"];
    verify_fail -> test [label="wrong failure\nor test passed"];
    impl -> verify_pass;
    verify_pass -> header [label="yes"];
    verify_pass -> impl [label="no"];
    header -> commit;
}
```

## Steps

Create `TaskCreate` entries and complete in order.

### 1. Port (or write) the test FIRST

Choose the test source based on the plan's **test strategy** for this chunk:

- **port-reference-test:** open the reference's test file at the path in the plan. Copy or translate the relevant test cases to the target test framework. Adapt imports, type annotations, naming. Preserve assertions and the shape of the test data.
- **fresh-equivalence-test:** the reference has no usable tests; the plan provides input/output pairs captured from running the reference. Encode each pair as a test case. Cite the captured run in a comment.
- **property-based fallback:** the plan provides properties (sort stable, length preserved, idempotent). Encode each property as a test in the target's property-testing library (fast-check, hypothesis, proptest) or a hand-written generative test.

The test goes in the target project's test tree, mirroring the source structure where reasonable. The test file carries an attribution header (see `attribution-and-license`).

<Good>
```typescript
// test/cache/lru.test.ts
// Source: awesome-auth@a1b2c3d:test/util/lru.test.ts
// License: MIT
// Distilled into this project on 2026-05-26

import { LruCache } from '../../src/cache/lru';

test('evicts the least-recently-used entry when capacity is exceeded', () => {
  const cache = new LruCache<string, number>(2);
  cache.set('a', 1);
  cache.set('b', 2);
  cache.set('c', 3); // evicts 'a'
  expect(cache.get('a')).toBeUndefined();
  expect(cache.get('b')).toBe(2);
  expect(cache.get('c')).toBe(3);
});
```
Named after behavior, real assertions, mirrors the reference's test structure.
</Good>

<Bad>
```typescript
test('lru works', () => {
  const cache = mockLruCache();
  expect(cache.set).toHaveBeenCalledTimes(3);
});
```
Vague name, tests a mock instead of behavior, no relationship to reference.
</Bad>

### 2. Run the test against an unimplemented / stub target

**MANDATORY. Never skip.**

```bash
<target test command>
```

**Acceptable failures:**

- Module not found / import error pointing at the not-yet-ported target file.
- Assertion failure showing the unimplemented stub returning the wrong thing.
- Compilation failure pointing at the missing target.

**Unacceptable outcomes:**

- **The test passes.** Investigate: the test asserts on the import-error path, asserts something trivially true, imports the reference directly instead of the target, or matches an unrelated already-existing function in the target.
- **The test fails for an unrelated reason** (test framework misconfigured, wrong file path). Fix the environment, not the test.

If the test passes accidentally, **do not proceed.** Fix the test until it fails for the right reason.

### 3. Port (or write) the implementation

Apply the adaptations from the spec:

- Naming conventions (camelCase ↔ snake_case ↔ PascalCase).
- Type system translation (per `cross-language-notes.md` if the plan references it).
- Library substitutions per the spec's external-library plan.
- Hidden-coupling resolutions per the spec.

For **`copy` mode**, the implementation is the reference's code with adapted imports/naming. **Nothing more.**

For **`port` mode**, the implementation preserves the reference's algorithmic structure with target-idiomatic translations.

For **`learn-then-rewrite` mode**, the implementation is **independent** — you understood the reference, now you write code that satisfies the test. **You do not refer to the reference's lines while typing.** If you find yourself needing the lines, the chunk is `port`, not `learn-then-rewrite` — stop and escalate to re-classify.

Each target file gets its attribution header (see `attribution-and-license` per-file rules) **in the same commit** as the code.

### 4. Run the test

**MANDATORY.**

```bash
<target test command>
```

Confirm:

- Test passes.
- Other tests still pass.
- Output is pristine (no errors, no warnings).

**If the test fails:**

- Is the port wrong? Compare the test's assertion to what the implementation produces. Debug the implementation.
- Is the test importing something not yet ported? Check the plan: are upstream chunks already done? If not, the plan ordering is wrong — escalate.
- Is the test relying on behavior the spec marked out of scope? Confirm with the spec; if intentional, mark the test as skipped with a comment pointing at the spec section.

**Do NOT weaken the test to make it pass. The test is the spec.**

### 5. Commit

One commit per chunk, containing:

- The new/modified test file.
- The new/modified implementation file.
- The attribution header inside each.

Commit message follows `attribution-and-license`'s convention:

```
distill(<repo>): <what was distilled>

<optional body with adaptation notes>

Source: <repo>@<short-sha>:<source path>
License: <SPDX>
```

For learn-then-rewrite, use `Source-influence:` instead of `Source:`.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "The test is obviously correct, no need to run it failing first" | Step 2 is the only thing that confirms the test exercises the right code. |
| "I'll adapt the test to make it pass" | The test is the spec. Adapting it discards equivalence. Fix the impl, or escalate. |
| "Let me add some fresh tests while I'm here" | Out of scope. Equivalence first; new behavior is a separate workstream. |
| "Test and impl can be separate commits" | Splits attribution and makes review harder. One commit per chunk. |
| "Mid-port, this is actually learn-then-rewrite" | Mode shifts must be explicit. Stop, escalate, amend the spec, then continue. |
| "I'll just paste the reference in learn-then-rewrite, no one will know" | Defeats the mode and may breach the license. If you need the lines, the chunk is `port`. |
| "Let me just call the reference's code from the target" | Creates a dependency on `ref-code/` that ships nothing. Port the code. |
| "Tests were a hassle, I'll add them after I get it working" | Tests-after pass immediately and prove nothing. The cycle is RED → GREEN, in that order. |

## Red Flags - STOP and Start Over

- Implementation before test.
- Test passes on first run (before implementation exists).
- Can't explain why the test failed in step 2.
- Tests added "later" / "in a follow-up".
- Rationalizing "just this once".
- "Already manually tested the port, looks right."
- "Tests after achieve the same goals."
- "Keep the reference's code as a reference while I write the test."
- "Different language so the rule doesn't really apply."
- Mode silently shifted from `port` to `learn-then-rewrite` (or vice versa) without amending the spec.
- About to commit without the `Source:` or `Source-influence:` trailer.

**All of these mean: stop. Start over with the discipline.**

## When the Reference Has No Tests

The plan should already have decided: `fresh-equivalence-tests` or `property-based`. If neither was specified and the chunk is `learn-then-rewrite`, the plan should authorize **spot-check only** — and the chunk's success criteria are documented in the spec. Spot-check is a fallback, not a default.

If you encounter a chunk in execution that the plan did not provide a test strategy for, **stop and escalate**. Do not invent one silently.

## Cross-Language Specifics

When porting tests across languages:

- **Assertions** translate one-to-one (`expect(x).toBe(y)` ↔ `assert x == y` ↔ `assert.Equal(t, x, y)`).
- **Mocking** does not translate cleanly. Prefer reshaping the implementation to inject dependencies so tests don't need a mocking library. If you must mock, use the target's idiomatic mock library (per `cross-language-notes.md`).
- **Async tests** need the target's async test runner setup (`async def test_...` + pytest-asyncio, `it('...', async () => {})`, etc.).
- **Data fixtures** are usually fine to translate verbatim (JSON, dicts, arrays).

## Verification Checklist

Before declaring the chunk done:

- [ ] Test file exists at the target path, mirroring the reference structure.
- [ ] Test file has an attribution header.
- [ ] Implementation file has an attribution header.
- [ ] Step 2 was performed: test failed against the not-yet-ported target.
- [ ] Step 4 was performed: test passes now.
- [ ] Test command output was read fresh (not assumed from a prior run).
- [ ] Other tests in the project still pass.
- [ ] Commit message includes the `Source:` (or `Source-influence:`) trailer.
- [ ] No reference lines pasted into a `learn-then-rewrite` chunk.

Can't check all boxes? You skipped the discipline. Start over.

## Final Rule

```
Port code → ported test exists and was watched to fail first
Otherwise → not equivalence-TDD
```

No exceptions without your human partner's explicit permission, recorded in the spec.
