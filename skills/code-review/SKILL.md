---
name: code-review
description: >
  Two-axis review of a diff since a fixed point: Standards (does it follow these
  engineering skills plus the repo's AGENTS.md/CLAUDE.md?) and Spec (does it match the
  originating issue or request?). Use when the user wants to review a branch, a PR,
  work-in-progress changes, or asks to review since X. Do not use for writing the PR
  title/body (pr-description) or for implementing the fix.
---

# Code review

Two-axis review of the diff between `HEAD` (plus a dirty working tree, if any) and a
fixed point:

- **Standards**: does the code follow this pack and the repo's documented conventions?
- **Spec**: does the code faithfully implement the originating issue / spec / request?

Both axes run as **parallel sub-agents** so they don't pollute each other's context.
Then this skill aggregates. Do not merge or rerank findings across axes.

The Fowler smell list lives **here**, not in `code-quality-standards`.

## 1. Pin the fixed point

Whatever the user said (commit SHA, branch, tag, `main`, `HEAD~5`, a PR). If they
didn't specify:

- Open PR: the PR base branch
- Otherwise: merge-base of `HEAD` and `main` (or `master` / `develop`)

Capture once:

```bash
git rev-parse <fixed-point>
git log <fixed-point>..HEAD --oneline
git diff <fixed-point>...HEAD
```

Three-dot diff (against the merge-base). If the working tree is dirty, also capture
`git diff` (unstaged + staged). A bad ref or empty diff fails here, not inside
sub-agents.

## 2. Identify the spec source

In this order:

1. A path or issue the user passed.
2. Issue references in commit messages (`#123`, `Closes #45`), fetched with `gh` if
   available.
3. A spec file under `docs/`, `specs/`, or `.scratch/` matching the branch or feature.
4. The user's original request in this conversation.

If nothing is found, ask. If they say there isn't one, skip the Spec sub-agent and
report "no spec available".

## 3. Identify the standards sources

Read, and pass to the Standards sub-agent:

1. This pack, in full: `code-quality-standards`, `types-from-source`, `test-discipline`,
   `test-strategy`, `ui-verify`, `codebase-design`, `ponytail`. Paste the smell baseline below as well.
2. The current repo's `AGENTS.md` / `CLAUDE.md` / `CODING_STANDARDS.md` if present.
   **The repo overrides this pack** where they conflict (e.g. a library that documents
   positional public APIs).

Skip anything tooling already enforces (formatter, `tsc`, dead-code scan).

### Smell baseline (Fowler, *Refactoring* ch.3)

Judgement-call heuristics, never hard violations. A documented repo standard always
wins. Each smell reads *what it is* → *how to fix*:

- **Mysterious Name** — name doesn't reveal what it does/holds → rename; if no honest
  name comes, the design's murky.
- **Duplicated Code** — same shape in >1 place → extract, call from both.
- **Feature Envy** — a method reaches into another object's data more than its own →
  move it onto that data.
- **Data Clumps** — the same few fields travel together → bundle into one type.
- **Primitive Obsession** — a primitive/string standing in for a domain concept → give
  it its own small type.
- **Repeated Switches** — same `switch`/`if`-cascade on the same type recurs →
  polymorphism or one shared map.
- **Shotgun Surgery** — one change forces scattered edits → gather what changes together.
- **Divergent Change** — one module edited for several unrelated reasons → split it.
- **Speculative Generality** — abstraction/params/hooks for needs that don't exist →
  delete, inline back.
- **Message Chains** — long `a.b().c().d()` → hide the walk behind one method.
- **Middle Man** — a thing that mostly just delegates → cut it, call the target direct.
- **Refused Bequest** — subclass ignores most of what it inherits → drop inheritance,
  use composition.

## 4. Spawn both sub-agents in parallel

If the harness has sub-agents, spawn two `general-purpose` children. If it doesn't, run
the two briefs sequentially in this session — still keep the reports under separate
headings.

**Standards brief** (under 400 words):

- The diff command, commit list, and dirty-tree diff if any.
- The standards-source files plus this smell baseline, pasted in full.
- Report, per file/hunk where relevant: (a) every place the diff violates a documented
  standard — cite the skill or file and the rule; (b) any baseline smell — name it and
  quote the hunk. Distinguish hard violations (documented-standard breaches) from
  judgement calls (smells). Skip anything tooling enforces.

**Spec brief** (under 400 words):

- The diff command and commit list.
- The spec contents.
- Report: (a) requirements the spec asked for that are missing or partial; (b) behaviour
  in the diff that wasn't asked for (scope creep); (c) requirements that look implemented
  but where the implementation looks wrong. Quote the spec line for each finding.

## 5. Aggregate

Present the two reports under `## Standards` and `## Spec`, verbatim or lightly cleaned.
Do **not** merge or rerank.

End with one line: total findings per axis, and the worst issue *within each axis* (if
any). Don't pick a single winner across axes.

A change can pass one axis and fail the other:

- Follows every standard, implements the wrong thing → Standards pass, Spec fail.
- Does exactly what was asked, breaks conventions → Spec pass, Standards fail.
