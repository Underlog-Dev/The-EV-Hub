# The EV Hub: Phase 2 plan (real odds, statistics, backend, model, alerts)

**Phase 1 (now):** a demo website on synthetic data, on GitHub Pages, with no keys and no backend.

**Phase 2 starts only after you approve the Phase 1 design.** It begins with OddsAPI.dev's 7-day free trial. Nothing here has been built or tested against the real API yet.

## 1. Architecture (simplest reliable, low cost)

```
OddsAPI.dev ──┐                       ┌── GitHub Pages: The EV Hub website (public, no keys)
              ├─► Private backend ────┤      reads only from the backend's own API
Statistics ───┘   (Cloudflare Worker  └── Telegram bot (alerts)
provider          + D1 database,
                  scheduled every few minutes)
```

**Backend: Cloudflare Workers and D1** (SQLite).
- The free tier covers a personal site: scheduled jobs ("cron triggers"), encrypted secrets for the API keys, a small database, and HTTPS endpoints.
- Setup can be done in a browser, with no computer needed: the Cloudflare dashboard works on an iPhone.
- The code can be pasted or deployed from a private GitHub repository.
- **Alternative:** a private GitHub repository running GitHub Actions on a schedule. It is cheaper still, but it can only run every 5 minutes at best, is often delayed, and is fine for odds snapshots but weak for alerts.

**The website** stays on GitHub Pages.
- It swaps the demo adapter for a **backend adapter** that calls *our* backend, never OddsAPI.dev directly.
- The API key never appears in the website, the public repository or the browser.

## 2. Provider adapters (switchable without rebuilding the site)

The website only ever sees one normalised shape. Each provider has its own adapter in the backend.

| Adapter | Job | Phase 2 choice |
|---|---|---|
| `odds` | Bookmaker prices: `{ fixtureId, bookmaker, market, player?, team?, side, line, price, updatedAt, link? }` | **OddsAPI.dev** (primary) |
| `fixtures` | The fixture list, with one fixture ID across all bookmakers | Built from OddsAPI.dev events, matched across bookmakers |
| `playerStats` | Match-by-match player statistics (minutes, shots, SOT, tackles, fouls, fouled, passes, cards, saves) | **To choose:** OddsAPI.dev has none |
| `teamStats` | Team match statistics (shots, corners, cards, fouls) | Same provider as player statistics |
| `history` | Several seasons of the above, for the model | Same provider, if the licence allows storage and model use |

**Switching provider** means writing one new adapter that outputs the same shape. The website and the model don't change. The demo adapter in Phase 1 already produces this shape from the synthetic data.

## 3. OddsAPI.dev integration (from its documentation; verify during the trial)

**Calls** (base `https://api.oddsapi.dev/v1`, header `Authorization: Bearer <key>`):

1. `GET /bookmakers`, then `GET /{slug}/leagues?search=premier` for each of `bet365`, `paddy-power`, `betmgm-uk`, `betfair-exchange`, `pinnacle`. Note each bookmaker's own Premier League ID.
2. Every 10–15 minutes: `GET /{slug}/events?league_id=…&status=scheduled&date_from=today&date_to=+7 days`.
3. For each upcoming event and bookmaker: `GET /{slug}/events/{id}/odds?markets=<the keys below>&period=full_time`.

**Market keys requested**, priority order:
- `player_shots`, `player_shots_on_target`, `player_tackles`, `player_fouls`, `player_fouls_won`, `player_passes`, `player_to_be_booked`, `goalkeeper_saves`
- `anytime_goalscorer`, `player_to_score_2_plus`, `player_to_assist`
- `match_result`, `double_chance`, `draw_no_bet`, `total_goals`, `both_teams_to_score`
- `home_total_goals`, `away_total_goals`, `asian_handicap`
- `total_corners`, `home_total_corners`, `away_total_corners`
- `total_cards`, `home_total_cards`, `away_total_cards`
- `total_shots`, `total_shots_on_target`, `home_total_shots`, `away_total_shots`, `home_total_shots_on_target`, `away_total_shots_on_target`

**Not available at OddsAPI.dev:** tackles won, match fouls and team fouls. These stay "model only, no price".

**Matching events across bookmakers:** every bookmaker has its own event IDs and team names. Events are matched on kick-off time (±10 minutes) and normalised team names, using a hand-checked alias table. Unmatched events are listed for review, never guessed.

**Matching players:** player names come as text ("Erling Haaland"). They are matched to the statistics provider's player IDs with an alias table, and unmatched players are flagged.

**Line convention:** `over 1.5` = 2 or more. This is the same as The EV Hub's "2+".

**Budget check during the trial:**
- Count the requests: events × bookmakers × refreshes per day.
- The trial allows 10% of the plan's monthly requests (Starter: about 10,000). Request only Premier League, pre-match only, and refresh every 15 minutes at first.

**Rate limits:** read the `X-RateLimit-*` and `X-Monthly-Quota-*` headers on every response. On a 429, wait for `Retry-After`. Stop early when the monthly quota is low.

**Outages:** if a call fails, keep the last good prices but mark them with their real `updated_at`. After 30 minutes they become **stale** and are never used for EV.

**Odds movement:** store a small snapshot of each price when it changes, for movement charts. Keep it for the season at most. It is never published as raw data (Terms §10.1 and §10.3) and is deleted if the account ends (§17.4).

**Attribution:** show "Odds data by OddsAPI.dev" wherever its prices appear, if required (§11.1).

**Bookmaker links:** OddsAPI.dev documents no deep links. Link buttons stay disabled unless a verified link source exists for a selection. Nothing claims a bet has been added to a betslip.

## 4. Statistics provider (decision needed in Phase 2)

OddsAPI.dev supplies odds only. A licensed provider is needed for player and team match statistics and history.

**Requirements:**
- Premier League first, with per-player match statistics for shots, SOT, tackles, fouls, fouled, passes, cards, saves and minutes.
- Terms that allow storage and model training for our own use.
- Clear limits on what can be shown publicly.

**Candidates to evaluate**, comparing price and licence before choosing, with none chosen yet:
- API-Football
- Sportmonks
- Opta or Stats Perform, which is expensive

Any paid choice needs your approval.

## 5. The real model (only after data is licensed)

1. Load up to five seasons of history.
2. Fit the player-count model (rate per 90 × expected minutes × opponent × venue, negative-binomial) and the team model.
3. **Backtest chronologically:** fit on matches before each date and predict the next round, for every round.
4. **Calibrate per market**, using reliability curves, Brier score and log loss.
5. Show a market only when it calibrates, and compare against closing prices.
6. Replace the demo confidence weights in "Best Overall" with the measured calibration.
7. **Expected lineups and injuries:** include them only from a licensed source. Until then, show "lineup not confirmed".

## 6. Alerts

**Telegram first:**
- Create a bot with @BotFather (free) and send its token to the backend as a secret.
- The backend checks your saved alert rules each time prices refresh. The rules are the same as on the Alerts page.
- **Duplicates:** one alert per selection and bookmaker per window, unless the price improves again.
- **Stale prices:** a price older than your limit never alerts.
- **Rate limit:** at most N alerts an hour.
- Every alert shows the price time, the model probability and EV, and the words "check the price at the bookmaker before betting".

**iPhone web push (later):**
- Possible on iOS 16.4 or later, but only when The EV Hub is added to the Home Screen.
- It needs push keys in the backend. It is more work than Telegram, so it comes second.

## 7. Phase 2 steps for you (short, iPhone-friendly)

1. Approve the Phase 1 design.
2. Start the OddsAPI.dev free trial and create an API key. Never paste it into GitHub.
3. Create a free Cloudflare account. I'll give exact tap-by-tap steps to add the key as a secret and deploy the backend.
4. Choose a statistics provider. I'll compare the options and prices first.
5. Create the Telegram bot and send its token to the backend secret store.

## 8. Before going live (checklist)

- [ ] Real OddsAPI.dev responses checked against this plan: market keys, lines, bookmakers and freshness.
- [ ] Event and player matching reviewed, with no unmatched events silently dropped.
- [ ] No API key anywhere in the public repository or the website, checked by searching the files.
- [ ] No downloadable raw-odds files are published.
- [ ] Stale-price handling tested (prices older than 30 minutes never used).
- [ ] Model backtest and calibration results published on the site, before any probability is called a "model".
- [ ] The Demo label removed only from data that is real.
