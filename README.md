# Engineering skills

Personal agent skills I use across every project. Each rule has one home and only
loads when the task matches.

Plain `SKILL.md` files. They work in **Grok**, **Claude Code**, **Codex**, **Cursor**,
**Copilot CLI**, and **Gemini**.

## The skills

| Skill | When | Kind |
|---|---|---|
| [ponytail](skills/ponytail/SKILL.md) | Before writing or editing any code | auto |
| [code-quality-standards](skills/code-quality-standards/SKILL.md) | Writing, reviewing, naming, signatures | auto |
| [types-from-source](skills/types-from-source/SKILL.md) | Adding a type, schema, query, or prop that mirrors data | auto |
| [test-discipline](skills/test-discipline/SKILL.md) | Writing a test file: asserts, tautology, red-green | auto |
| [test-strategy](skills/test-strategy/SKILL.md) | Unit vs integration vs e2e; tests that would fail if the product is wrong | auto |
| [ui-verify](skills/ui-verify/SKILL.md) | Browser tests for screen jobs and usability; unknown UI breakage goes red | auto |
| [mobile-a11y](skills/mobile-a11y/SKILL.md) | Mobile-first UI; every job works with a thumb and a keyboard | auto |
| [ui-components](skills/ui-components/SKILL.md) | Which UI primitive to use; one token vocabulary; no px in classes | auto |
| [i18n-copy](skills/i18n-copy/SKILL.md) | User-facing strings go through i18n keys, not JSX literals | auto |
| [react-router-app](skills/react-router-app/SKILL.md) | Framework-mode RR: href(), typed routes, no DB in the route file | auto |
| [codebase-design](skills/codebase-design/SKILL.md) | Designing a module, placing a seam | auto |
| [fix-bug](skills/fix-bug/SKILL.md) | A live bug or regression | auto + `/fix-bug` |
| [code-review](skills/code-review/SKILL.md) | Review a branch, PR, or uncommitted diff | `/code-review` |
| [pr-description](skills/pr-description/SKILL.md) | Opening or updating a GitHub PR | `/pr-description` |

Auto skills also run as slash commands (`/ponytail`, `/test-discipline`, …).

```
ponytail                 → do I write this at all?
code-quality-standards   → how the code looks
types-from-source        → where the type comes from
test-discipline          → how the test is written
test-strategy            → which layer, and would it catch a bug
ui-verify                → screen jobs and usability in the browser
mobile-a11y              → mobile-first, thumb + keyboard
ui-components            → library primitive, one token set, rem not px
i18n-copy                → user-facing copy is a translation key
react-router-app         → href(), typed routes, data in domain modules
codebase-design          → how deep the module is
fix-bug                  → symptom → cause → red test → one fix
code-review              → Standards axis ∥ Spec axis
pr-description           → title + body from the diff, posted
```

## Install

The canonical layout is `skills/<name>/SKILL.md`. Point your agent at that folder.
Don't copy the files if you can help it — a path or symlink stays in sync.

Public source: https://github.com/abrulic/skills

### New laptop

```bash
git clone git@github.com:abrulic/skills.git ~/Desktop/skills
mkdir -p ~/.grok/skills ~/.claude/skills ~/.agents/skills
ln -sfn ~/Desktop/skills/skills/* ~/.grok/skills/
ln -sfn ~/Desktop/skills/skills/* ~/.claude/skills/
ln -sfn ~/Desktop/skills/skills/* ~/.agents/skills/
```

Grok also needs:

```toml
# ~/.grok/config.toml
[skills]
paths = ["~/Desktop/skills/skills"]
```

Then open a **new** session in any project.

### Grok (this machine is already wired)

`~/.grok/config.toml` has:

```toml
[skills]
paths = ["~/Desktop/skills/skills"]
```

Or as a plugin, once this repo is a git remote:

```bash
grok plugin marketplace add ./Desktop/skills
grok plugin install engineering-skills --trust
```

### Claude Code

Plugin (after the repo is on GitHub):

```
/plugin marketplace add <owner>/skills
/plugin install engineering-skills@<owner>
```

Drop-in, no plugin:

```bash
mkdir -p ~/.claude/skills
ln -sfn ~/Desktop/skills/skills/* ~/.claude/skills/
```

### Codex, Copilot CLI, Cursor, Gemini

They all honor `~/.agents/skills/`:

```bash
mkdir -p ~/.agents/skills
ln -sfn ~/Desktop/skills/skills/* ~/.agents/skills/
```

Gemini also has `gemini skills install <repo-url> --path skills`.

## What stays in a project's CLAUDE.md

These skills are stack-agnostic. A project's `AGENTS.md` / `CLAUDE.md` should only
hold what is true **here** and not elsewhere:

- Stack and commands (`pnpm validate` = biome + tsc + vitest + knip)
- File-layout exceptions (route filenames, Next.js special files)
- Product rules (changelog, which locales exist, …)

`href()`, typed routes, and “no DB in the route file” are `react-router-app` (only in
framework-mode apps). User-facing copy is `i18n-copy` (only if the repo already has
i18n).

See [examples/project-claude.md](examples/project-claude.md) for a slim template.

Where a project rule conflicts with a skill, **the project wins**.

## License

MIT. Fork them, edit them, make them yours.
