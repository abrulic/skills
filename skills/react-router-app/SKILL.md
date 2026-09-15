---
name: react-router-app
description: >
  React Router framework-mode conventions: href(), typed route modules, loaders and
  actions, no DB in the route file. Use when the repo is React Router framework mode
  (react-router.config, flatRoutes, app/routes) and you are adding a route, loader,
  action, Link, Form, or redirect. Do not use for library/CLI packages or the data
  router without framework mode.
---

# React Router app

**Stop unless this repo is framework mode.** Signals: `react-router.config.ts` (or
`.js`), `flatRoutes()` / `app/routes.ts`, `@react-router/dev`, route files under
`app/routes/`. If those are missing, this skill does not apply.

Use the framework's typesafe primitives. Do not hand-roll path strings or untyped
loaders.

## Paths

Every internal path goes through `href()` from `react-router`: `<Link to>`,
`<NavLink to>`, `<Form action>`, `redirect(...)`.

```ts
href("/members")
href("/members/:memberId", { memberId })
redirect(href("/login"))
```

Never `" /members/" + id`. Path patterns come from the route filenames.

After adding or renaming a route, run the repo's typegen (`pnpm typegen` or whatever
`package.json` names). `href()` and `Route.*` types will not see the file until then.

## Route modules

Import per-route types from `./+types/<route>`: `Route.LoaderArgs`,
`Route.ActionArgs`, `Route.ComponentProps`, `Route.MetaArgs`.

Read data from the typed `loaderData` / `actionData` props. Do not re-annotate
`useLoaderData()` by hand when `Route.ComponentProps` is there.

## Data flow

- Loaders and actions return **plain objects**. `data()` only when you need a status
  or headers. `redirect()` for navigation.
- Mutations: `<Form>` + `useNavigation` for pending UI. Not ad-hoc `fetch` to your
  own action unless the framework cannot do that job.
- Submit to the **current route's `action`** by default. A resource route only when
  one page must own several distinct actions.

Types of loader results: `types-from-source` (derive from the query, do not mirror).

## Server vs client

- Server-only modules use the `.server.ts` suffix.
- Env: one validated env module (`code-quality-standards`). Not `process.env` in
  feature code.
- **No database in the route file.** Query/load in a domain module (`*.server.ts` or
  the repo's service/query folder). The route calls that function and returns its
  result.

## Red flags

| You catch yourself | What it means | Do instead |
|---|---|---|
| `to="/members"` or `redirect("/login")` | Untyped path | `href(...)`. |
| `prisma` / `db.` inside `app/routes/` | Data layer in the route | Domain `*.server.ts` (or this repo's query module). |
| `useLoaderData()` + a hand-written type | Mirror type | `Route.ComponentProps` / `loaderData`. |
| `fetch("/api/...")` for a form you own | Hand-rolled mutation | `<Form>` to the route action. |
| New route, skipped typegen | `href` and `Route` are stale | Run typegen. |
| Applying this in a CLI or library package | Wrong repo | Stop. |
