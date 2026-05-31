---
name: equivalence-testing
description: Opt-in tool for code distilling. Use when the distillation spec marks a chunk testable (pure functions, clear inputs/outputs) and you want objective, reference-derived evidence. Rigid RED-GREEN discipline; tests are derived from the reference, not invented.
---

# Equivalence Testing (opt-in)

Objective evidence that a distilled chunk behaves like its reference. **Tests are derived from the reference, not invented from requirements** — that is what makes this an equivalence check and not a from-scratch test.

This is opt-in. The distillation spec decides, per chunk, whether it applies. Use it for testable chunks; for fuzzy output (renderers, generative text) rely on `gap-report` instead.

**Type:** Rigid when chosen. If you opt in, follow the cycle exactly — a half-followed discipline gives false confidence.

## When to Use

- ✅ Pure functions, algorithms, clear input→output, deterministic behavior.
- ✅ The reference has usable tests, OR you can capture input/output pairs by running it, OR the behavior has checkable properties (idempotent, length-preserving, sorted).
- ❌ Rendered/visual output, generative/LLM output, behavior with no stable oracle → use `gap-report` reading-based verification instead.

## The Cycle

```dot
digraph equivalence {
    rankdir=LR;
    test [label="PORT TEST\nfrom reference", shape=box, style=filled, fillcolor="#ffcccc"];
    red [label="Watch it FAIL\nfor the right reason", shape=diamond];
    impl [label="PORT / REWRITE\nimplementation", shape=box, style=filled, fillcolor="#ccffcc"];
    green [label="Watch it PASS\nclean", shape=diamond];
    commit [label="COMMIT", shape=ellipse];

    test -> red;
    red -> impl [label="yes"];
    red -> test [label="passed / wrong failure"];
    impl -> green;
    green -> commit [label="yes"];
    green -> impl [label="no"];
}
```

**Core principle:** if you didn't watch the test fail against the not-yet-ported code, you don't know it tests the right thing.

## Steps

1. **Write the test FIRST, from the reference.** Choose the source per the spec's test strategy:
   - **port-reference-test:** translate the reference's test cases to your framework; preserve assertions and data shape.
   - **captured-io:** encode input/output pairs captured from running the reference; cite the run in a comment.
   - **property-based:** encode the properties the spec lists, in your property-testing library (fast-check, hypothesis, proptest).
2. **Run it against the unimplemented/stub target — watch it FAIL for the right reason.** An import error or a wrong-output assertion is acceptable. If the test *passes*, it's testing the wrong thing (a mock, the reference, something trivially true) — fix it before proceeding.
3. **Port or rewrite the implementation** per the chunk's mode and keep-verbatim items.
4. **Run the test — watch it PASS, clean** (and other tests still pass). Do NOT weaken the test to make it pass; fix the implementation. The test is the contract.
5. **Commit** — test + implementation together, one commit per chunk.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Obviously correct, skip the failing run" | Step 2 is the only proof the test exercises the right code. |
| "I'll adapt the test to make it pass" | The test is the contract; adapting it discards equivalence. Fix the impl. |
| "Tests after, once it works" | Tests-after pass immediately and prove nothing. RED before GREEN. |
| "Different language, the rule doesn't apply" | Assertions translate one-to-one; the discipline holds. |

## Cross-Language Notes

- Assertions map one-to-one (`expect(x).toBe(y)` ↔ `assert x == y` ↔ `assert.Equal(t, x, y)`).
- Data fixtures (JSON, dicts, arrays) usually translate verbatim.
- Mocking does not translate cleanly — prefer dependency injection over a mocking library.
