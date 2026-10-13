# Blueprint — JR Names Meaning ("Name Explorer")

Single source of truth for this project. Referenced before every change, per `GEMINI.md`.

## Overview & capabilities

A standalone, framework-less web app that explores any given name: meanings, statistics,
birth-date facts, music & entertainment charts, historical facts, once-in-a-lifetime
events, and live life statistics — plus multi-profile management, photo uploads,
settings-driven section configuration, dark mode, and image/PDF export.

No build step: everything ships in `index.html`. Data lives in the browser via
`use-fireproof` (IndexedDB database `baby-names-db`).

## Structure

- `index.html` — the entire app: head config, CDN import map, Babel-transpiled
  React module (`script type="text/babel" data-type="module"`), and root render.
- `main.js` / `style.css` — starter placeholders, intentionally unused.
- `plans/`, `API-Docs/` — migration plan and Puter/Pollinations API references.
- `.idx/dev.nix` — Firebase Studio environment (python http.server preview).

### Dependencies (all via CDN, deliberate for standalone HTML export)

- React 19 + react-dom 19 via esm.sh import map
- `@babel/standalone` (in-browser JSX transpile)
- Tailwind CSS v4 browser build (`@tailwindcss/browser`) with **class-based dark
  mode** enabled via `<style type="text/tailwindcss">@custom-variant dark ...</style>`
- `use-fireproof` — local database (`useDocument`, `useLiveQuery`)
- `react-icons/fa` — icon set
- `puter.js` — primary AI (no API key); Pollinations REST API — optional fallback,
  key stored in `localStorage` only

### Data model (fireproof docs)

- `{ type: "profile", name, birthDate, _files: { photo }, createdAt, _id }`
- `{ type: "settings", ...sections }` — all section toggles/counts, `darkMode`,
  `selectedCountry`. **Never contains the API key** (localStorage only).

### AI flow

`callAI(prompt, schema)` → Puter `puter.ai.chat` (system prompt embeds JSON schema)
→ on failure, Pollinations if a key is configured. `extractJSON` strips code fences
and validates the payload. Six generators (meanings, statistics, birthFacts, music,
facts, lifetimeEvents) run **concurrently** per profile load, each with its own
loading flag (`loadingSections`) and its own error card + retry button.

`loadGenerationRef` invalidates stale loads: switching profiles mid-flight discards
the previous profile's in-flight AI results instead of overwriting the new profile.

### UI conventions

- Cards: `rounded-2xl shadow-2xl p-6` on white/gray-800, gradient page background.
- Collapsible headers use real `<button>` elements (`SectionHeader` /
  `SubsectionHeader` components) with `aria-expanded` and `aria-hidden` chevrons.
- Life statistics update live via a 1s interval.
- Version label is derived from `versionHistory[0].version` (changelog lives in the
  header modal data).

## Change log

- **Review pass (current)** — security: removed a committed local tool token from
  `.vscode/settings.json` and untracked it, deleted `firebase-debug.log`, hardened
  `.gitignore`, stopped writing the Pollinations key into the database.
  Correctness: escaped user query before `new RegExp`; fixed blob-URL leaks in both
  photo effects; `createProfile` can no longer stick on "Creating..."; stored
  settings merge over defaults so old/partial docs can't crash the app; strict
  YouTube ID validation; top-names numbering no longer breaks on duplicates.
  Robustness: per-section loading state, concurrent section loading, load-generation
  guard against races. A11y: all collapsible headers are keyboard-operable buttons.
  Housekeeping: removed dead CSS/imports/state, created this blueprint.
- **v1.0.7** — accurate chart data integration.
- **v1.0.6** — live statistics and YouTube integration.
- **v1.0.5** — music/facts/events improvements.
- **v1.0.4** — UI reorganization, live stats, saved profiles auto-minimize.
- **v1.0.3** — event markdown rendering, dark mode, scroll-to-top.
- **v1.0.2** — chart songs by exact DOB, fact/event counts and topics, celebrities.
- **v1.0.1** — header layout responsive fixes.
- **v1.0.0** — initial launch.

## Verification

No build/lint/test scripts exist in the project. During the review pass the app
script was extracted and validated with Babel (syntax) and ESLint `no-undef`
(0 errors, unchanged warning count), plus a jsdom harness that boots the real
component with stubbed CDN modules and exercises profile loading, AI rendering,
collapse/expand, dark mode, settings persistence, search filtering, error/retry,
and the profile-switch race (50 checks, all passing).

## Current plan

No pending work.
