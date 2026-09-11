---
name: code-quality-standards
description: >
  Senior-level TypeScript/JavaScript taste: forbidden patterns, SOLID, naming,
  object-parameter signatures, control flow, and the ternary decision table. Use when
  writing, reviewing, or refactoring code, choosing function signatures, naming things,
  or splitting a module. Do not use for test-file craft (test-discipline), layer choice
  (test-strategy), type-source rules (types-from-source), or git/PR workflow.
---

# Code Quality Standards

Personal, stack-agnostic engineering standards. Apply whenever writing or reviewing
TypeScript/JavaScript. Project-specific conventions (framework, file layout, DB access)
belong in that project's `AGENTS.md` / `CLAUDE.md`.

How much to write, and whether to write at all: `ponytail`.
Where a type comes from: `types-from-source`.
How to test: `test-discipline`. Which layer, and whether it can catch a bug: `test-strategy`.
Screen jobs in the browser: `ui-verify`.
Module shape: `codebase-design`.

## Non-negotiable rules

- No barrel exports (`export * from`). Named re-exports at a package entry are fine.
- No type assertions (`as any`, `as Type`, `as unknown as`, `as T`, etc.). Prefer
  type-predicate guards or schema validation that narrows correctly. `as const` is
  allowed (it narrows a literal; it does not abandon the type).
- No duplicate components/functions with only minor variations — extract the shared behavior.
- No mixed responsibilities in one module/file.
- No vague utility names (`helper`, `utils2`, `doStuff`, etc.).
- No overengineering when a simpler equivalent exists.
- No unnecessary comments — rely on clear naming and structure. Only comment the
  non-obvious *why* (hidden constraint, workaround, subtle invariant).
- No deeply nested conditional logic — use early returns and guard functions.
- No non-trivial ternary chains — see **Ternaries**.
- No repeated inline validation checks across functions — extract reusable guard functions.
- No new function/component/module without first checking whether an equivalent already exists.
- No positional-parameter public APIs when input is required — use a single object
  parameter; zero-arg functions are fine when there's no input.
- No primitive/array/void return by default — prefer returning an object unless there's
  a clear reason not to (boolean predicate, pure transform).
- No explicit function return type annotations — rely on TypeScript inference.
- No scattered `process.env` access in feature/domain code — centralize behind one
  validated env module.
- No hardcoded fallback/default secrets in source code.
- No block-body + `return` for single-expression functions — use concise arrow form.
- No generic fallback values for required identifiers (e.g. `"user_1"`) — validate and
  fail fast instead.
- No exported utility/helper without a dedicated unit test (happy path + edge/error).
  How to write that test: `test-discipline`.
- No hand-written type that mirrors a query, schema, or DB shape — derive it.
  `types-from-source`.

## SOLID

- **S** — Single Responsibility: each module/file has one reason to change.
- **O** — Open/Closed: extend via composition/strategy; don't edit stable core to bolt on a feature.
- **L** — Liskov: provider/adapter modules preserve expected behavior when swapped.
- **I** — Interface Segregation: minimal, purpose-driven module/component APIs; no fat interfaces.
- **D** — Dependency Inversion: domain logic depends on internal service APIs, not concrete
  third-party SDK clients.

## Naming

Describe intent and result, not mechanism.

- Functions: `buildRecommendationContext`, `mapWebhookToPaymentEvent`, `resolveOverallStatus`
- Components: `ChatMessageList`, `CheckoutStatusBanner`, `ProductGridCard`
- Provider/adapter modules: `stripePaymentProvider`, `anthropicLlmProvider`
- Files and folders must be self-explanatory without opening the file.
- File names are **kebab-case, always** — components included: `member-avatar.tsx` exports
  `MemberAvatar`. A project `AGENTS.md` may carve out framework exceptions (flat-route
  filenames, Next.js special files).

## Function signatures and objects

- Accept a single object parameter when input exists: `fn({ userId, limit })`, not
  `fn(userId, limit)`.
- Return an object by default; use primitive/array/void only when clearly justified.
- Don't annotate return types — let inference do it.
- Concise arrow form (no braces + `return`) for single-expression bodies.
- Object-parameter inputs with 2+ properties: extract a named input type. Exactly 1
  property: keep it inline in the signature.
- Object-literal shorthand: when values are already in scope, write `{ foo, bar }` —
  never `{ foo: source.foo, bar: source.bar }`. Destructure the source first if you
  need to reuse it. Applies to component props, JSX object literals, and any returned object.

## Control flow

- Early returns and guard clauses over nested conditionals; extract a guard function
  instead of multiple nested conditions.
- Prefer `&&` and `??` where they read more clearly than an `if`.
- In components: `if (x) return <A />` over `x ? <A /> : <B />` when branching between
  significant sections; reserve ternaries for simple one-line value selection.

## Ternaries — the decision table

A ternary is allowed only when it picks **one value** on **one logical line**. Anything
else has a better shape. Work down this table; the first row that matches wins.

| You have | Write instead |
|---|---|
| `{x ? <El /> : null}` | `{x && <El />}` |
| `{x ? null : <El />}` | `{!x && <El />}` |
| `{x ? (<A />) : (<B />)}` spanning lines | two guards: `{x && (<A />)}` then `{!x && (<B />)}` |
| A branch that *is* the component's output | extract a component and `if (x) return <A />` |
| `a ?? (b ? c : d)` — a fallback chain | a guard function with one early return per case |
| A ternary nested in another ternary — including inside a template literal | a guard function with early returns. Never nest. |
| The same condition branched 3+ times in one render | derive **one** object from it, then read fields off that |
| Two branches rendering near-identical markup | one component with a prop; the branches were duplication |

Leave these alone — "fewer ternaries" is not a goal, and rewriting these makes the code worse:

```tsx
const date = typeof value === "string" ? new Date(value) : value
className={({ isActive }) => cn(base, isActive ? "on" : "off")}
<Badge variant={overdue.length ? "destructive" : "success"} />
```

An early return that has to duplicate the surrounding call to avoid a ternary is not an
improvement — it has traded a branch for duplication.

Two traps when converting to `&&`:

- **Numbers render.** `{items.length && <List />}` prints a literal `0` when the array
  is empty. Always compare: `{items.length > 0 && <List />}`.
- **Guard functions returning class names** should return the *whole* resolved value,
  not a partial one the caller then has to `??` a fallback onto.

```tsx
// bad — the fallback chain is still at the call site
const style = session?.status ? statusStyles[session.status] : null
cn(base, style?.cell ?? (session ? "bg-muted/40" : "text-muted-foreground"))

// good — one guard function, every case stated once, caller just reads it
const resolveDayStyle = ({ session }: { session: Session | undefined }) => {
	if (!session) return { cell: "text-muted-foreground", dot: null }
	if (!session.status) return { cell: "bg-muted/40", dot: "bg-muted-foreground" }
	return statusStyles[session.status]
}
```

## Duplication and reuse

- Before adding any function/component/module, search for an existing equivalent.
- If the same logic exists in multiple places, extract a shared utility/service/component.
- Shared UI/behavior belongs in a reusable primitive, not repeated per call site.

## Definition of done

- No forbidden patterns in the changed files; the laziness ladder was actually climbed.
- New code follows SOLID, naming, and the signature/control-flow rules above.
- Duplicate logic is reduced, not increased.
- Every new exported utility has a passing unit test with edge/error coverage.

Don't hand-check what tooling enforces (formatter, typecheck, dead-code scan, test
runner). Spend attention on the judgement calls above.
