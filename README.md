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
| `DAYS`   | Day-by-day timeline items. |

Nothing below that block needs to change to update the trip.

**Place fields:** `id`, `name`, `cat` (`food` \| `bar` \| `coffee` \| `venue` \| `sight`),
`star`, `addr`, `lat`, `lng`, `note` (the one-line "why"), `url`.

**Timeline item fields:** `t` (24h `"HH:MM"`, always Mountain Time), `title`,
and optionally `disp` (override the displayed time string), `approx`, `tbd`,
`tag` (`flight` \| `show` \| `food` \| `open`), `d` (detail line),
`place` (an id from `PLACES`/`HOME` — adds Directions + On map),
`flight` (an ICAO ident like `UAL5520` — adds a FlightAware link).

All times are Mountain Time. November 2026 is MST, UTC−7, set via `OFFSET`.

## Local preview

```sh
python3 -m http.server 8000
```

Then open <http://localhost:8000>. Geolocation works on `localhost` — browsers
treat it as a secure context — so "Where am I" is testable locally. It will
*not* work if you open `index.html` as a `file://` URL.

## Deploy

### Option A — Netlify (recommended)

Fastest path to a working HTTPS link, and no CLI to install.

**First deploy, by hand:**

1. Go to <https://app.netlify.com/drop>.
2. Drag the `underoath` folder onto the page.
3. You get a live HTTPS URL in a few seconds. Sign in to keep it.
4. Site configuration → Change site name → something shareable,
   e.g. `underoath-denver` → `https://underoath-denver.netlify.app`.

**Then wire it to git so edits auto-deploy:**

1. Push this repo to GitHub (see below).
2. In Netlify: Site configuration → Build & deploy → Link repository.
3. Build command: *leave empty*. Publish directory: `.`
   (`netlify.toml` already sets this.)

After that, every `git push` redeploys. `netlify.toml` also sends
`Cache-Control: must-revalidate` on the HTML so nobody's phone shows a stale
plan — which matters, since the trip data is baked into the page.

### Option B — GitHub Pages

Free and tied to the repo, but slower to update (~1 min per deploy) and the
URL is longer.

```sh
git remote add origin git@github.com:<your-username>/underoath.git
git branch -M main
git push -u origin main
```

Then: repo → Settings → Pages → Source: **Deploy from a branch** →
Branch `main`, folder `/ (root)` → Save.

Live at `https://<your-username>.github.io/underoath/` within a minute or two.

Note: Pages ignores `netlify.toml`, so you don't get the cache headers. GitHub
serves HTML with a 10-minute cache. Not a problem for planning, but if you
change something mid-trip, tell people to pull-to-refresh.

### Pushing to GitHub

No `gh` CLI on this machine, so create the repo in the browser at
<https://github.com/new> — name it `underoath`, keep it empty (no README,
no .gitignore), then:

```sh
git remote add origin git@github.com:<your-username>/underoath.git
git branch -M main
git push -u origin main
```

Make it **public** if you use GitHub Pages on a free account. Netlify deploys
private repos fine.

## Dependencies

Loaded from cdnjs at runtime, pinned:

- Leaflet 1.9.4 (CSS + JS)
- Inter via Google Fonts
- CARTO dark basemap tiles

No build step, no package manager, no `node_modules`.
