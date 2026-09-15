# CLAUDE.md — \<project\>

Stack: \<framework\> · \<bundler\> · \<styling\> · \<server\> · \<db\> · Biome · Vitest · knip · pnpm.

Cross-project taste lives in the engineering-skills pack (`ponytail`,
`code-quality-standards`, `types-from-source`, `test-discipline`, `test-strategy`,
`ui-verify`, `mobile-a11y`, `i18n-copy`, `react-router-app`, `codebase-design`).
This file only holds what is true **in this repo**.

---

## Commands

```bash
pnpm validate    # biome check + tsc + vitest + knip
pnpm dev
pnpm test
```

## Stack conventions

- React Router framework mode: `react-router-app` (this repo matches that gate).
- Locales live in `resources/locales/{lng}/common.json`. Fallback is `en`.
- After adding or renaming a route, `pnpm typegen`.

## Product rules

- User-visible changes bump `APP_VERSION` and add a changelog entry.
- Internal-only changes (refactors, tooling, CI) do not.

## Definition of done (this repo)

- Skills pack gates pass (forbidden patterns, tests, types-from-source).
- `pnpm validate` passes.

Don't hand-check what tooling enforces.
