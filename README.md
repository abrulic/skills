# Engineering skills

Personal agent skills I use across every project. Extracted from the standards that
used to live as a 340-line `CLAUDE.md` in [prokick](../prokick), then split so each
rule has one home and only loads when the task matches.

Plain `SKILL.md` files. They work in **Grok**, **Claude Code**, **Codex**, **Cursor**,
**Copilot CLI**, and **Gemini**. No process framework, no issue-tracker setup, no
marketing pack — those already live in [mattpocock/skills](https://github.com/mattpocock/skills)
and [AlemTuzlak/skills](https://github.com/AlemTuzlak/skills).

## The skills

| Skill | When | Kind |
|---|---|---|
| [ponytail](skills/ponytail/SKILL.md) | Before writing or editing any code | auto |
| [code-quality-standards](skills/code-quality-standards/SKILL.md) | Writing, reviewing, naming, signatures | auto |
| [types-from-source](skills/types-from-source/SKILL.md) | Adding a type, schema, query, or prop that mirrors data | auto |
| [test-discipline](skills/test-discipline/SKILL.md) | Writing a test file: asserts, tautology, red-green | auto |
| [test-strategy](skills/test-strategy/SKILL.md) | Unit vs integration vs e2e; tests that would fail if the product is wrong | auto |
| [ui-verify](skills/ui-verify/SKILL.md) | Browser tests for screen jobs and usability; unknown UI breakage goes red | auto |
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
codebase-design          → how deep the module is
fix-bug                  → symptom → cause → red test → one fix
code-review              → Standards axis ∥ Spec axis
pr-description           → title + body from the diff, posted
```

## Install

The canonical layout is `skills/<name>/SKILL.md`. Point your agent at that folder.
Don't copy the files if you can help it — a path or symlink stays in sync.

### Grok (this machine is already wired)

`~/.grok/config.toml` has:

```toml
[skills]
paths = ["~/Desktop/skills/skills"]
```

Edits in this repo show up in the next session. To install on another machine:

```bash
git clone <this-repo> ~/Desktop/skills
# then add the [skills] paths block above
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
- Framework primitives (`href()`, `.server.ts`, flat-routes)
- File-layout exceptions (route filenames, Next.js special files)
- Product rules (changelog on the Info page, i18n locale, …)

See [examples/project-claude.md](examples/project-claude.md) for a slim template.
The old prokick `CLAUDE.md` can drop §§1–6 and keep the stack + product sections.

Where a project rule conflicts with a skill, **the project wins**.

## What I did not add (yet)

Useful, but they are a different job than "my taste, every repo":

| If you want | Use / steal from | Why it isn't here |
|---|---|---|
| Grill the spec before coding | mattpocock `/grill-me` | Alignment ritual, not a standard |
| Hard/perf diagnosis loop | mattpocock `/diagnosing-bugs` | Heavier than `fix-bug` |
| Architecture survey | mattpocock `/improve-codebase-architecture` | Periodic, not per-change |
| Conventional commits | a small `/commit` skill | I haven't needed a skill for this |
| Docs, RFC, changelog, social | [AlemTuzlak/skills](https://github.com/AlemTuzlak/skills) | Already a pack |
| Official Ponytail plugin (lite/full/ultra, audit) | [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) | This repo keeps the inlined ladder only |

Add one of those when the prompt has been typed twice.

## Attribution

- Laziness ladder: [Ponytail](https://github.com/DietrichGebert/ponytail) (MIT)
- Deep modules, two-axis review, TDD anti-patterns: [mattpocock/skills](https://github.com/mattpocock/skills)
- Taste (object params, no return annotations, no barrels, types-from-source, ternary table): the prokick `CLAUDE.md` / `code-quality-standards` skill

## License

MIT. Fork them, edit them, make them yours.
