# The EV Hub — Phase 1 (demo)

**DEMO MODE: sample data, not live betting odds.** Every club, player, statistic and price is synthetic, made up for this demo, and generated in your browser. Nothing here is a betting recommendation.

The site has seven pages: Dashboard, EV Explorer, Player Stats, Team Stats, Fixtures, Alerts and Settings. It is one self-contained web page plus a few small files. It needs no server, no API key and no build step. It runs on GitHub Pages for free.

## Put it on GitHub from your iPhone (about 10 minutes)

**1. Unzip the files**
- Save the ZIP to the **Files** app, then tap it. Files makes a folder next to it.
- That folder holds about ten files, with no sub-folders. Keep them together.

**2. Make a new repository**
- In **Safari**, go to **github.com** and sign in.
- Tap your picture (top right), then **Your repositories**, then **New**.
- Name it `the-ev-hub` and choose **Public**. Free GitHub Pages sites need a public repository.
- Leave every box unticked and tap **Create repository**.
- This is a brand-new repository. It doesn't touch PropLab.

**3. Upload the files**
- On the new, empty repository page, tap the link **uploading an existing file**.
- Tap **choose your files**, then **Browse**, and open the unzipped folder.
- Tap **Select** at the top, select **every file** in the folder, and tap **Open**.
- Wait until all the files show in the list, then tap **Commit changes**.
- If Safari shows the cut-down mobile page without these buttons, tap **aA** in the address bar, then **Request Desktop Website**.

**4. Turn on GitHub Pages**
- In the repository, tap **Settings**, then **Pages** (in the left menu, under "Code and automation").
- Under **Build and deployment**, set **Source** to **Deploy from a branch**.
- Set **Branch** to **main** and the folder to **/ (root)**, then tap **Save**.
- Wait one or two minutes and refresh the page. The address appears at the top: `https://YOUR-NAME.github.io/the-ev-hub/`

**5. Add it to your Home Screen**
- Open that address in **Safari**.
- Tap **Share**, then **Add to Home Screen**, then **Add**.
- It opens full screen, named "The EV Hub", and works offline.

**Updating later:** upload the new files the same way. Files with the same name are replaced. Don't delete anything first.

## What's inside

| File | What it is |
|---|---|
| `index.html` | The whole website: design, pages and demo data generator in one file |
| `manifest.webmanifest`, `sw.js` | The Home Screen app settings, and offline support |
| `icon-192.png`, `icon-512.png`, `icon-maskable-512.png`, `apple-touch-icon.png` | App icons |
| `version.txt` | Build number |
| `PHASE-2-PLAN.md` | The plan for real odds (OddsAPI.dev), statistics, the backend, the model and alerts |
| `ODDSAPI-DEV-COVERAGE.md` | What OddsAPI.dev's own documentation says it supports, and what it doesn't |
| `MODEL-AND-SCORING.md` | How the demo model, EV, fair odds, Kelly and "Best Overall" are calculated |

All paths are relative (`./…`), so the site works at `https://YOUR-NAME.github.io/the-ev-hub/` without changes.

## The maths (used everywhere on the site)

- **Fair odds** = 1 ÷ p.
  - For draw no bet: (1 − p(draw)) ÷ p(win).
- **EV (%)** = (p × O − 1) × 100, where p is the model probability and O the decimal bookmaker odds.
  - Draw no bet adds the stake back on a draw: (p × O + p(draw) − 1) × 100.
- **No price means no EV.** "No price available" is shown, never an invented price.
- **Stale prices** (older than 30 minutes) are shown greyed and never used for EV or "best price".
- **Betfair Exchange:** EV uses the price after its commission (5% by default; you can change it in Settings).
- **Kelly:** full Kelly = (p × O − 1) ÷ (O − 1). The calculator shows 1/10, 1/15 or 1/20 of it, and only when EV is positive. It is not a guarantee of profit.
- **Missing statistics are never zero.** For example, Ligue 1 "tackles won" is deliberately left unrecorded in the demo to show this. Missing values are left out of every average and hit rate.

## Your settings

Settings, filter presets and the bet-alert rules are saved in **this browser only** (local storage). Nothing is sent anywhere.
