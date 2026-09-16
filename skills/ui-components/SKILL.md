---
name: ui-components
description: >
  Build screens from the repo's component library instead of hand-rolled markup, and
  size everything in relative units off one token scale. Use when adding or restyling a
  button, input, dialog, card, toggle, badge or any control, when writing className, or
  when the user says shadcn, component library, design tokens, or /ui-components. Do not
  use for layout, thumb reach and keyboard paths (mobile-a11y), browser tests
  (ui-verify), or TypeScript taste (code-quality-standards).
---

# UI components

Two rules: the control already exists, and nothing is measured in pixels.

Layout, thumb reach and keyboard path: `mobile-a11y`. Screen jobs in a browser:
`ui-verify`. Prop types and signatures: `code-quality-standards`. Copy: `i18n-copy`.

## Use the library

**shadcn/ui is the default.** Before writing a `<button>`, a bordered `<div>` that is
really a card, or a `role="switch"` you are about to wire by hand, check the repo's ui
folder — and if it is not there yet, add it (`pnpm dlx shadcn@latest add <name>`) rather
than reinventing it. Radix gives you the focus trap, the roving tabindex, `aria-*` and
Escape; a hand-rolled version gets one of those subtly wrong.

Reach for the primitive that matches the **job**, not the markup you had in mind:

| You are about to write | Use |
|---|---|
| `<button>` with variant classes | `Button` with `variant` / `size` |
| A bordered div with a title and body | `Card` |
| `role="switch"` + `aria-checked` | `Switch` |
| A row of buttons where one is "active" | `ToggleGroup` |
| A `<dialog>`, or a div with a backdrop | `Dialog` |
| `<label>` + `<input>` + an error `<p>` | `FormItem` / `FormLabel` / `FormControl` / `FormMessage` |
| `<p role="alert">` | `Alert` |
| A link that should look like a button | `<Button asChild><Link …></Button>` |

What the library does not have — anything carrying **domain** meaning — is yours to
write, composed from the primitives. A unit-aware number box is a domain component that
renders `Input`; it is not a reason to hand-roll an input.

Vendored components are yours to edit, but edit them for a **repo-wide** reason — an
untranslated string, a missing aria wiring — and leave a comment saying what upstream
did and why you changed it. Per-screen tweaks go in `className` at the call site, never
into the vendored file.

Exclude the ui folder from the dead-code scan. Vendored files legitimately export
variants and sub-components nothing imports yet.

## One token vocabulary

The component library and your own screens read from **one** set of names. Two palettes
in one app — the library's `primary` beside a bespoke `ink` — is the same colour twice
and they drift.

When adopting a library into an existing design, map the design's values onto the
library's semantic names rather than teaching the library your names. Keep a token of
your own only where the library has no equivalent.

Watch for collisions: a design's `accent` is usually a brand colour, while shadcn's
`accent` is a subtle hover surface. Same word, opposite jobs — rename yours.

## No pixels

**Never put a px value in a className.** `text-[13px]`, `rounded-[10px]`,
`max-w-[520px]` and `h-[44px]` all ignore the user's font size and are invisible to the
scale everything else is on.

- **First** use the framework's scale: `text-sm`, `rounded-lg`, `p-4`, `max-w-lg`,
  `size-11`. Tailwind's spacing and type steps are already rem.
- **If the design genuinely needs a step the scale lacks**, add it to the theme as a
  named token in rem — once, centrally — and use it everywhere. Do not inline it.
- **Only then** an arbitrary value, and in `rem` / `ch` / `em` / `dvh`: `max-w-[62ch]`.

Hairlines are the exception: `border`, and `0 0 0 1px` inside a shadow, mean one device
pixel and should stay that way.

A design handed over in px is a set of measurements, not a mandate to write them down.
Convert: 16px is `1rem`, so a 13px step is `0.8125rem`. Snap to the existing scale when
the difference is a pixel or two, and add a token when it is not.

Full-height shells use `dvh` / `svh`, never `100vh` (`mobile-a11y` says why).

## Red flags

| You catch yourself | What it means | Do instead |
|---|---|---|
| `className="… text-[13px] …"` | A pixel in a class | Scale step, or a rem token in the theme. |
| Hand-writing `role="switch"` / a focus trap | Rebuilding a primitive | Take it from the library. |
| Copying a component "to tweak one thing" | Duplication | One component, one more prop or `className`. |
| Editing a vendored file for one screen | Drift you cannot upgrade past | `className` at the call site. |
| `bg-ink` next to `bg-primary` | Two palettes | Map onto one vocabulary. |
| A bespoke colour for "just this" | A token you did not add | Add it to the theme, or reuse one. |
| Library added, screens still hand-rolled | Half an adoption | Convert the screens in the same change. |
