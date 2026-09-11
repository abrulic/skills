---
name: pr-description
description: >
  Write a pull request title and body from the diff, then post with gh. Use when writing
  a PR title or body, when about to run gh pr create, when about to git push on a branch
  that already has an open PR, or when the user says /pr-description. Do not use for
  commit messages, changelogs, or review comments.
---

# PR description

Write the GitHub PR title and body from the **diff**, then post. Do not wait for
approval. Do not write from the branch name or from memory.

## When to run

- The user asks for a PR title or body, or types `/pr-description`.
- Immediately before `gh pr create`.
- Immediately after an **agent** `git push` on a branch that already has an open PR
  (`gh pr edit` with a fresh title and body).

Do not run it on a human-only `git push` in another terminal. Do not add a git hook.

## Procedure

### 1. Ground in the repo

1. Find the base branch (`main` / `master` / `develop`).
2. Read `git log --oneline base...HEAD` and `git diff base...HEAD`.
3. Read `.github/pull_request_template.md` if it exists (also `PULL_REQUEST_TEMPLATE.md`
   and `.github/PULL_REQUEST_TEMPLATE/`).
4. Read about 15 recent PR titles (`gh pr list --limit 15 --state all --json title`).
   Match that shape.
5. Collect linked issues from branch name, commit messages, and `Closes #` / `Fixes #`.

### 2. Classify

| Kind | Signal |
|---|---|
| **fix** | Restores broken behaviour. A user-visible bug, a failing test, a regression. |
| **feat** | Adds behaviour that did not exist. |
| **chore / docs** | Agent files, CI, docs-only, refactors with no user-visible behaviour change. |

If both a feat and a fix are in the diff, the larger user-visible story wins. Say so
in the lead.

### 3. Feat gate

A **feat** PR cannot be posted until the branch has at least one of:

- A new or updated automated test that covers the new behaviour
- A command a reviewer can copy and run
- A change in an official example app that shows the new behaviour

If none exist, stop. Tell the user what is missing. Do not run `gh pr create` or
`gh pr edit`. A **fix** or **chore / docs** PR does not use this gate.

### 4. Title

Match this repo's recent PR titles. In a conventional-commit repo, write
`type(scope): summary`. One line. Must still make sense if the reviewer never opens
the body.

### 5. Body

**Lead (required, 1 to 4 sentences, before the template).** No preamble. No "This PR
aims to". Start with the fact.

- **feat:** what the PR does for a user of the product.
- **fix:** what the bug is, then how this PR fixes it.
- **chore / docs:** what changed, in product or repo terms, not a file list.

**Repo template** — fill every section honestly. Leave a checkbox unchecked when the
claim is false.

Then these extra sections. Testing and Risk / rollback are always present. Linked
issues and Public API change are omitted when they do not apply.

#### Testing

1. **Commands run.** What you ran, what passed, what you skipped and why. If you ran
   nothing, say so.
2. **Manual test.** Numbered steps a reviewer can follow. For a **fix**, step 1 is how
   to reproduce the old bug, then how to confirm it is gone.
3. **How this PR makes testing easy.** Name the artifacts on the branch. If there are
   none (allowed for fix and chore / docs), write that.

#### Linked issues

Omit when nothing links. Otherwise `Closes #N` or `Fixes #N`.

#### Risk / rollback

What could break. How to undo. If risk is low, say that in one line. Do not skip the
heading.

#### Public API change

Omit when the PR does not touch the published surface (exported functions, types,
components, CLI flags, env vars, documented contracts of **published** packages).
Tests, internals, skills, `AGENTS.md` do not count.

When it does, show **caller usage**, not the internal diff: a short Before snippet and
a short After snippet.

### 6. Post immediately

- No PR yet: `gh pr create --title "..." --body-file ...`
- PR already open: `gh pr edit --title "..." --body-file ...`

If `gh` fails, stop and show the error. Leave the drafted body in the chat.

Do not attach screenshots. Do not wait for the user to approve the text.
