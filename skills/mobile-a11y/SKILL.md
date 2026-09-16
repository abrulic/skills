---
name: mobile-a11y
description: >
  Build UI mobile-first and accessible: mobile is the default canvas, every job works
  with a thumb and with a keyboard. Use when writing or changing JSX, CSS, layout,
  forms, navigation, or PWA screens, or when the user says a11y, accessibility,
  mobile-first, mobile, phone, touch, or /mobile-a11y. Do not use for browser-test
  craft (ui-verify) or TypeScript taste (code-quality-standards).
---

# Mobile a11y

These apps are mobile first (often a PWA). A layout that only works on desktop with a
mouse is not the product.

How the screen is *tested* (jobs, hit-target, widths): `ui-verify`. This skill is how
you **build** the screen so a thumb and a keyboard can finish the job. Which primitive
to build it from, and why nothing is sized in px: `ui-components`. The words in labels
and `aria-label`: `i18n-copy` when the repo has i18n.

Don't catalogue widgets. The next failure is not in a list of roles.

## Two users, every job

A change is not done until **both** can complete the job you just added or touched:

1. **Thumb on mobile** (one hand, coarse pointer).
2. **Keyboard** (Tab / Shift+Tab / Enter / Escape / arrows where the control is a
   composite), with a name a screen reader can speak.

If either path cannot finish the job, stop. Do not ship a mouse-only or desktop-only
path and "add mobile later".

## Mobile is the canvas

Write the layout for mobile first. `sm:` / `md:` / `lg:` are enhancements, not the
starting point.

- Default type, spacing, and stacking assume mobile. Wider screens get extra columns
  or side-by-side, not the reverse.
- The primary CTA and the next field sit in the thumb zone: reachable without the
  user covering the thing they are reading. Do not park the only action under the
  home indicator, a system bar, or the app's own bottom nav. Use the safe-area
  insets the platform already gives you (`env(safe-area-inset-*)`).
- Prefer `dvh` / `svh` over `100vh` for full-height shells. Mobile browser chrome
  steals `vh`.
- A job must not require hover. Hover may enrich; it may not be the only way.

Print one line when you touch UI: `canvas: mobile, keyboard ok`.

## Named, focusable, large enough

For every control the job needs:

- It is a real control (`button`, `a`, `input`, `select`, `textarea`) or it has an
  explicit role **and** is in the tab order. `div onClick` is not a control.
- It has an **accessible name**: the visible label, or `aria-label` when the visible
  thing is only an icon. The name is the words the user would use.
- Focus is visible. Do not `outline-none` unless you replace it with a focus ring
  that still contrasts.
- The box is thumb-sized (at least ~44×44 CSS px of hit area, padding counts). Tiny
  icon hits fail on mobile even if they look fine with a cursor.

While a sub-job is open (filter, confirm, menu): that sub-job's controls are in the
tab order and Escape (or the visible dismiss) leaves it. After it closes, focus
returns to the control that opened it. The page behind is not a second tab trap.

## Contrast and motion

Text and the focused control meet contrast against the background they actually sit
on (including overlays). Do not use color as the only signal (error, selected, on).

Honor `prefers-reduced-motion` for anything that moves. Do not make a job depend on
an animation completing.

Don't hand-check what the project's a11y linter already flags. Spend attention on
keyboard path, names, thumb reach, and hover-only jobs.

## Procedure

1. Build the screen for mobile. Stack first; enhance at breakpoints.
2. Tab through the job. Every stop has a name and a focus ring. Escape leaves a
   sub-job.
3. Hit each control as a thumb: the box is large enough and not under system/app
   chrome.
4. Then `ui-verify` on mobile (and desktop only if this layout actually changes
   there).

## Red flags

| You catch yourself | What it means | Do instead |
|---|---|---|
| First media query is `min-width: 1024px` | Desktop-first | Mobile layout is the default; enhance up. |
| `div onClick` / `span onClick` | Not a control | `button` / `a`, or role + tab index + keyboard. |
| `outline-none` and nothing else | Invisible focus | Visible ring that contrasts. |
| Icon-only control, no label | No accessible name | `aria-label` (or visible text). |
| Job exists only in `hover:` | Mobile cannot do it | Same job on tap / click / keyboard. |
| CTA flush with the screen bottom | Home indicator / nav covers it | Safe-area padding; sit it above chrome. |
| Only verified on desktop | You did not test the product | Mobile canvas, then `ui-verify`. |
