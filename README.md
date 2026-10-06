# OPEN5 — Opening Auction Intelligence

A TradingView (Pine Script v5) indicator that builds a **real-time, probabilistic auction thesis** for the New York cash open. It is **not** a buy/sell signal generator and it does **not** try to guess whether the first 5-minute candle will close green or red.

Instead, every NY session it continuously answers five questions:

1. **WHERE** is price most likely trying to go? (destination / liquidity target)
2. **WHY** is it likely going there? (plain-English reasoning)
3. **WHAT** evidence currently supports the thesis? (component scores)
4. **WHAT** would invalidate the thesis? (invalidation + flip conditions)
5. **HOW** strong is the overall case? (composite 0–100 conviction score)

Designed primarily for **NQ futures** on **1-minute and 5-minute** charts, using higher-timeframe and session levels for context. It works on any symbol you load on the chart.

## Two auto-switching modes

The engine evaluates the market continuously all session, but it clearly signals *which question it is answering* right now:

- **◉ OPENING DRIVE** — active during the **first 5 minutes** after the NY open (9:30–9:35). This is the headline: the opening-move prediction. The opening-5-min candle becomes a **dominant, scored component**, and the weights shift toward opening-relevant evidence (overnight location, gap, sweep, opening candle). At the 5-minute mark the call is **frozen**.
- **▸ CONTINUATION** — the rest of the session. The engine keeps evaluating live, but it also shows the **frozen opening call** and its live status: `• STILL VALID`, `✓ TARGET HIT`, or `✗ INVALIDATED`.

A running **HIT RATE** (targets hit ÷ directional opening calls) accumulates across sessions so you can measure how often the opening-move call actually works out — the dashboard shows it as `NN%  (hits/total)`.

> File: [`OPEN5_OpeningAuctionIntelligence.pine`](OPEN5_OpeningAuctionIntelligence.pine)

---

## Core Output

Every NY session the indicator produces three primary outputs:

- **DIRECTION** — bullish, bearish, or neutral (with a 0–100 conviction score)
- **DESTINATION** — the most probable liquidity target (primary + secondary)
- **INVALIDATION** — the specific price/action that would void the current thesis

Example (as shown in the dashboard and the "WHY?" chart label):

```
BULLISH 78/100

WHY:
1. ONL swept and rejected (sell-side liquidity cleared)
2. Price trading above the overnight midpoint
3. Price accepting above NY VWAP
4. Market structure: higher highs + higher lows
5. PDH is the strongest untouched liquidity above
6. Relatively clean liquidity path toward the target

→ TARGET: PDH
→ INVALIDATION: Opening Low / ONL / VWAP Loss
THESIS FLIP IF:
• Loss of VWAP
• Failure below ONM/ONL
• Bearish displacement
• Opening low violation
```

---

## Engines

The projection is assembled from independent analysis engines, each producing a 0–100 score where **>50 = bullish evidence** and **<50 = bearish evidence**:

| # | Engine | What it measures |
|---|--------|------------------|
| 1 | NY Opening Session | Tracks the open price, first 5-min candle, first 15-min range, session high/low |
| 2 | Liquidity Pool | Scores PDH/PDL, PWH/PWL, ON H/L/mid, London, Asia, prev close, swings (0–100 magnet score) |
| 3 | Liquidity Destination | Combines liquidity location **+ delivery** so liquidity above price ≠ automatically bullish |
| 4 | Liquidity Delivery | Up/down/net delivery from candle bodies, structure, VWAP acceptance |
| 5 | Overnight Range | Classifies open location (upper 20% / half / mid / lower half / lower 20%) and breakout/sweep/accept/reject |
| 6 | Liquidity Sweep / Trap | Detects sweeps of ON H/L, PDH/PDL, London — distinguishes acceptance vs. "SWEEP — NO ACCEPTANCE" |
| 7 | Opening 5-Minute Candle | Classifies the opening candle (displacement, sweep, trap, reversion, rotational, acceptance/rejection) |
| 8 | VWAP | Acceptance / reclaim / loss / rotation. Repeated crosses → "ROTATIONAL AUCTION" |
| 9 | FVG | Detects bullish/bearish gaps; classifies fresh / partial / mitigated / inverted; supporting vs. obstructing |
| 10 | Liquidity Path | Scores obstacles (levels, obstructing FVGs) between price and the target (0–100) |
| 11 | Opening Gap | Classifies gap size and gap-into-liquidity vs. gap-away conditions |
| 12 | Market Structure | HH/HL/LH/LL, break of structure, change of character |
| 13 | **Composite Auction Score** | Weighted blend of all engines (0–100) |
| 14 | Real-Time Thesis | Updates every bar; asks "is the original thesis still valid?" |
| 15 | Scenario Flip | Shows exactly what would flip the projection and auto-flips when it happens |
| 16 | Visual Projection | Target zones, invalidation, projected path, FVG boxes, opening-range box |
| 17 | Dashboard | Compact right-side panel with every engine's read |
| 18 | "Why?" Explanation | Dynamic plain-English reasoning on the chart |
| 19 | No-Trade Detection | Says "NO CLEAR AUCTION EDGE / DO NOTHING" in poor conditions |

### Composite Score Weights (default)

| Component | Weight |
|-----------|--------|
| Liquidity Destination | 20% |
| Price Delivery | 20% |
| Overnight Location | 15% |
| Liquidity Sweep / Acceptance | 15% |
| VWAP State | 10% |
| Market Structure | 10% |
| FVG Alignment | 5% |
| Liquidity Path | 5% |

### Conviction bands

| Score | Meaning |
|-------|---------|
| 0–44 | LOW QUALITY / NO CLEAR EDGE |
| 45–59 | CONFLICTING |
| 60–74 | MODERATE DIRECTIONAL CASE |
| 75–89 | HIGH-CONVICTION DIRECTIONAL CASE |
| 90–100 | EXTREME CONFLUENCE |

The indicator does **not** force a directional signal when the score is low.

---

## Installation

1. Open **TradingView → Pine Editor**.
2. Create a new indicator and paste the contents of [`OPEN5_OpeningAuctionIntelligence.pine`](OPEN5_OpeningAuctionIntelligence.pine).
3. Click **Save**, then **Add to chart**.
4. Load an NQ futures chart on the **1-minute** or **5-minute** timeframe.

---

## Settings

- **Session Times** — fully configurable NY / Overnight / Asia / London sessions and timezone (default `America/New_York`, NY open `09:30`).
- **Composite Score Weights** — tune the weight of each engine.
- **Liquidity Detection** — swing lookback, equal-high/low threshold, liquidity lookback.
- **FVG Settings** — minimum gap size and detection lookback.
- **Visual Settings** — toggle dashboard, levels, FVGs, projection zone, and the "Why?" label.
- **Colors** — customize every visual element.

---

## Alerts

Built-in `alertcondition`s:

- High-Conviction BULLISH (score ≥ 75)
- High-Conviction BEARISH (score ≥ 75)
- Thesis Flip
- No Clear Edge
- Liquidity Sweep / No Acceptance

---

## Philosophy

> LIQUIDITY + LOCATION + DELIVERY + STRUCTURE + ACCEPTANCE/REJECTION + LIQUIDITY PATH = PROBABILISTIC PROJECTION

OPEN5 is a **thesis builder**, not a prediction machine. It prioritizes explainability, liquidity context, and scenario changes over signal frequency — and it is explicitly allowed to say "do nothing."

---

## Disclaimer

This indicator is a decision-support and educational tool. It does not provide financial advice and makes no guarantee of future performance. Trade at your own risk.
