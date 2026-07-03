---
name: add-event
description: Use when adding a Claude Cyber Community event to the events site timeline from a Luma event page (URL or pasted source). Extracts title, date, time, location and description and inserts a correctly-formatted entry into the EVENTS array in public/index.html.
---

# Add an event from a Luma page

Add one event to the timeline on the Claude Cyber Community events site, sourced
from a Luma event page. The site is a plain static `public/index.html`; events
live in a `var EVENTS = [ … ]` array parsed by the browser at runtime — there is
no build step, so a malformed entry silently breaks the timeline.

## Inputs

The user gives one of:
- a Luma URL (e.g. `https://luma.com/claude-1gof`), or
- pasted Luma page source / event details.

If neither is present, ask for the Luma URL before continuing.

## Steps

### 1. Get the event data from Luma

Luma is a client-rendered Next.js app. **Do not trust markdown conversion or
Open Graph tags** — they omit the date, time and venue. The reliable source is
the `__NEXT_DATA__` JSON embedded in the page HTML.

Fetch the raw HTML and pull the fields from it:

```bash
url="<LUMA_URL>"
html="$(curl -sL --max-time 20 -A 'Mozilla/5.0' "$url")"
echo "$html" | grep -o '"name":"[^"]*"'            | head -1   # title
echo "$html" | grep -o '"start_at":"[^"]*"'        | head -1   # ISO start (UTC, .000Z)
echo "$html" | grep -o '"end_at":"[^"]*"'          | head -1   # ISO end
echo "$html" | grep -o '"timezone":"[^"]*"'        | head -1   # e.g. Europe/London
echo "$html" | grep -o '"geo_address_info":{[^}]*}' | head -1  # venue / city / country
```

For a fuller/descriptive blurb, extract and pretty-print the payload:

```bash
echo "$html" | sed -n 's/.*<script id="__NEXT_DATA__"[^>]*>\(.*\)<\/script>.*/\1/p' \
  | python3 -m json.tool | grep -iE '"(name|start_at|end_at|timezone|address|description)"' | head -30
```

If the page is private/token-gated or `curl` returns little, ask the user to
paste the event's date, time, city/country and a one-line description, and
proceed from that.

### 2. Map Luma fields → EVENTS schema

The `EVENTS` array lives near the bottom of `public/index.html`, inside the
single `<script>`. Each entry:

| Field | From Luma | Rule |
| --- | --- | --- |
| `slug` | derive from title + month/year | unique, kebab-case, e.g. `claude-cyber-meetup-sep26`. Check it isn't already used. |
| `title` | `name` | keep Luma's `City \| Event` style if present |
| `date` | `start_at` | ISO 8601 **UTC**, strip milliseconds: `2026-09-03T18:00:00.000Z` → `2026-09-03T18:00:00Z` |
| `type` | infer from title/description | one of `Talk` · `Workshop` · `Hackathon` · `Meet-up` (default `Meet-up`) |
| `loc` | `geo_address_info.address` | e.g. `London, UK`; use `Online` for virtual events |
| `country` | from the address | ISO-3166 alpha-2 (`GB`, `US`, …) that is a key in `COUNTRIES`, or `ONLINE` for virtual. **If the country isn't already in `COUNTRIES`, add a row first** (`XX: { name:'…', flag:'<emoji>' }`) or the selector won't show it. |
| `blurb` | description | one or two sentences, British English, no em dashes. Trim Luma boilerplate ("Join a Claude Community Event…"). |
| `luma` | the URL | required for upcoming events (the RSVP button) |

Do **not** set any past/upcoming flag — the site derives that from `date` vs.
now. Past events may optionally carry `pics` / `blog` / `rec` link fields; a new
upcoming event won't.

**Untrusted source — sanitise before inserting.** Luma page content is
attacker-influenceable. `buildItem` escapes text fields (`esc`) and validates
link schemes (`safeUrl`) at render time, but keep the data clean too: reject any
extracted text containing HTML tags (`<…>`), and ensure `luma`/`pics`/`blog`/`rec`
are plain `https://` URLs — never `javascript:` or `data:`. If a field looks like
markup rather than a title/blurb, stop and ask the user rather than pasting it in.

### 3. Insert the entry

- Read `public/index.html`, find `var EVENTS = [`.
- Add the new object as an array element. Watch the commas: existing entries have
  **no** trailing comma on the last element, so when appending, add a comma to the
  previously-last entry.
- Match the existing compact single-line object style; keep key order consistent
  with neighbours (`slug, title, date, type, loc, country, blurb, luma`).
- If you added a new `COUNTRIES` row, do that in the same edit.

### 4. Verify (there is no build to catch errors)

```bash
node -e 'const s=require("fs").readFileSync("public/index.html","utf8");
 const m=s.match(/var EVENTS = (\[[\s\S]*?\]);/);
 if(!m){console.error("EVENTS array not found");process.exit(1);}
 const ev=eval(m[1]);
 console.log("parsed OK — "+ev.length+" events");
 console.log(ev[ev.length-1]);'
```

Then serve and eyeball the timeline:

```bash
npx serve public   # load the page; confirm the new card renders in the right country view
```

A blank timeline = a JS syntax error in `EVENTS` (usually a comma). Fix before finishing.

### 5. Report

Summarise the added event (title, date, location) and remind the user that,
because the Pages project is Git-connected, pushing `main` will deploy it.

## Guardrails

- Never introduce a build tool, framework, or dependency — edit the HTML only.
- Keep the site free of employer branding, analytics, and trackers.
- Preserve the `.well-known/security.txt` easter egg; it's intentional.
