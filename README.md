# Denver · Underoath Weekend

A mobile-first static site for the Denver trip, Fri Nov 20 – Sun Nov 22, 2026.
One shared link, no backend, no per-user state — everyone sees the same data.

Three tabs, routed by hash: `#plan`, `#map`, `#places`.

## Editing the trip

Everything lives in one block at the top of the `<script>` in `index.html`,
marked `TRIP DATA`. Four arrays:

| Array    | What it holds |
|----------|---------------|
| `HOME`   | The hotel. Used as the map's home marker and the fallback origin for distances. |
| `CREW`   | Name + one-line subtitle per person. |
| `PLACES` | Food, bars, coffee, the venue. `star: true` gives a gold ring on the map and shows in the ★ filter. |
| `DAYS`   | Day-by-day timeline items, in chronological order per day. |

Nothing below that block needs to change to update the trip.

**Place fields:** `id`, `name`, `cat` (`food` \| `bar` \| `coffee` \| `venue` \| `sight`),
`star`, `addr`, `lat`, `lng`, `note` (the one-line "why"), `url`.

**Timeline item fields:** `t` (24h `"HH:MM"`, always Mountain Time), `title`,
and optionally `disp` (override the displayed time string), `approx`, `tbd`,
`tag` (`flight` \| `show` \| `food` \| `open`), `d` (detail line),
`place` (an id from `PLACES`/`HOME` — adds Directions + On map),
`flight` (an ICAO ident like `UAL5520` — adds a FlightAware link).

All times are Mountain Time. November 2026 is MST, UTC−7, set via `OFFSET`.

### Map tiles need a CARTO API key

CARTO requires a free API key for its basemap tiles as of September 2026 —
without one the map shows CARTO's own "API KEY REQUIRED" placeholder instead
of the city. Get one (no account needed, 5M tile loads/month free for
non-commercial use) at <https://carto.com/basemaps/apikey>, then set
`CARTO_KEY` right under `OFFSET` near the top of the script:

```js
const CARTO_KEY = "your-key-here";
```

Left blank, the app shows a "Map needs a free API key" notice instead of
CARTO's broken tiles, so it fails visibly rather than silently.

## Local preview

```sh
python3 -m http.server 8000
```

Then open <http://localhost:8000>. Geolocation works on `localhost` — browsers
treat it as a secure context — so "Where am I" is testable locally. It will
*not* work if you open `index.html` as a `file://` URL.

## Deploy

Repo: <https://github.com/jstnheo/underoath-denver> · hosted on Vercel.

### Push

```sh
git remote add origin git@github.com:jstnheo/underoath-denver.git
git push -u origin main
```

(Use `https://github.com/jstnheo/underoath-denver.git` instead if you're not
set up with SSH keys.)

### Import into Vercel

1. <https://vercel.com/new> → Import Git Repository → `underoath-denver`.
   Authorize the GitHub app for the repo if Vercel prompts.
2. **Framework Preset: Other.** Leave Build Command, Output Directory and
   Install Command empty — there's no build step, Vercel serves the repo root.
3. Deploy. Live at `https://underoath-denver.vercel.app` (Project → Settings →
   Domains to change it).

After the import, every push to `main` is a production deploy; pushes to other
branches get preview URLs. Production is publicly reachable on a Hobby plan,
so the one shared link works for everyone without a login.

`vercel.json` sends `Cache-Control: public, max-age=0, must-revalidate` on the
HTML. That matters here: the trip data is baked into `index.html`, so without
it a phone can keep serving a stale plan after you've updated something. With
it, a refresh always picks up the current version.

### Checking it on your phone

Geolocation needs HTTPS, which the `.vercel.app` URL gives you. Open the link
on the phone, go to **Map → 📍 Where am I**, and allow location. From outside
Denver you should see "You're N mi from Denver" rather than a walk time — that
branch is deliberate, not a bug.

## Dependencies

Loaded from cdnjs at runtime, pinned:

- Leaflet 1.9.4 (CSS + JS)
- Inter + Cinzel via Google Fonts
- CARTO dark basemap tiles — needs a free API key, see above

No build step, no package manager, no `node_modules`.
