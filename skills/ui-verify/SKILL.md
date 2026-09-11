---
name: ui-verify
description: >
  Browser tests that pin user jobs and screen usability, so unknown UI/UX breakage
  goes red. Use when changing rendered UI, writing a browser/e2e test, covering a
  screen, or when the page looks wrong, is unusable, or the user says UI, UX, or
  /ui-verify. Do not use to pick unit vs integration (test-strategy) or for
  test-file craft (test-discipline).
---

# UI verify

Do not catalogue widgets. You will not know the next failure (a panel covering the
list, a clipped search, a CTA off-screen, a shared control breaking another page).
The net is the same every time: **the jobs on this screen still work, and the control
the user needs for the next step is actually usable.**

Which layer: `test-strategy` (browser). How to assert: `test-discipline`. This skill
owns **what a browser test pins** when the product is a screen.

Use the repo's existing browser runner. Do not add a new one.

## Jobs, not chrome

Before writing a browser test, list the **user jobs** on the screen you touched, in
product terms. Example for a members list: browse a member, search, filter, add.

Each browser test drives one job as a person would, then asserts the **outcome of that
job** (the list changed, the next page opened, the row is there). It does not assert
that a title string exists.

A job you did not list will not be caught. List every job the changed UI can affect,
including on **other routes** that reuse the same primitive (grep the component).

## Usable, not present

`toBeVisible()` / `getByText` stay green when the node is in the DOM and another
surface is painted on top, clipped, or off-screen. For the control the job needs
**right now** (the thing the user must click, type into, or read), assert **usable**:

1. It has a non-zero box in the viewport.
2. `elementFromPoint` at the box center lands on that control (or a descendant) — it
   is the hit target, not something covering it.

```ts
const box = el.getBoundingClientRect()
expect(box.width * box.height).toBeGreaterThan(0)
const hit = document.elementFromPoint(box.x + box.width / 2, box.y + box.height / 2)
expect(el.contains(hit) || el === hit).toBe(true)
```

That is the general pin. It fails for covering, clipping, stacking, and off-screen
controls without naming any of those.

## Coherent after every step

A screen has a default job (browse the list, read the page) and optional sub-jobs
(filter, edit, confirm). The test does not care how the sub-job is implemented.

- **During a sub-job:** the controls for *that* job are usable (apply, confirm, type,
  dismiss).
- **After it finishes or is dismissed:** the default job's controls are usable again
  (list, primary CTA, search). The user is not stuck.

Would-fail: *if a person could not finish this job, or could not use the page after,
is this test red?* If yes only when a specific widget is missing from the DOM, the
test is too narrow. Pin the job and usability.

## Widths

If the app is used on a phone and a desktop, run the same jobs at a phone width and
a desktop width. Unknown layout failures are often width-specific. Do not add a width
because of a widget type; add it because people use that width.

## Look

After the jobs run, screenshot the screen at the step that failed or at the end of
each job. Look. If a human would not ship it, the test is still too weak or the bug
is still there. Do not use pixel snapshots as the expected value (`test-discipline`).

## Procedure

1. List jobs on this screen, and on every other screen that imports the changed UI.
2. Print: `jobs: <list> on <routes> @ <widths>`.
3. For each job: drive it, assert the outcome, assert the needed control is usable,
   assert the default job is usable after.
4. Would-fail on "person cannot do the job / cannot use the page after".
5. Run. Screenshot. Look. File craft: `test-discipline`.

## Red flags

| You catch yourself | What it means | Do instead |
|---|---|---|
| Asserting a title/role exists | Presence | Job outcome + usable control. |
| A table of widget kinds to special-case | Catalogue | Jobs + usable. The next bug is not in the table. |
| Only the screen you had open | Shared UI | Grep the primitive; run jobs on each route. |
| Only one width, app is used on two | Width-specific miss | Same jobs at phone and desktop. |
| Unit test of a helper, no screen job | Wrong layer | `test-strategy` → browser, then this skill. |
| Pixel snapshot as the oracle | Flaky, banned | Usable + job outcome + look. |
