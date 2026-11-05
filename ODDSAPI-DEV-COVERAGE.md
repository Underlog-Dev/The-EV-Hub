# OddsAPI.dev: coverage audit (from its own documentation)

**Checked:** 5 November 2026. No API key was used. Phase 1 makes no calls to OddsAPI.dev.

**Sources:**
- the OpenAPI specification at `https://api.oddsapi.dev/openapi.json` (version 0.1.0)
- the home page and the player-props page at `oddsapi.dev/markets/player-props`
- the Terms at `oddsapi.dev/terms`

Everything below must be **re-verified against real responses during the free trial**. Documentation is not proof that every market is present for every match.

## Connection (documented)

| Item | Documented value |
|---|---|
| Base URL | `https://api.oddsapi.dev/v1` |
| Authentication | `Authorization: Bearer <api_key>` header. Keys need an active plan or free trial. |
| Flow | `GET /bookmakers` → `GET /{bookmaker}/leagues` → `GET /{bookmaker}/events?league_id=…` → `GET /{bookmaker}/events/{event_id}/odds` |
| Other endpoints | `GET /health`; `GET /{bookmaker}/leagues/{league_id}`; `GET /{bookmaker}/events/{event_id}`; `GET /{bookmaker}/events/{event_id}/odds/live` |
| Odds filters | `?markets=player_shots,total_goals` (an unknown key returns 400); `?period=full_time\|first_half\|second_half` |
| Events filters | `league_id`, `team_id`, `status` (`scheduled`, `live`, `finished`, `removed`), `date_from`, `date_to`, `page`, `per_page` (up to 100) |
| IDs | `lg_…` league, `tm_…` team, `ev_…` event. Every ID belongs to **one** bookmaker: another bookmaker's ID returns 404. |
| Responses | One item is `{ "data": … }`. A list is `{ "data": [ … ], "meta": { page, per_page, total, total_pages } }`. An error is `{ "error": { code, message, status_code } }`. |
| Odds object | `event_id`, `bookmaker`, `type` (`prematch`/`live`), `updated_at` (when the bookmaker's board last changed), `markets[]` |
| Market | `key`, `period`, `player` (player markets), `name` (only on `other`), `outcomes[]` |
| Outcome | `key` (e.g. `home`, `over`, `yes`), `line` (totals, handicaps, player thresholds: `over 1.5` = 2 or more), `price` (decimal). Betfair Exchange also has `back`/`lay` ladders. |
| Limits | Every response carries `X-RateLimit-*` (per minute) and `X-Monthly-Quota-*` headers. Over a limit you get 429 with `Retry-After`. |
| Freshness | Cached a few seconds: live up to 10 s, pre-match up to 30 s, leagues 60 s. In-play boards refresh every 30 s. |
| Plans | Starter $50/month (100,000 requests, 120 a minute), Growth $129, Scale $379. Each has a 7-day free trial with 10% of the plan's monthly requests. |

## Bookmakers

**Live** (from the `bookmaker` enum in the spec):

| Bookmaker | Slug | Note |
|---|---|---|
| bet365 | `bet365` | |
| Paddy Power | `paddy-power` | |
| BetMGM UK | `betmgm-uk` | |
| Pinnacle | `pinnacle` | Does not accept UK customers, so it is useful as a sharp reference price, not to bet with. |
| Betfair Exchange | `betfair-exchange` | Exchange prices: EV must allow for commission. |

**Listed as "coming soon"** on the home page: William Hill, Betfred, Ladbrokes, and some non-UK books.

**Not listed:** Sky Bet, Coral, Betway, Unibet UK, BetVictor and others. These can't be offered through OddsAPI.dev today.

The demo uses the five live bookmakers plus the three "coming soon" UK ones, so you can see how selecting "my bookmakers" will work.

## Leagues

The site says "2,000+ leagues" across its bookmakers, and the spec's own example is "England Premier League". Leagues are listed **per bookmaker** under each bookmaker's own name. The same match therefore has a different ID at every bookmaker, so **The EV Hub must match events across bookmakers itself**, using kick-off time and team names. That mapping is a Phase 2 task, with checks.

## Markets The EV Hub needs → OddsAPI.dev market keys

| The EV Hub market | OddsAPI.dev key | Documented bookmakers | Status |
|---|---|---|---|
| Player shots | `player_shots` | bet365, BetMGM UK | ✅ documented |
| Player shots on target | `player_shots_on_target` | bet365, BetMGM UK, Paddy Power, Betfair Exchange | ✅ documented |
| Player tackles | `player_tackles` | bet365 | ✅ documented (one bookmaker) |
| Player **tackles won** | — | — | ❌ **not in the market list**: the model can show a probability, but there will be no price |
| Player fouls committed | `player_fouls` | bet365 | ✅ documented (one bookmaker) |
| Player fouls drawn | `player_fouls_won` | bet365 | ✅ documented (one bookmaker) |
| Player passes | `player_passes` | bet365 | ✅ documented (one bookmaker) |
| Player to be booked | `player_to_be_booked` | bet365 (per the props page) | ✅ documented |
| Goalkeeper saves | `goalkeeper_saves` | bet365 | ✅ documented |
| Anytime goalscorer / 2+ | `anytime_goalscorer`, `player_to_score_2_plus` | Paddy Power, BetMGM UK (bet365 to confirm) | ✅ documented |
| Player assists | `player_to_assist`, `player_assists` | not stated | ⚠️ key exists; coverage unknown |
| Match result / double chance / draw no bet | `match_result`, `double_chance`, `draw_no_bet` | all (expected) | ✅ |
| Match goals, BTTS | `total_goals`, `both_teams_to_score` | all (expected) | ✅ |
| Team goals | `home_total_goals`, `away_total_goals` | to confirm | ✅ key exists |
| Handicap | `asian_handicap`, `european_handicap` | to confirm | ✅ key exists |
| Match / team shots, shots on target | `total_shots`, `total_shots_on_target`, `home_/away_total_shots(_on_target)` | to confirm | ✅ keys exist |
| Match / team corners | `total_corners`, `home_/away_total_corners`, `corners_handicap` | to confirm | ✅ keys exist |
| Match / team cards | `total_cards`, `home_/away_total_cards` | to confirm | ✅ keys exist |
| Match / team **fouls** | — | — | ❌ **not in the market list**: model only, no price |

Markets outside the standard list come back as `key: "other"`, with the bookmaker's own names. These can't be matched reliably, so The EV Hub will ignore them unless they are mapped by hand.

## Things OddsAPI.dev does **not** document

- **No bookmaker or betslip links.** The schema has no URL fields. Bookmaker buttons must stay disabled, or open nothing, unless another verified link source is found. Links must not be invented.
- **No player or team statistics, results or history.** It is an odds-only service. Hit rates and the model need a **separate licensed statistics provider**.
- **No historical odds archive or opening prices.** Only the current board and `updated_at`. "Previous odds" and "movement" must come from our own snapshots, kept privately and only as long as the Terms allow.
- **No lineups or injuries.**

## Terms (summary, not legal advice)

- **Allowed (§9.2):** displaying odds, or information derived from them, to your own users within your own application; caching temporarily as needed to run the app; use in your own models and research.
- **Not allowed:**
  - redistributing the data as a feed, file download or API (§9.3, §10.1, §10.4);
  - bulk archiving to redistribute (§10.3);
  - building a competing odds service (§10.2);
  - sharing or publishing API keys, or putting them in client-side code or public repositories (§5.2, §5.4, §10.5).
- **Attribution** may be required, e.g. "Odds data by OddsAPI.dev" (§11.1).
- **When the account ends,** raw data must be deleted within 30 days (§17.4).
- **Paid memberships:** not addressed directly. Ask OddsAPI.dev before charging users.

### What this means for The EV Hub

The API key lives only in a private backend. GitHub Pages never sees it. The website shows prices and EV, but never publishes downloadable raw odds files. An "Odds data by OddsAPI.dev" credit is added in Phase 2.
