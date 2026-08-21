# Sports Live Gamecast

A self-hosted, ESPN-Gamecast-style live scoreboard for **NFL, MLB, NHL, MLS, and Premier League**. Pure static HTML/CSS/JS — no build step, no server, no API keys. All game data is fetched live in the visitor's browser from ESPN's public scoreboard API, so the site updates itself every day with whatever games are on each slate.

**Live site:** https://rclevenger-hm.github.io/sports-gamecast/

## Pages

- **`index.html`** — scoreboard of all games, with sport tabs (NFL · MLB · NHL · MLS · EPL). Defaults to each sport's current slate (ESPN decides what's current, so it's always fresh). Cards show live score, clock, situation, and last play, refreshing every 20 seconds while any game is live. A date picker jumps to any past or future slate. Click any game to open its Gamecast. Sport is also addressable directly: `?sport=mlb|nhl|mls|epl` (omit for NFL).
- **`gamecast.html`** — full Gamecast for one game: score header with live clock and possession, period/inning linescore, drive-by-drive play feed (NFL) or flat play-by-play (other sports), scoring summary, box score, and player leaders. NFL games also get down & distance and a field-position graphic. Refreshes every 15 seconds while live, stops at final. Params: `?event=<espn-event-id>`, `?sport=`, `?theme=director` (chain params with `&`). With `?sport=` and no `?event=`, it auto-picks the first live game on that slate.

Both pages have a day/night toggle (☀️/🌙) that follows your system preference by default and remembers your choice.

## Mini strip versions

Two minimized, banner-shaped versions scale to fill whatever height you give them (OBS browser source, embedded iframe, or a thin browser window — e.g. 1920×205 for the bottom fifth of a 1080p canvas). Opened standalone in a full-size window they letterbox to a centered strip automatically.

- **`ticker.html`** — ESPN BottomLine-style ticker of every game on the slate: logos, score, possession, live clock or kickoff time. Auto-scrolls (marquee) when the slate is wider than the strip; clicking a game opens its Gamecast. Params: `?sport=`, `?date=YYYYMMDD`, `?speed=90` (marquee px/sec), `?theme=light|director`, `?brand=TEXT` (left badge text), `?bg=transparent` (for overlays).
- **`gamebar.html`** — one game as a single bar: logos, score, possession, live clock, situation, a mini field-position graphic for NFL, and last play, with team-color underlines. Clicking it opens the full Gamecast. Params: `?event=<espn-event-id>`, `?sport=`, `?theme=light|director`, `?brand=TEXT`, `?bg=transparent`. With `?sport=` and no `?event=`, it follows the first live game on that slate.

`?theme=director` is a stream-overlay skin — bolt yellow + electric teal on black (Chargers broadcast vibe) with a badge block (default ⚡, override with `?brand=`).

Embed example: `<iframe src="https://rclevenger-hm.github.io/sports-gamecast/gamebar.html?theme=director" style="width:100%;height:205px;border:0"></iframe>`

## CI/CD (GitHub Actions)

`.github/workflows/deploy.yml` runs on every push to `main` (and manually via *Run workflow*):

1. **test** — installs Playwright, then runs `test/smoke.mjs`: the scoreboard and Gamecast are loaded in headless Chromium against a mocked ESPN API and ~23 assertions check that everything renders with no JS errors.
2. **deploy** — only if tests pass, publishes the four pages to GitHub Pages via `actions/deploy-pages`.

## Run the tests locally

```bash
cd test
npm install
npx playwright install chromium
node smoke.mjs
```

## How the data works

Same public, CORS-enabled endpoints that power ESPN's own site, one URL pattern for every sport:

- Scoreboard: `https://site.api.espn.com/apis/site/v2/sports/<sport>/<league>/scoreboard` (optionally `?dates=YYYYMMDD`)
- Game detail: `https://site.api.espn.com/apis/site/v2/sports/<sport>/<league>/summary?event=<id>`

Leagues used here: `football/nfl`, `baseball/mlb`, `hockey/nhl`, `soccer/usa.1` (MLS), `soccer/eng.1` (EPL). Other leagues (NBA `basketball/nba`, college `football/college-football`, any soccer league code) drop straight into the `SPORTS` map at the top of each page's script.

These endpoints are unofficial and undocumented, so treat this as a personal project: they could change shape or availability without notice. The pages parse defensively and show a retry notice if the feed is unreachable.

## Tweaks

- Refresh cadence: `REFRESH_LIVE_MS` in `gamecast.html`, and the `20000` / `120000` values in `schedule()` calls in `index.html`.
- Default game for `gamecast.html` when no `?event=` or `?sport=` is given: `DEFAULT_EVENT`.
- Add a sport/league: extend the `SPORTS` map in each page.
