# Earnings HUB (unofficial)

**Earnings HUB** is a fan-made, non-official podcast feed for personal use only. It carries the
latest official quarterly earnings conference call from each tracked company. It is **not affiliated
with, endorsed by, or sponsored by IREN, GlobalFoundries, Cipher Digital, Switch, Roblox, Vistra, CoreWeave,
NRG Energy, or any other company.** All call audio belongs to the respective companies.

Feed: `https://calearnpods.github.io/earnings-hub/feed.xml`
Site: `https://calearnpods.github.io/earnings-hub/`

The feed sets `itunes:block=Yes`; add it by URL (Apple Podcasts: Library → … → Follow a Show by URL).

## Companies

| Key | Company | IR source | Webcast platform | Quarter label |
|---|---|---|---|---|
| iren | IREN Limited (NASDAQ: IREN) | iren.com/investors/events | Nasdaq MMC | `Q1 FY27` (FY ends June 30) |
| gfs | GlobalFoundries (NASDAQ: GFS) | investors.gf.com events | Nasdaq MMC | `Q3 2026` (calendar) |
| cifr | Cipher Digital (NASDAQ: CIFR) | investors.cipherdigital.com events | Nasdaq MMC | `Q3 2026` |
| rblx | Roblox (NYSE: RBLX) | ir.roblox.com events | Q4 Events | `Q3 2026` |
| vst | Vistra Corp. (NYSE: VST) | investor.vistracorp.com events (ajax list) | Open Exchange (headless Chrome guestbook, `.venv_pw`) | `Q3 2026` |
| crwv | CoreWeave (NASDAQ: CRWV) | investors.coreweave.com events (Q4 event feed) | Q4 Events | `Q3 2026` |
| nrg | NRG Energy (NYSE: NRG) | investors.nrg.com presentations & webcasts → event detail pages | Nasdaq MMC | `Q3 2026` |
| switch | Switch, Inc. | switch.com | — | skipped: privately held, no public earnings webcast |

Rules: one episode per company (the newest completed call; a new quarter replaces that company's
previous episode). Items are ordered by call date, newest first. Episode titles are
`<Company> — <Quarter> Results`.

## Layout

```
earnings_hub/
  companies.json        # companies, IR events pages, platform, fiscal labelling
  episodes.json         # current episode metadata (latest per company)
  audio/                # encoded .m4a working copies
  assets/cover_source.png
  brands/<key>/         # per-company thumbnail generator, official logo, font
  site/                 # GitHub Pages root: feed.xml, index.html, art/, audio/
  scripts/
    mmc_client.py       # Nasdaq MMC (edge.media-server.com) guestbook unlock + HLS audio URL
    q4_client.py        # Q4 Events (events.q4inc.com) replay URL
    openexchange_client.py  # Open Exchange (<event>.open-exchange.net) replay via headless Chrome (run with .venv_pw)
    update.py           # scrape IR pages, detect new completed calls, download + encode
    build_feed.py       # latest-per-company selection, art, cover, RSS, index
    publish.sh          # push site/ to CalEarnPods/earnings-hub and verify live
    daily.sh            # update.py + publish.sh (publishes only when something changed)
```

## Daily update

```bash
cd /workspace/earnings_hub
pip install -r requirements.txt
python3 -m venv .venv_pw && .venv_pw/bin/pip install playwright requests   # Vistra (Open Exchange); uses /usr/bin/google-chrome
./scripts/daily.sh                       # full daily run
python3 scripts/update.py --dry-run      # detection only
python3 scripts/build_feed.py            # rebuild site from audio/ + episodes.json
```

`update.py` reads every webcast link on each company's official IR events page, keeps calls whose
title is a quarterly results/earnings call, picks the newest one that started more than two hours ago
and has a replay, and adds it if it is newer than the feed's episode for that company. Exit code 0 =
feed changed, 3 = nothing new. Summaries for automatically added episodes are generic; edit
`episodes.json` with figures from the company's release and re-run `build_feed.py` if desired.

Encoding: AAC 96 kbps stereo 48 kHz `.m4a` with `+faststart`. If a file would exceed GitHub's
100 MiB limit (104,857,600 bytes) it is re-encoded at 90, 80, 72, 64 … kbps. Audio is never cut.

## Thumbnails

3000×3000, cream margin 165, rounded inner panel (radius 5% of inner side), official logos only:

- IREN: navy panel, mint logo, Playfair Display, cream titles
- GlobalFoundries: orange #FF6012 panel, white logo, white Zilla Slab
- Cipher Digital: near-black #0d0e10 panel, lime nine-dot wordmark, cream Unbounded
- Roblox: white panel, black tilted wordmark, blue #335FFF Bricolage Grotesque
- Vistra: Vistra green #63A70A panel, official white wordmark (from Vistra's Q3 2026 results deck), navy #003F6E Fraunces
- CoreWeave: CoreWeave aqua #5EF5FA panel (2026 site palette), official full-colour logo (coreweave.com SVG), black #02030B Chakra Petch
- NRG Energy: NRG magenta #EC008C panel (official palette), official white logo (NRG media-library kit SVG), white Syne ExtraBold

## Publish

```bash
./scripts/publish.sh
```

Uses only `GITHUB_EARNINGS_TOKEN` (CalEarnPods PAT, `repo` scope); if unset it reads
`/home/box/.secrets/GITHUB_EARNINGS_TOKEN`. With no token it exits 2 and leaves the site in `site/`.
The token is never printed. The script confirms the token belongs to CalEarnPods, creates the public
repo if needed, force-pushes a single fresh commit, enables Pages, and waits until the live
Content-Length of `feed.xml`, `art/cover.jpg` and every audio file matches the local files.
