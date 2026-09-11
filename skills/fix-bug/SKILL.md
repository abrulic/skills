---
name: fix-bug
description: >
  Root-cause bug fix: hypotheses on the hot path, a failing test first, then fix the
  shared function once (including sibling callers). Use when the user reports a bug,
  a failing test, a regression, or says /fix-bug or "fix this". Do not use for new
  features (ponytail + code-quality-standards + test-discipline) or for review.
---

# Fix bug

A report names a symptom. Fix the cause once. Do not patch only the path the ticket
names.

`ponytail` still applies: understand first, then the smallest diff that holds.
`test-discipline` owns how the regression test is written.

## Loop

### 1. Understand the hot path

Read the task and the code it touches. Trace the real flow end to end.

Grep every caller of the function you suspect. List them. A sibling caller that does
the same wrong thing is in scope even if the ticket didn't name it.

Write **at least three hypotheses** that could produce this symptom on that path.
Don't start editing until you have them.

### 2. Reproduce, red

Write a failing test (or the smallest runnable check) that goes red on this bug
**before** changing production code. If you cannot reproduce, stop and say so — don't
guess a fix.

The test pins observable behaviour (`test-discipline`) at the cheapest layer that can
go red on this bug (`test-strategy`). A broken or unusable screen: `ui-verify`. It is
the regression test you will keep.

### 3. Eliminate, then fix

If third-party debug logging exists, use it. Temporary logs on the suspected branches
are allowed; delete them before you finish.

Eliminate hypotheses. Fix the **shared function** (or the shared type/guard), not each
call site. One guard there is a smaller diff than one per caller.

Scan opened files for the same pattern and fix those too. Do not mechanical-replace
the rest of the monorepo.

### 4. Green, then nearby

The new test is green. Package tests still pass.

If the same class of bug exists next to the one you fixed (same guard missing on a
sibling field, same parse skipped on another route), fix it in this change. That's
still the root cause, not scope creep.

### 5. Stop

Don't add speculative features, extra abstractions, or leftover debug. Don't "while
I'm here" drive-by refactors.

If the user wants a review of the fix, that's `/code-review`, not this skill.

## Don't

- Don't patch the ticket's call site and leave a sibling broken.
- Don't ship a fix without a test that was red on the old code.
- Don't treat a flake as a flake until you've ruled out the cause.
- Don't start with "add a try/catch and return a default". Fail fast at the trust
  boundary; defaults for required identifiers are forbidden
  (`code-quality-standards`).
