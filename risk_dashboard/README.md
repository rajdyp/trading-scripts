# Risk Dashboard

`risk_dashboard.pine` is a Pine Script v6 indicator that combines a market-regime check with a watchlist position-sizing table, in one compact dashboard on the price chart.

The regime is baselined on Nasdaq instruments by default (QQQ, QQQE and Nasdaq breadth and new-high/new-low indexes). All of the symbols are settings, so you can point it at another market.

## What it shows

```
MARKET  NO-GO  ·  Oct 02  ·  NH/NL negative for 3 days                  <-- verdict strip
QQQ ✓   QQQE ✓   VIX 16.39 ✓   BR200 49.95% ~   NH/NL 3D- ✗             <-- gate chips
$100,000 · 1% risk · Full · 4 symbols | 20D … · 50D … · Live VIX …
Ticker  Day%  Vol vs Avg  Entry  Stop  Stop%  R/ATR  Shares  Pos$  Risk
```

The NH/NL net new highs minus new lows histogram is drawn in the indicator pane. Optionally, the price chart background is shaded by regime.

## Market regime

QQQ and QQQE pass when their completed daily close is at or above both the 10-day and 21-day EMAs.

| Regime | Rule (defaults) |
| --- | --- |
| **NO-GO** | VIX ≥ 20, 200-day breadth < 45%, net NH−NL negative on all three completed days, or both QQQ and QQQE fail. |
| **FULL GO** | VIX < 20, breadth ≥ 50%, net NH−NL positive on all three completed days, and both QQQ and QQQE pass. |
| **SELECTIVE GO** | No blocking condition, but a FULL GO requirement is missing. |
| **UNAVAILABLE** | Required regime data is missing. No regime alert fires. |

Daily requests use the latest snapshot whose closing time has passed (the current clock in realtime, the bar's closing time in history). The date in the verdict strip is QQQ's selected daily bar. 20/50-day breadth and live VIX are context only and never change the regime.

Chart background (optional, on by default): green = FULL GO, amber = SELECTIVE GO, red = NO-GO, applied to every historical bar.

## Watchlist

Paste up to 15 rows in Settings → Watchlist rows:

```text
AAPL,3.2
NVDA,125,,4.5,0.5,Half
```

- Short format: `ticker,ATR%`
- Full format: `ticker,buy price,stop,ATR%,risk% override,exposure override`. Keep the commas for empty fields (for example `AAPL,,,3.2`).
- Entry defaults to the last price and Stop to the day's low unless you supply them.
- Rows are sorted by Stop% ascending, so the lowest-risk names are on top.

Sizing is independent of the market regime. Shares are the smaller of the risk-based size (portfolio × risk% ÷ risk per share) and the exposure cap: Full 20%, Half 10%, Quarter 5%, Probe 2.5% of the portfolio, or No Trade.

### Columns

| Column | Meaning |
| --- | --- |
| Day% | Change from the prior close |
| Vol vs Avg | Projected full-day volume compared with the average daily volume of the prior N completed days (default 50), as a % change. `+100%` means on pace for double the average. Today's volume is scaled by the elapsed share of the 9:30–16:00 ET session (floored at 15 minutes); outside the live session the last full day is used. It is approximate early in the day because volume is front-loaded. Bright green ≥ +100%, green ≥ +50%, white ≥ 0%, muted below. |
| Entry, Stop, Stop% | Entry and stop prices and the percent distance between them |
| R/ATR | Stop distance in multiples of the ATR% you entered |
| Shares, Pos$ | Position size in shares and dollars |
| Risk | Dollar risk and percent of the account, for example `$95.00 (0.49%)` |

Detailed view also shows Last, LOD and ATR%. Hover a ticker for the price time, data status, volume and average volume.

## Settings

| Group | Setting |
| --- | --- |
| Market regime | VIX blocking threshold, minimum and full-GO 200-day breadth |
| Market data | Benchmark symbols (default QQQ, QQQE), VIX, and the breadth and new-high / new-low symbols (defaults are Nasdaq indexes) |
| Watchlist, sizing | Watchlist rows, portfolio value, max account risk, default exposure |
| Display | Compact or Detailed view, table position (default bottom right), text size, column width %, show dashboard, regime background tint, live VIX |
| Relative volume | Average volume lookback, and the green and bright-green thresholds (% vs average) |

If the table looks cramped or clipped on a narrow chart, lower "Table column width (%)".

## Alerts

`Risk Dashboard FULL GO`, `Risk Dashboard SELECTIVE GO` and `Risk Dashboard NO-GO` fire when the regime changes. Alerts are tied to the saved script, so recreate them if you replace the indicator.

## Data requests

9 market requests plus 2 per watchlist row, at most 39 with 15 symbols. This stays within TradingView's standard 40-request limit, which is why the volume figures reuse the existing daily request.
