---
name: test-discipline
description: >
  Test-file craft: red-green vertical slices, seams not internals, tautological tests
  banned, how to assert and mock. Use when writing or changing a test file, adding a
  helper that needs coverage, or deciding whether new code needs a test. Do not use
  to pick unit vs integration vs e2e (test-strategy), or for production-only edits
  with no test impact.
---

# Test discipline

Tests verify behaviour through public interfaces, not implementation. A good test reads
like a spec ("user can checkout with a valid cart") and survives refactors.

When to add a test at all is also owned here. Which layer, and whether the test would
actually catch a bug: `test-strategy`. Signature/taste rules for production code stay
in `code-quality-standards`.

## When a test is required

- Every file that exports a utility/helper needs a dedicated unit test covering:
  - the happy path
  - edge cases (empty/boundary input)
  - error cases (invalid input, thrown/rejected paths)
- A bug fix comes with a regression test that reproduces the failure **first**.
- Non-trivial logic leaves **one runnable check** behind — the smallest thing that fails
  if the logic breaks (an assert-based self-check or one small test file). Trivial
  one-liners need no test.
- New code must not reduce existing coverage.

Types already forbid it: do not test "wrong input fails" when TypeScript blocked that
call. Do not invent cases that cannot happen.

## What a good test pins

Pin **observable behaviour**: return value, thrown error, rendered UI, or a real side
effect (HTTP, DB, file).

Do not pin private helpers, internal call order, or framework wiring.

**How many:** one happy critical path, plus one test per distinct runtime edge. Stop when
a new test does not fail for a different reason.

If `A` calls `B`, cover `B` next to `B`. `A`'s tests pin `A`'s branches only. Do not
replay `B`'s matrix through `A` unless that path is critical to success.

## Seams

A **seam** is the public boundary you test at. Tests live at seams, never against
internals. The vocabulary (module, interface, depth, seam, adapter) lives in
`codebase-design`.

Ask: "What's the public interface, and which seams should we test?" Confirm seams before
writing a pile of tests.

## Anti-patterns

- **Implementation-coupled**: mocks internal collaborators, tests private methods, or
  verifies through a side channel (querying the database instead of using the interface).
  The tell: the test breaks when you refactor but behaviour hasn't changed.
- **Tautological**: the assertion recomputes the expected value the way the code does
  (`expect(add(a, b)).toBe(a + b)`), a snapshot of whatever the code did, or a mock-only
  test that still passes if the unit is deleted. Expected values come from an independent
  source of truth: a known-good literal, a worked example, the spec.
- **Horizontal slicing**: writing all tests first, then all implementation. Bulk tests
  verify *imagined* behaviour. Work in **vertical slices**: one test → one implementation
  → repeat.

## Rules of the loop

- **Red before green.** Write the failing test first, then only enough code to pass it.
- **One slice at a time.** One seam, one test, one minimal implementation per cycle.
- **Refactoring is not part of the loop.** It belongs to review (`code-review`), not the
  red → green cycle.

## How to assert

```ts
// BAD: expected is the formula under test
expect(discount(100, "gold")).toBe(100 * 0.75)

// BAD: still green if fetchOrder is empty; the mock was the test
await fetchOrder("1")
expect(http.get).toHaveBeenCalledWith("/orders/1")

// GOOD: independent expected; mock is the network only
http.get.mockResolvedValue({ id: "1", status: "open" })
expect(await fetchOrder("1")).toEqual({ id: "1", status: "open" })
```

- **Banned:** `toMatchSnapshot`, `toMatchInlineSnapshot`, and any snapshot as the expected value.
- **Mocks:** I/O at the boundary only (HTTP, DB, clock, filesystem). Do not mock the unit
  under test. Do not mock code **you wrote**.
- **No `as any` / `as unknown as` in tests.** Build a typed factory. `as const` is allowed.
- Reuse setup: same three lines in every `it()` → extract a helper in this file.
- Imports at module scope. No `await import(...)` inside `it` / `describe` / hooks.

Don't hand-check what the test runner, typecheck, and linter already catch.
