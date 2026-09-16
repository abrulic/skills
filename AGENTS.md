# AGENTS.md

Rules for any agent working **in this repo**. This repo holds personal engineering
skills, published as plain `SKILL.md` files plus Claude/Grok plugin manifests.

## One home per fact

Each rule, table, or list lives in exactly one `SKILL.md`. Sibling skills point at
it; they do not restate it.

| Fact | Home |
|---|---|
| Laziness ladder, pre-code gate, bug-fix = root cause (one-liner) | `ponytail` |
| Forbidden patterns, SOLID, naming, signatures, ternary table | `code-quality-standards` |
| Derive types, zod only at trust boundaries | `types-from-source` |
| What/how to test, tautology ban, red-green | `test-discipline` |
| Layer choice (unit/integration/e2e), independent oracle, would-fail gate | `test-strategy` |
| Screen jobs and usability in the browser (unknown UI breakage) | `ui-verify` |
| Mobile-first layout, thumb + keyboard path, accessible names | `mobile-a11y` |
| Component library primitives, design tokens, relative units (no px) | `ui-components` |
| User-facing copy through i18n keys | `i18n-copy` |
| React Router framework mode: href(), typed routes, no DB in the route, forms | `react-router-app` |
| Deep module / seam / adapter vocabulary | `codebase-design` |
| Fowler smells, two-axis review procedure | `code-review` |
| PR title/body procedure | `pr-description` |
| Diagnose → red test → shared-function fix | `fix-bug` |

If you are about to paste a rule into a second skill, stop. Link the owner.

## Skill file conventions

- Frontmatter has `name` and `description`. The description says **when** to use the
  skill, in third person, with trigger phrases. It is not a summary of the workflow.
- Load-bearing rules stay in `SKILL.md`. `references/` is for detail agents will not
  need on the hot path.
- The body is a prompt, not documentation. No "welcome to this skill". No restating
  the description.
- Cross-links name the sibling skill (`types-from-source`), not a file path.

## Definition of done

- [ ] `skills/<name>/SKILL.md` written or updated
- [ ] Root `README.md` table and flow diagram updated in the same change
- [ ] No rule copied from another skill; the owner is the one in the table above

## Out of scope for this repo

Project stack, commands, and product rules do not belong here. Put them in that
project's `AGENTS.md`. See `examples/project-claude.md`.
