# CLAUDE.md — Claude Cyber Community events site

Guidance for Claude Code working in this repo.

## What this is

A **plain static site**, deliberately. One `public/index.html` holds all markup,
inline CSS, and inline JS. No framework, no bundler, no build step, no
dependencies, no `package.json`. Do not add any of those. If a change tempts you
toward a build tool, it's the wrong change — edit the HTML directly.

## Layout

- `public/` is the deployed site and the Cloudflare build output directory.
  Anything the public should not see must live **outside** `public/`.
- `public/index.html` — the whole site. The `EVENTS` array (near the bottom,
  inside the single `<script>`) is the data model; `COUNTRIES` maps country
  codes to name + flag.
- `public/_headers` — Cloudflare Pages header rules. Keep the `/.well-known/*`
  → `text/plain` rule.
- `public/.well-known/security.txt` — disclosure file with a base64 easter-egg
  nudge. **Do not remove or "clean up" the egg.**

## Adding / editing events

Prefer the `add-event` skill (`.claude/skills/add-event/SKILL.md`) for adding
from a Luma page. Manual edits: append an object to `EVENTS` following the schema
in the README. Rules that matter:

- `date` is ISO 8601 **UTC** (trailing `Z`). Upcoming vs. past is computed from
  it against `new Date()` — never hand-set a "past" flag.
- `country` must be a key in `COUNTRIES`, or `ONLINE`. If it's a new country,
  add a `COUNTRIES` row first (code → `{name, flag}`) or the selector won't list it.
- Upcoming events need `luma`. Past events may add `pics` / `blog` / `rec`.
- Keep entries valid JS object literals — a trailing syntax error silently breaks
  the timeline (it's parsed by the browser, not a build step).

## House rules

- British English. No em dashes. No filler.
- Match the existing terse, dependency-free style in `index.html`: single-letter
  helpers (`ic`, `fmt`), compact literals. Don't reformat the whole file.
- Comments only where the code can't explain itself; one line max.

## Verifying a change

No build to catch errors, so check by eye and by serving:

```bash
npx serve public   # then load the page and confirm the timeline renders
```

A blank timeline almost always means a JS syntax error in the `EVENTS` array.

## Deployment

Cloudflare Pages, Git-connected, output dir `public`, production branch `main`.
Pushing `main` auto-deploys; there is no CI workflow here by design. Do not add
one unless the hosting model changes.

Deploys run under a **personal** account — keep employer branding, analytics, and
trackers out of the site.
