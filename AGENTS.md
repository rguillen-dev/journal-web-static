# AGENTS.md — journal-web-static
# Public marketing / App Store compliance site for Journal

GitHub: `rguillen-dev/journal-web-static` (the older `Mr-Nizzle` org is outdated — do not reference it).

> Canonical, cross-tool instructions for this repo. **Claude Code and GitHub
> Copilot both read this file** — it is the single source of truth.

## Session handoff protocol — read first, update last

Multiple agents work this repo (Claude Code, GitHub Copilot); they do **not**
share session memory. The baton is passed through files, not sessions:

1. **Start:** read `../knowledge/PROJECT_STATE.md` (current state, decisions,
   open blockers) before planning.
2. **During:** small commits, conventional messages —
   `feat|fix|test|refactor|chore|docs (scope): description [auto]`.
3. **End (if state changed):** update `../knowledge/PROJECT_STATE.md` (what
   shipped · what's in progress · the next step) and commit it. That update is
   what lets the next agent — in either tool — continue where you left off.

Git history + branch state are the durable trail. Never rely on a tool's own
session memory for cross-tool continuity.

## What this repo is

Hand-written static HTML/CSS/JS. **No framework, no build step, no package
manager, no npm/node dependency.** What's in the file is what ships — there is
no compilation, bundling, or templating layer between the source and the
browser.

Current pages: `index.html` (hero/marketing landing, family-facing),
`support.html`, `privacy.html`. Shared assets: `styles.css` (one shared
stylesheet, all pages), `script.js` (one shared script — fade-in-on-scroll via
`IntersectionObserver`, nothing else), `assets/icon.png`, `CNAME`.

There is no `.mise.toml` and no per-page or per-component scaffolding —
add a new page by copying the structure of an existing one (`<head>` block,
`.page` body, shared `.footer`), not by inventing a new layout system.

## Deployment

GitHub Pages, deployed from the repo root on push (no build step, no Actions
workflow needed for a plain static site). Custom domain via `CNAME` →
`journal.ricardoguillen.dev`. There is no staging environment and no branch
mapping — what's on the default branch is what's live. Treat every commit to
the default branch as a production deploy; there is no `develop` →
`release/*` → `main` promotion flow here unlike the other three repos.

## Content Security Policy — per page, hand-set

Every page carries its own `<meta http-equiv="Content-Security-Policy">` tag
in `<head>` — there is no shared CSP config or server-side header injection.
When adding a page, copy the CSP meta tag from the closest existing page and
adjust deliberately; don't drop it.

Current baseline, present on every page today:
```
default-src 'self'; style-src 'self'; img-src 'self'; script-src 'self' | 'none'; form-action 'none';
```
- `index.html` allows `script-src 'self'` (needed for `script.js`'s
  fade-in-on-scroll). `support.html` and `privacy.html` set `script-src
  'none'` — they don't load `script.js`.
- **`form-action 'none'` is set on every page today.** This repo has no forms
  yet. The first page that adds a real `<form>` (e.g. a future
  demo/application-request page — see
  `../knowledge/handoffs/apple-3.2-front-door.md` §3) must scope
  `form-action` deliberately for that page only (to the actual submission
  target — same-origin API route, or the specific third-party form host if a
  hosted embed is chosen) rather than loosening it site-wide or leaving it at
  `'none'` and silently breaking the form.
- Never widen `default-src`, `style-src`, or `img-src` beyond `'self'`
  without a specific reason (a third-party embed, a hosted font, etc.) — this
  site has no external dependencies today and that's deliberate.

## i18n — Spanish first, English second

`index.html` is Spanish-only — it's the App Store / hero landing page and
its primary audience is parent-facing.

`support.html` and `privacy.html` follow the bilingual pattern: Spanish
content first, English second, both on the same page (not separate `/en/`
routes). Structure:
```html
<div class="lang-block">
  <p class="lang-label">Español</p>
  ...
</div>
<div class="lang-block">
  <p class="lang-label">English</p>
  ...
</div>
```
Any new page whose audience includes non-family readers (schools, App Review)
should follow the bilingual `support.html`/`privacy.html` pattern, not the
Spanish-only `index.html` pattern. Match this exactly — don't introduce a
third i18n approach (no query params, no separate English files, no JS-driven
language switcher).

## Design tokens

Source of truth: `../knowledge/design/journal-tokens-v1.json` (shared with
`agenda-web` and `agenda-ios` — platform-agnostic).

This site does not consume the token file programmatically (no build step to
pipe it through) — `styles.css`'s `:root` custom properties are a hand-copied
subset of it, chosen for this site's dark/teal marketing aesthetic rather than
the app's full skin system:
- `--color-bg: #041A1A`, `--color-bg-alt: #072B2B` — from
  `color.brand.teal.900` / the `calma` skin's dark surface family.
- `--color-accent` / `--color-glow: #2A9C9B` — `color.brand.teal.500`
  (`DEFAULT`), the same brand teal used everywhere else in Journal.
- Font stack is system (`-apple-system`/`SF Pro`), **not** the app's DM
  Sans/DM Serif Display pair from `typography.family` — this site predates
  that typography decision and hasn't been reconciled to it. Don't
  silently "fix" this by swapping fonts in an unrelated change; if it's worth
  reconciling, that's its own task.

When a new page needs a color, check the token file first and prefer an exact
match over eyeballing a new hex value. If you introduce a value the token
file doesn't have, that's a signal to flag it rather than invent a one-off.

## Locked decisions

- No framework, no build step, no bundler. Do not introduce React, a static
  site generator, npm, or a CSS preprocessor to "modernize" this repo — its
  entire value is being trivially auditable, hand-editable HTML.
- One shared `styles.css`, one shared `script.js` — no per-page stylesheets
  or scripts unless a page has a genuinely page-specific behavior (and even
  then, prefer adding a scoped class/block to the shared files first).
- CSP stays maximally restrictive per page; widen only the specific directive
  a specific new feature needs, only on the page that needs it.
- Spanish-first bilingual pattern (`support.html`/`privacy.html`) for any
  page addressing schools, App Review, or a non-family audience.

## Commands

None. There is no build, lint, or test tooling in this repo — open the HTML
file directly in a browser, or serve the directory with any static file
server, to check a change.

## Pre-edit checklist

- [ ] New page copies the `<head>` CSP/meta pattern from the nearest existing
      page (`index.html` if script-loading, `support.html`/`privacy.html` if
      not)?
- [ ] `form-action` only widened on a page that actually has a form, and only
      to the real submission target?
- [ ] Colors/spacing checked against `../knowledge/design/journal-tokens-v1.json`
      before inventing a new value?
- [ ] Bilingual `.lang-block` pattern used for any non-`index.html` page
      addressing schools or App Review?
- [ ] No framework, build step, or dependency introduced?
