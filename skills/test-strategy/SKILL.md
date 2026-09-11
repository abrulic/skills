---
name: test-strategy
description: >
  Pick the right test layer and write tests that would fail if the product is wrong.
  Use when adding unit, integration, or e2e tests, covering existing code, or when
  tests are all green while a bug remains. Also when the user says intelligent tests,
  testing pyramid, or /test-strategy. Do not use for screen jobs and usability
  (ui-verify), assertion-file craft (test-discipline), or a known-bug fix (fix-bug).
---

# Test strategy

A test that stays green while the product is wrong is not a test. It is a snapshot of
the bug.

How to write the assertion, tautology, red-green loop, mocks, snapshots: `test-discipline`.
This skill owns **which layer** and the **would-fail gate**. Screen jobs and
usability in the browser: `ui-verify`.

## Hard gates

### 1. Independent oracle

Expected values come from one of:

- the user's stated rule
- a worked example they gave
- a domain table / spec
- a known-good fixture that did not come from running this code

They do **not** come from reading the current implementation and copying its output,
logging `fn(x)` and pasting the result, or asking the model to "match existing
behaviour" unless the user explicitly asked for a characterization test.

If you do not know the correct expected, **ask**. A missing test is better than a green
test that freezes a bug.

### 2. Would-fail

Before keeping a test, name **one production change** that would make it red. Examples:
the function body deleted, the known bug left in, the SQL filter dropped, the redirect
removed.

If you cannot name that change, the test is decorative. Delete it or rewrite it.

If the user said there is a bug and the new test is green on current code, you wrote
the bug into the expected. Stop. Fix the oracle or move the layer — do not "update the
test to pass".

### 3. Characterization is opt-in

Freezing current behaviour is allowed only when the user asked to characterize or lock
legacy output before a refactor. Name those tests `characterization: …`. Default is
**not** characterization. Covering existing code is still a spec test: ask what *should*
happen, then run. Red means you found a bug (good). Green means the code matches the
oracle, or the oracle is wrong.

## Pick the layer

Choose the **cheapest layer that can go red** on this failure. First row that matches
wins.

| The thing that can be wrong | Layer | Run it as |
|---|---|---|
| Pure function, mapper, guard, schema parse, date/money math | **unit** | In-process, no I/O. Next to the source (`foo.test.ts`). |
| Wiring you own: loader/action → query → DB, service → real collaborator, env, cookies | **integration** | Real DB / filesystem / HTTP adapter. `*.server.test.ts` or the repo's integration folder. Mock only the outer world you don't own. |
| A person doing a flow, or the rendered screen is wrong/unusable in a way you have not named | **e2e / browser** | The repo's browser runner. Pin jobs and usability, not DOM presence: `ui-verify`. |

Wrong-layer tells:

- A browser test that only asserts a helper's return → that is a unit test. Move it down.
- A browser test that only asserts a string is in the document → the screen can still
  be unusable. `ui-verify`.
- A unit test that mocks `*.server.ts` you wrote to "prove" a loader → that is a fake
  integration. The bug in SQL/cookies/redirects will never go red. Move it up, use the
  real collaborator.
- The same assertion at two layers → keep it at the lowest layer that can fail. The
  higher layer may smoke the path; it does not replay the matrix.

If a unit test would still pass with the bug (the bug is in wiring, SQL, a cookie, a
redirect, a form post), you picked the wrong layer. Move up.

## Pyramid

- **Many unit** — every exported util/guard/schema (see `test-discipline` for when).
- **Some integration** — each write path that hits storage or an adapter you own; each
  loader/action whose bug would not show up in a unit test.
- **Few e2e** — one happy critical path per user-facing feature, plus runtime edges that
  only exist in the browser (focus, empty view, client navigation). Not a second copy of
  the unit matrix.

## Procedure

1. **Name the behaviour** in one sentence, in product terms ("search ignores diacritics",
   "login rejects a used code").
2. **Name the wrongness** this test must catch (the bug class).
3. **Pick the layer** from the table. Print one line:
   `layer: unit | integration | e2e — <why this is the cheapest that can go red>`
4. **Write expected from the oracle**, then the test. Do not run the implementation to
   discover expected.
5. **Would-fail check.** State the production change that makes it red.
6. **Run it.**
   - Red on a new feature: implement until green (`test-discipline` loop).
   - Red on existing code: you found a bug. That is `/fix-bug`, not "edit expected".
   - Green while a known bug remains: gate 2 failed. Rewrite.
7. File craft: `test-discipline`.

## Integration

- Use the real collaborator behind the seam (test database, real query module). Seed the
  minimum row. Assert the persisted data or the loader/action result.
- Mock only I/O you do not own (Stripe, email, clock). What may be mocked: `test-discipline`.
- Match the repo's existing harness (testcontainers, a `.server.test.ts` project, an
  in-memory DB the app already uses). Do not add a new stack.

## E2E / browser

- Drive as a user: go to the URL, click, fill, assert what's on screen or in the
  resulting record.
- One flow per test. No 20-step novels.
- Auth: reuse the repo's test helper or session factory. Don't click through signup
  unless signup is the feature.
- What to pin on a screen (jobs, usability, other routes, widths): `ui-verify`.

## Red flags

| You catch yourself | What it means | Do instead |
|---|---|---|
| Expected is whatever `console.log(fn(x))` printed | Oracle is the implementation | Gate 1. Ask, or use a worked example. |
| Suite is green, user says there is a bug | You froze the bug | Gate 2. Test must be red on current code. |
| Mocked your own query/service in a "unit" of the loader | Wrong layer | Integration, real module. |
| Browser test for `searchKey("Hadžić") === "hadzic"` | Wrong layer | Unit, next to `searchKey`. |
| String-in-the-document as the only pin | Screen can be unusable and still green | `ui-verify`: job + usable control. |
| Three layers asserting the same parse | Matrix replayed | Keep the lowest layer; higher layer smokes the path. |
| Added Playwright because the repo only has Vitest browser | New runner | Use what is already there. |
| "I'll write tests that match current behaviour so CI passes" | Characterization without consent | Gate 3. Spec test, or ask. |
