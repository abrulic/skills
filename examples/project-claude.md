# CLAUDE.md — \<project\>

Stack: \<framework\> · \<bundler\> · \<styling\> · \<server\> · \<db\> · Biome · Vitest · knip · pnpm.

Cross-project taste lives in the engineering-skills pack (`ponytail`,
`code-quality-standards`, `types-from-source`, `test-discipline`, `test-strategy`,
`ui-verify`, `codebase-design`). This file only holds what is true **in this repo**.

---

## Commands

```bash
pnpm validate    # biome check + tsc + vitest + knip
pnpm dev
pnpm test
```

## Stack conventions

- Navigation is typesafe via `href()` from `react-router`. Never hardcode path strings.
- Typed route modules: `Route.LoaderArgs`, `Route.ComponentProps` from `./+types/<route>`.
- Server-only modules use the `.server.ts` suffix. Read env via `~/env.server`.
- DB access lives in per-feature `*.server.ts` query modules, not in routes.
- After adding or renaming a route, run `pnpm typegen`.

## Product rules

- User-visible changes bump `APP_VERSION` and add a changelog entry.
- Internal-only changes (refactors, tooling, CI) do not.

## Definition of done (this repo)

- Skills pack gates pass (forbidden patterns, tests, types-from-source).
- `pnpm validate` passes.

Don't hand-check what tooling enforces.
