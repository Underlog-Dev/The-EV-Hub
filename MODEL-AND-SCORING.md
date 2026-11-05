# The EV Hub: demo model, EV and scoring

**Phase 1 uses a DEMO model on SYNTHETIC data.** It is not trained on real football. It has not been backtested or calibrated. It exists so that every page, filter and calculation can be tried out.

## The synthetic world

- **What it contains:** 5 competitions (real names, fictional clubs), 8 clubs each and 18 players per club. Each club has 21 played matches and 2 upcoming rounds.
- **How it is generated:** from a fixed seed, so it is the same on every visit. Kick-off times are set relative to today, so "Today" and "Tomorrow" always work.
- **Player statistics:** every player has hidden per-90 rates for each statistic.
  - Match figures are drawn from these rates (negative-binomial counts), scaled by his minutes and the opponent.
  - Team totals are the sums of the players' figures.
  - Goalkeeper saves = shots on target faced − goals conceded.
- **Missing data on purpose:**
  - Ligue 1 has no "tackles won".
  - A few matches have no passing data and one has no corners.
  - Players who didn't play have no row at all (never a zero).
- **Injury news:** a few doubts and one absentee per league.

## The demo model

**Player markets:** P(stat ≥ line) is calculated in five steps.
1. Rate per 90: a recency-weighted average (each older match × 0.93), pulled towards the position average by 3 matches' worth of the average.
2. Expected minutes: recent average minutes × how often he played, × 0.6 if he is doubtful. A player who is out gets no probability.
3. Opponent: what the opponent allows to his position per 90, relative to the league. Only half of the difference is used.
4. Home/away: about ±4%.
5. Distribution: negative binomial (shots, tackles and so on), Poisson (goals, assists, cards).
- The 80% range is the same calculation at the low and high ends of his rate.

**Team and match markets:**
- Team for and opponent against rates are pulled towards the league average, and home advantage is applied.
- Goals: a Poisson score matrix gives 1X2, double chance, draw no bet, totals, BTTS, team goals and handicap.
- Shots, corners, cards and fouls: negative-binomial counts, combined for match totals.

**Hit rates** come from the stored history only, over the last 5, 10 and 20 matches and all matches. Missing values are skipped.

**Confidence** (High / Medium / Low) is a score out of 6 built from:
- sample size
- the width of the 80% range
- how stable his minutes are
- how much data is missing
- injury doubts

**Demo bookmaker prices** are built in four steps.
1. Start from the model's probability.
2. Move it by a "market difference" shared by every bookmaker, mirrored for over and under on the same line.
3. Add each bookmaker's own small difference and its margin. Betfair Exchange gets a small margin.
4. Round down to the usual fractional ladder.
- Coverage per bookmaker follows OddsAPI.dev's published market coverage, so many markets have only bet365.
- About 10% of quotes are stale.
- This is why only some selections are EV+ and most are negative, as with real bookmakers.

## Calculations

| Item | Formula |
|---|---|
| Fair odds | 1 ÷ p. Draw no bet: (1 − p(draw)) ÷ p(win). |
| EV % | (p × O − 1) × 100. Draw no bet: (p(win) × O + p(draw) − 1) × 100. |
| Exchange odds for EV | 1 + (O − 1) × (1 − commission) |
| Full Kelly | (p × O − 1) ÷ (O − 1). Shown as 1/10, 1/15 or 1/20, and only when EV > 0. |
| Best price | The highest current (not stale) price among the chosen bookmakers |

## "Best Overall" (illustrative demo score)

**Only selections with positive EV at the price are scored.** Those with zero or negative EV get no score. They are listed after the scored ones, ordered by EV.

Score = 100 × cautious EV × reliability.

- **Cautious EV** = ((p + pLow) ÷ 2) × O + push − 1. This is halfway between the model probability and the low end of its 80% range.
- **Reliability** is the product of five factors:
  - sample factor = n ÷ (n + 6)
  - confidence: High 1, Medium 0.8, Low 0.55
  - freshness: 1 up to 15 minutes, 0.9 after that. Stale prices are never used.
  - liquidity proxy = √(min(1, bookmakers pricing it ÷ 3))
  - long-odds discount = √(3 ÷ O) above 3.00
- A negative cautious EV is not multiplied, so those selections sit below the confident ones.

This is a transparent ranking rule, **not** a validated predictive model. Phase 2 replaces the confidence weights with measured calibration from chronological backtests.

## Before a real model can be trusted (Phase 2)

1. Licensed match-by-match history, ideally five seasons: player minutes and statistics, team statistics, results.
2. Chronological backtesting only: train on the past and test on later matches, with no look-ahead.
3. Calibration checks: when the model says 60%, does it happen about 60% of the time? Measured with reliability curves, Brier score and log loss, per market.
4. Comparison with closing bookmaker prices (closing-line value) as an honest benchmark.
5. Uncertainty for every probability, and a minimum sample before a market is shown.
6. Monitoring after launch, and switching off markets that stop calibrating.
