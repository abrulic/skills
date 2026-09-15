---
name: i18n-copy
description: >
  Put user-facing copy through the repo's i18n layer. Use when writing JSX, toasts,
  emails, empty states, labels, aria-label, placeholders, or any string a person
  reads, or when the user says i18n, translation, locale, or /i18n-copy. Skip if the
  repo has no i18n (no i18next, no locales/). Do not use for log lines, identifiers,
  or CSS.
---

# i18n copy

If a person reads it, it is a translation key, not a string literal in the component.

Skip this skill when the repo has no i18n setup (no `i18next`, no `locales/` /
`resources/locales`). Do not add an i18n library to a package that never shows UI.

How labels must exist for assistive tech: `mobile-a11y`. This skill only owns that
those words come from i18n.

## Gate

1. Find how this repo translates (usually `useTranslation` / `t()`, locale JSON under
   `resources/locales` or similar). Match that. Do not invent a second system.
2. Find the fallback language and the locale files already in the repo.

## What must go through `t()`

Visible UI, `placeholder`, `title`, `alt`, `aria-label`, toast/flash text, email
subject and body the user sees, empty/error copy, button names.

Do **not** translate: route paths, CSS, log/error codes for developers, enum values
stored in the DB, test IDs.

## Keys, not sentences in source

- Add the key to **every locale file this repo already has**. Do not add a new
  language. Do not leave one locale missing the key (fallback is a safety net, not a
  license to skip).
- Key names describe the copy (`members.empty`, `auth.login.submit`), not the English
  sentence.
- Reuse an existing key that is the same words and the same meaning. Do not mint
  `save2` next to `common.save`.
- Interpolation for dynamic bits: `t("members.count", { count })`, not string concat
  around a literal.

Hardcoded copy in JSX/TS is a miss even if the app currently has one language. The
second locale (or the first real translator) is when it breaks.

## Procedure

1. Confirm i18n exists. If not, stop.
2. For each new user-facing string: search locale files for the same meaning.
3. Reuse or add the key in every existing locale, then `t("the.key")` at the call site.
4. Accessible name = the same `t()` as the visible words (`mobile-a11y`).

## Red flags

| You catch yourself | What it means | Do instead |
|---|---|---|
| `"Save"` / `"Nema članova"` in JSX | Hardcoded copy | `t("…")` + key in every locale. |
| Key added only in `en/` | Other locales drift | Every locale file the repo already has. |
| `aria-label="Close"` next to `t("common.close")` | Two sources | Same `t()` for both. |
| New `i18next` setup in a CLI package | Wrong repo | Stop. This skill does not apply. |
| Concat `"Hello " + name` | Untranslatable | `t("greeting", { name })`. |
