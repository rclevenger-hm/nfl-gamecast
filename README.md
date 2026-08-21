# NFL Live Gamecast

A self-hosted, ESPN-Gamecast-style live scoreboard. Pure static HTML/CSS/JS — no build step, no server, no API keys. All game data is fetched live in the visitor's browser from ESPN's public scoreboard API, so the site updates itself every day with whatever games are on the slate.

## Pages

- **`index.html`** — scoreboard of all NFL games. Defaults to the current week's slate (ESPN decides what's current, so it's always fresh). Cards show live score, quarter/clock, down & distance, possession, and last play, refreshing every 20 seconds while any game is live. A date picker jumps to any past or future slate. Click any game to open its Gamecast.
- **`gamecast.html`** — full Gamecast for one game: score header with live clock and possession, quarter-by-quarter linescore, down & distance with a field-position graphic, drive-by-drive play feed, scoring summary, box score, and player leaders. Refreshes every 15 seconds while live, stops at final. Takes `?event=<espn-event-id>`.

Both pages have a day/night toggle (☀️/🌙) that follows your system preference by default and remembers your choice.

## CI/CD (GitHub Actions)

`.github/workflows/deploy.yml` runs on every push to `main` (and manually via *Run workflow*):

1. **test** — installs Playwright, then runs `test/smoke.mjs`: both pages are loaded in headless Chromium against a mocked ESPN API and ~23 assertions check that the scoreboard cards, gamecast score header, linescore, situation, field marker, play feed, box score, leaders, tabs, and the day/night toggle all render with no JS errors.
2. **deploy** — only if tests pass, publishes `index.html` + `gamecast.html` to GitHub Pages via `actions/deploy-pages`.

## Deploy to GitHub Pages

1. Create a new repository on GitHub (e.g. `nfl-gamecast`) and push these files:

   ```bash
   cd nfl-gamecast
   git remote add origin https://github.com/rclevenger-hm/nfl-gamecast.git
   git push -u origin main
   ```

2. In the repo: **Settings → Pages → Build and deployment → Source: GitHub Actions**.
3. The workflow tests and deploys automatically; the site goes live at `https://rclevenger-hm.github.io/nfl-gamecast/`.

## Run the tests locally

```bash
cd test
npm install
npx playwright install chromium
node smoke.mjs
```

## How the data works

- Scoreboard: `https://site.api.espn.com/apis/site/v2/sports/football/nfl/scoreboard` (optionally `?dates=YYYYMMDD`)
- Game detail: `https://site.api.espn.com/apis/site/v2/sports/football/nfl/summary?event=<id>`

These are the same public, CORS-enabled endpoints that power ESPN's own site. They're unofficial and undocumented, so treat this as a personal project: they could change shape or availability without notice. The pages parse defensively and show a retry notice if the feed is unreachable.

## Tweaks

- Refresh cadence: `REFRESH_LIVE_MS` in `gamecast.html`, and the `20000` / `120000` values in `schedule()` calls in `index.html`.
- Default game for `gamecast.html` when no `?event=` is given: `DEFAULT_EVENT`.
- Other sports work too — ESPN uses the same URL pattern (e.g. swap `football/nfl` for `basketball/nba`), though a few field names differ.
