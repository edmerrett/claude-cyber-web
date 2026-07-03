# Claude Cyber Community — Events

A single-page, zero-build events site for the Claude Cyber Community: talks,
workshops, hackathons and meet-ups for security people building with Claude and
AI agents. Hosted free on Cloudflare Pages.

It is intentionally a **plain static site** — one `index.html` with inline CSS
and JS, no framework, no build step, no dependencies. Editing means editing HTML.

---

## Structure

```
public/                    # ← the entire deployed site (Cloudflare output dir)
  index.html               # the whole site: markup, styles, timeline logic, EVENTS data
  _headers                 # forces text/plain on /.well-known/*
  .well-known/security.txt # disclosure contact + a base64 easter-egg nudge
.claude/skills/add-event/  # skill: add an event from a Luma URL
README.md · CLAUDE.md · .gitignore
```

Everything outside `public/` (this README, `CLAUDE.md`, the skill) is repo-only
and is **not** served — the build output directory is `public/`.

---

## Editing events

All events live in one array near the bottom of `public/index.html`:

```js
var EVENTS = [
  {slug:'claude-cyber-meetup-sep26', title:'London | Claude Cyber Meetup',
   date:'2026-09-03T18:00:00Z', type:'Meet-up', loc:'London, UK', country:'GB',
   blurb:'…', luma:'https://luma.com/claude-1gof'}
];
```

| Field | Required | Notes |
| --- | --- | --- |
| `slug` | yes | unique id, kebab-case |
| `title` | yes | shown as the card heading |
| `date` | yes | ISO 8601 **UTC** (`…Z`); upcoming vs. past is derived from it |
| `type` | yes | `Talk` · `Workshop` · `Hackathon` · `Meet-up` |
| `loc` | yes | e.g. `London, UK` or `Online` |
| `country` | yes | ISO-3166 alpha-2 key in `COUNTRIES`, or `ONLINE` (shows in every country view) |
| `blurb` | yes | one or two sentences |
| `luma` | upcoming | RSVP link; used for the "Register on Luma" button |
| `pics` / `blog` / `rec` | past | optional links shown on past-event cards |

Cards sort newest-first and split upcoming/past automatically. To support a new
country, add a row to the `COUNTRIES` map (code → name + flag emoji) and tag
events with that code.

**Fastest path:** in Claude Code, run the `add-event` skill with a Luma URL —
see [`.claude/skills/add-event/SKILL.md`](.claude/skills/add-event/SKILL.md).

---

## Local preview

No build. Serve the folder with any static server:

```bash
npx serve public
# or
python3 -m http.server -d public 8000
```

Then open the printed URL. `/.well-known/security.txt` is served correctly by
these servers and by Cloudflare in production.

---

## Deployment

Hosted on **Cloudflare Pages**, **Git-connected** to this repo:

- Production branch: `main`
- Build command: *(none)*
- Build output directory: `public`

Every push to `main` triggers an automatic deploy — no CI workflow in this repo.
Pushes to other branches produce preview deployments.

Live: <https://claude-cyber-community.pages.dev>

---

## Easter egg

`public/.well-known/security.txt` carries a base64 nudge — part of a light
capture-the-flag trail for the security crowd. Leave it in.

*Community-run. Part of the official Claude Community.*
