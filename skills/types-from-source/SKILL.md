---
name: types-from-source
description: >
  Derive types from the schema, query, or validator that owns them. Use when adding a
  type, interface, union, zod schema, Prisma/ORM query, loader/action return, or a
  component prop that mirrors stored or parsed data. Also when about to write
  `interface X { id: string ... }` next to a query. Do not use for test-file craft or
  git/PR workflow.
---

# Types from source

The schema (ORM model, SQL column, zod parser at a trust boundary) is the source of
truth. Propagate it by **inference**, never by re-typing it.

If adding a value in one place requires editing a second place, it was a mirror.

## Derive it instead

```ts
// bad — a hand-written mirror of the query's select. Nothing links them, so it drifts
// silently, and `string | Date` is the author hedging about what the loader returns.
interface ActivityEntry {
	id: string
	createdAt: string | Date
	user: { firstName: string; lastName: string }
}

// good — the select IS the type. Change the query and every consumer updates or fails.
export type RecentActivityEntry = Awaited<ReturnType<typeof listRecentActivity>>["entries"][number]
```

The query `select` is the obvious one. These are the ones that get missed:

| About to write | Derive it instead |
|---|---|
| A union of DB enum values — `role: "ADMIN" \| "TRAINER"` | The generated enum (`$Enums.UserRole`, `UserRole`, …) |
| A runtime check for those values | `Object.hasOwn(Enum, value)` when keys and values are identical |
| A shape matching a query's `select` | `Awaited<ReturnType<typeof listThing>>["items"][number]` |
| A list of a schema's field names | `keyof z.infer<typeof schema>` |
| The shape of validation errors | `z.inferFlattenedErrors<typeof schema>["fieldErrors"]` |
| A component prop mirroring any of the above | import the derived type; never restate it in the props |
| An identifier that is a model field | `User["id"]`, not `string` |

- Export derived types from the module that owns the query or schema. Components import
  them with `import type`.
- A ceremonial widening like `string | Date` is a symptom: derive the type and the real
  one appears.
- **Write inputs are not read shapes.** `RecordActivityInput` (`userId: string`) is a
  different type from the row you read back (`user: { firstName, lastName }`). Keep both;
  they aren't duplication.

## Where zod belongs: trust boundaries, and nowhere else

| Data | Type from |
|---|---|
| Form data, URL/search params, env vars, external API responses, webhooks | **zod** — untrusted bytes, parse them |
| Query results, loader→component props, internal function inputs | **inference** — already typed by the schema/query, nothing to validate |

A zod schema for DB-shaped data is a *second* source of truth: adding a column then
means editing two files, which is the drift you were trying to remove — plus runtime
parsing cost for no safety.

Prefer a dedicated parse/guard (`parseLoginInput`) wrapping `schema.parse(...)` over
repeating inline parse blocks. After parsing, destructure once; don't keep reading
`parsedInput.field`.

## Don't

- Don't invent a parallel `interface` for a query result, Prisma model, or zod output.
- Don't annotate `userId: string` when `User["id"]` is available.
- Don't write a zod schema for data that never crosses a trust boundary.
- Don't scatter `process.env` — one validated env module owns that parse.
