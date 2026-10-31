# A.W. Strategy on TradingView

`tradingview/aw_strategy.pine` is a Pine Script v6 indicator for index and metal futures (NQ/MNQ,
ES/MES, RTY, YM, GC/MGC) on 5 to 30 minute charts. It marks setups, draws ranked GEX and
options-flow levels with a plan for each, and fires alerts. It never places orders.

It's meant to run next to two indicators that stay on the chart, HaP AVWAP (use the English copy,
[below](#hap-avwap-in-english)) and Buyers & Sellers Profile + Dynamic S/R by Zeiierman. Their
levels, a liquidity sweep and rolling pressure all feed the one A.W. signal; see
[Indicators folded in](#indicators-folded-in).

- [Install](#install)
- [What's on the chart](#whats-on-the-chart)
- [The A.W. signal](#the-aw-signal)
- [Entries and exits](#entries-and-exits)
- [Indicators folded in](#indicators-folded-in)
- [GEX and options-flow levels](#gex-and-options-flow-levels)
- [Ten more playbooks](#ten-more-playbooks)
- [GEX + anchored VWAP](#gex--anchored-vwap)
- [Optional tables](#optional-tables)
- [HaP AVWAP in English](#hap-avwap-in-english)
- [Alerts](#alerts)
- [Limits of TradingView](#limits-of-tradingview)

## Install

1. Open the Pine Editor, paste `tradingview/aw_strategy.pine` over everything in it, save it as
   "A.W. Strategy" and click "Add to chart".
2. Copy [`gex/history.txt`](../gex/history.txt) and paste it into the settings under GEX levels →
   "GEX levels (paste gex/history.txt)". Pine can't download data, so paste it again whenever you
   want the newest levels; the [data pipeline](data-pipeline.md) refreshes the file during the
   session. Until it's pasted, the chart shows a reminder where the levels would be.

On TradingView's free plan CME and COMEX futures are delayed (the "D" badge on the chart). Live
signals need TradingView's real-time CME data add-on.

## What's on the chart

Nothing covers the chart. The three numbers next to "A.W. Strategy" in the status line (top left)
are:

| Value | Meaning |
|---|---|
| Delta | This bar's buying minus selling volume, green when buyers won the bar |
| Flow score | −1 (sellers in control) to +1 (buyers in control); ±0.5 is in control, ±0.2 leaning |
| Z-score | This bar's delta against the last 30 bars; past ±1.5 is unusual |

On the chart itself, kept to what matters now:

- the last 4 breaks of structure (BOS, CHoCH) and the 79% retracement of the current leg,
- the 6 fair value gaps and inverse gaps nearest to price (finished gaps are removed),
- the latest A.W. trade only: its LONG / SHORT label (with its score), stop, targets and exit,
- the nearest buy-side (BSL) and sell-side (SSL) liquidity pool,
- the [GEX and options-flow levels](#gex-and-options-flow-levels),
- big-print bubbles, the playbooks' labels while their trades are open, and the GEX + AVWAP line
  while it's live (each can be switched off).

"Keep past A.W. trades on the chart" and "Playbook labels on the chart: All" bring the history
back. Hover any label for the details.

## The A.W. signal

Ten factors, each met or not, scored on candle close:

| Factor | Met when |
|---|---|
| Structure | The last break of a 5-bar swing went the trade's way and the 30-minute structure agrees (the trend filter is a setting). A break against the trend is labelled CHoCH. |
| Fibonacci 79% | Price recently tagged the 79% retracement of the leg that broke structure and is still in the leg's lower half (upper half for shorts). |
| Level touched | Price touched a fair value gap or inverse gap (three-candle gaps of at least 0.15 ATR; one closed through flips into an inverse gap), the swing-low AVWAP (swing-high for shorts) or a volume-profile support (resistance) this bar or the last. |
| Reaction | The candle closed back off it: past the gap's midpoint, a bullish (bearish) close back above (below) the 79% level, the AVWAP or the volume level. |
| Order flow | More of the last 10 gap reactions were bullish than bearish (bearish for shorts): support respected or resistance broken. |
| Delta | Buyers dominate the bar (delta z-score past +1.5 or 60%+ of the volume) and cumulative delta rose over the last 5 bars or a bullish CVD divergence formed (mirror for shorts). Delta comes from 1-minute bars inside each candle. |
| Volume | At least the 20-bar average. |
| Session | London 02:00–05:00, New York AM 08:30–11:00 or PM 13:30–15:30, New York time. |
| Liquidity sweep | A sell-side pool was swept in the last 20 bars and price is still above it (a buy-side sweep for shorts). |
| Pressure | Rolling buying pressure leads selling by at least 0.25 and price is above a rising 21 EMA (the mirror for shorts). |

A LONG or SHORT needs structure, a touch of the 79% level or another level, a reaction there, a
session (a setting), a stop no wider than 3 ATR and a score of at least 6 of 10 (a setting). Hover
the label to see which factors lined up and which level was touched.

The flow score follows [SuperFlow](https://github.com/profitnotiondaily/atasx-superflow) with its
weights: absorption, delta z-score, big prints, effort versus result, CVD slope, rolling CVD and
volume. Big prints are 1-minute bars that traded 4x the usual volume, drawn as circles in five
sizes.

## Entries and exits

- **Entry** on the close of the signal candle.
- **Stop** beyond the reaction candle and the zone it came from, plus 0.25 ATR. Setups whose stop
  is wider than 3 ATR are skipped.
- **Targets** at 1R and 2R. The stop moves to breakeven after 1R.
- **Exits** (SELL or COVER) on the stop, the second target, a structure flip against the trade, or
  order flow disrespected (a support gap closed through while long, a resistance gap while short).

## Indicators folded in

The indicators below are rebuilt in A.W.'s own code with their default settings, so they feed the
signal without their own labels and tables on the chart. Their ideas, not their code; credit to
their authors.

| Indicator | In A.W. Strategy |
|---|---|
| HaP AVWAP / Anchored VWAP, by agahakanaga | The VWAP of the highs anchored at the most important swing high of the last 500 bars and the VWAP of the lows anchored at the most important swing low (10/5-bar pivots that moved at least 2 ATR away, scored by that move, the move into them, volume, wick, position in the range and age; broken ones score a quarter). A bullish candle that dips to the swing-low AVWAP and closes above it (HaP's S mark) is a level touched and held for longs; HaP's R mark for shorts. Keep HaP AVWAP on the chart to see the lines. |
| Buyers & Sellers Profile + Dynamic S/R, by Zeiierman | Each 5-bar swing checked against a 72-row volume profile of the 600 bars before it; a strong nearby node turns it into support or resistance, scored 0–100, removed on a close 0.1 ATR beyond it. A touch of a support that closes above it is a level touched (held on a bullish candle). Keep the indicator on the chart to see the levels. |
| Liquidity Sweep Reversal & Targets, by PhenLabs | Unbroken 10-bar swing highs (buy-side liquidity) and lows (sell-side). A sweep wicks through one and closes back with the other side absorbing (55%+ on close location, volume and body), no deeper than 1.5 ATR. It's the liquidity-sweep factor, and the nearest pool on each side is drawn as a dashed BSL / SSL line. |
| Rolling Candle Profile, by Zeiierman | Buying and selling pressure from each candle's travel, body, close location and wick rejection, with a 3-bar half-life, against a 21 EMA anchor. It's the pressure factor. |

Each has its own settings group and can be switched off (a switched-off factor counts as met).
The heavy profile and AVWAP loops run on the last 5,000 bars.

## GEX and options-flow levels

Gamma exposure (GEX) estimates where options dealers' hedging pushes price or pins it. The levels
come from the [data pipeline](data-pipeline.md) in the options' own prices and are moved onto the
futures with the futures-to-index ratio from regular-hours bars.

**Strong levels (the default):**

| Tag | Meaning |
|---|---|
| Call wall, Put wall | The strikes with the most positive and most negative GEX (thick lines). Grade A. |
| γ flip | Where dealers' total gamma changes sign (dashed orange). Above it moves tend to be calm and levels hold more; below it moves are faster and levels break more often. |
| #1 to #8 | The 8 biggest strikes by GEX, ranked by size. #1 to #3 are grade A, #4 to #8 grade B. |
| ★ OV #n | The level is also one of today's busiest option strikes (#1 is the busiest), so it's more likely to matter. |

L marks support below price (look for a bounce long), S resistance above (look for a bounce
short). Each level gets a status for the day: ✓ held (a test closed back with absorption or a
delta flip), ✗ broke (closed through with delta behind it: don't fade it) or · retest (touched
before without a result, so weaker). Hover a level for its plan: wait for the candle at the level
to close with absorption or delta flipping your way, then enter on the pullback, not the spike.

**Options-flow levels.** OV #1 to #4 mark today's busiest option strikes near price as dotted
lines: teal when calls dominate, pink when puts do. TradingView's Options Volume Profile only
draws on daily charts and Pine can't read another indicator's data, so these come from the same
CBOE data and show on every timeframe. They use the index and ETF options (NDX + QQQ for NQ), not
the CME futures options, so the strikes are close to the Options Volume Profile's but not
identical. Hover one for its plan.

Other settings under "GEX levels to show":

| Setting | Drawn |
|---|---|
| Trusted only | The walls and the flip, plus a big strike only with ★ OV and only until it breaks. Gold, silver, crude, gas and bonds show only their walls and flip. |
| Full ladder | Also every strike within 1.5% of price, green for positive GEX and red for negative, bolder the more gamma it holds |

When the newest levels are more than 36 hours old, the walls and the flip say "old" with their
date.

## Ten more playbooks

Each takes its own trades on candle close, prints its own small labels (▲ BO, ▼ RG and so on;
hover one for the reason) and keeps its own record.

| Playbook | Entry | Stop | Exit |
|---|---|---|---|
| Breakout | A close outside a tight 20-bar range (under 3 ATR tall) on 1.5x volume | Inside the range, under the breakout candle | 2R or the range height projected, whichever is further |
| Pullback | Trend up (structure plus 20 EMA over 50 EMA): a reversal candle at the 20 EMA, the 38–62% zone, or old resistance turned support | Under the last 5 bars | The last swing high |
| Range | 2+ touches of each side over 40 bars with ADX under 25: a reversal candle or RSI divergence near an edge | Just outside the edge, plus 0.25 ATR | The middle first, then the far edge |
| Mean reversion | A push outside the Bollinger band (20, 2) with RSI past 30/70, then a close back inside | Beyond the extreme of the push | The band middle (or VWAP) |
| News | A real move at a usual release time (8:30, 10:00, 14:00 ET): wait 10 minutes, then buy or sell the first pullback | 0.5 ATR beyond the pullback | 0.75R (stop to breakeven), then 1.5R, or out after an hour |
| Spread | NQ vs ES, or gold vs silver: the price ratio's z-score passes ±2 | z-score past ±3.5 | z-score back to 0. Scored in percent, not R |
| Scalping | 1–5 minute charts only: 9 EMA over 21 and a break of the last micro swing | 0.5 ATR | 1.25R, or a close back through the 9 EMA |
| Order flow | Absorption: heavy one-sided delta on high volume at a key level (session high or low, VWAP, the previous POC, a gap) while price doesn't follow, then a close back through that bar | Beyond the absorption bar | VWAP or the point of control, else 2R |
| GEX + order flow | Price tests a GEX level (the flip, a wall or a big strike) and closes back on its side, with absorption or a delta reversal | Beyond the level, plus 0.25 ATR | The next GEX level at least 1R away, else 2R |
| GEX + AVWAP | See below | Beyond the test's extreme (reversal) or the level (continuation), plus 0.25 ATR | The next GEX level at least 1R away, else 2R |

## GEX + anchored VWAP

From the video "Stop Guessing If a Gamma Exposure (GEX) Level Will Reverse — Use AVWAP" (Grateful
Futures Trading University): GEX tells you *where* price may turn, and a VWAP anchored at the start
of the move into the level tells you *whether* that move is still in control. The move's start is
the highest high (for a drop into support) or lowest low (for a rally into resistance) of the last
48 bars, and the move must be at least 1.5 ATR.

- **Reversal:** price holds the level and closes back through the AVWAP: the move lost control.
- **Continuation:** the level breaks while the AVWAP holds, price moves away, then a retest of the
  level holds. A close back through the AVWAP ends the trade.

An idea expires after 24 bars without a trigger. The AVWAP is drawn in purple: solid while a level
is being tested or a trade is open, dotted from the current move's start otherwise.

## Optional tables

Both are off so nothing covers the chart; switch them on in the settings.

- **Dashboard** (Dashboard → "Show the dashboard table"): structure, the 5/15/30-minute trend, the
  79% level, the nearest support and resistance gaps, order flow, delta, flow score, volume, GEX,
  session, the long and short scores, what the setup is waiting for, the signal's record, GEX +
  AVWAP and the Kronos 2-hour forecast (an experiment: grey until at least 10 calls are scored and
  55% were right).
- **Best long / short** (Strategies → "Show the best long / short table"): for each side, the plan
  with the best record on the current chart among those within 4 ATR of price that offer at least
  1R. Records with few trades are discounted. Levels are estimates, not orders.

## HaP AVWAP in English

`tradingview/hap_avwap_english.pine` is an English translation of "HaP AVWAP / Anchored VWAP" by
agahakanaga (Mozilla Public License 2.0, kept with the author's credit). The logic and defaults
are the original's; every setting, label, tooltip and alert is in English, and its info panel (a
table) starts off. Install it the same way and save it as "HaP AVWAP (English)".

## Alerts

Create alert → Condition "A.W. Strategy" → "Any alert() function call". One alert then covers
every A.W. entry and exit and every playbook entry and exit, with the stop and target in the
message. Single conditions are listed too: long signal, short signal, sell (exit long), cover (exit
short) and any playbook entry.

## Limits of TradingView

- **Order flow:** Pine has no order book, time and sales or footprint data on regular plans, so
  delta comes from 1-minute bars and volume profiles from bar ranges.
- **News:** Pine has no economic calendar. The News playbook watches the usual release times and
  only trades a real move there.
- **Spread:** both legs are shown, but TradingView can't place a two-leg order.
- **GEX data:** Pine can't download files, so the levels have to be pasted into the settings.
