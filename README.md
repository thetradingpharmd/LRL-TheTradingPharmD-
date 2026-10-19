# LRL [TheTradingPharmD] — NinjaTrader 8

**Standalone NinjaScript indicator | Port of the TradingView Pine Script v6 LRL indicator**

## Overview

LRL (Low Resistance Liquidity) identifies clusters of confirmed swing highs or swing lows that fit an orderly liquidity line. It highlights potential pools of resting liquidity above or below price, including near-horizontal clusters, descending highs and ascending lows. The indicator is designed for chart context, not automatic trade execution.

This NT8 port aims to preserve the TradingView model's principal structure: pivot confirmation, ATR-adjusted trendline fit, active/untapped swings, distinct LRL runs, recent-line visibility filtering, sweep alerts, and an optional Liquidity Draw readout.

## Features

- **LRL lines:** Identifies qualifying clusters containing at least 3 confirmed swing points (configurable from 3–8).
- **Swing detection:** Sensitive (3 bars), Medium (5 bars, default), or Strict (8 bars) pivot strength on each side.
- **Untapped swing dots:** Marks confirmed swing highs/lows and removes dots once price trades through their levels.
- **Directional requirements:** LRL above price uses flat-to-descending swing highs; LRL below price uses flat-to-ascending swing lows.
- **ATR filters:** Trendline-fit tolerance, maximum slope, and optional connector-break tolerance.
- **Persistent runs:** Tracks active and historical LRL segments; can display only the latest distinct lines while suppressing visually overlapping same-side lines.
- **Liquidity Draw panel:** Optional summary of unresolved high/low swing counts, imbalance, immediate-range comparison and nearest distances.
- **Alerts:** LRL above formed, LRL below formed, any LRL formed and active LRL swept.

## Installation in NinjaTrader 8

1. Back up your existing NinjaScript indicators.
2. Open **New > NinjaScript Editor**.
3. Under **Indicators**, create or open a file named **`LRLTheTradingPharmD.cs`**.
4. Replace its contents with the full code from `LRLTheTradingPharmD_CompileFix.cs` (or paste the contents of the `.txt` version into the C# editor).
5. Ensure **only one indicator class named `LRLTheTradingPharmD`** exists in your NinjaScript project. If `MyCustomIndicator.cs` contains an earlier copy of the LRL class, remove that duplicate *only after confirming it contains nothing else you need*.
6. Press **F5** to compile, resolve any remaining errors, then add **LRL [TheTradingPharmD]** to a chart from the Indicators menu.

**Important:** NinjaTrader generates an accessor/cache section automatically. Do not hand-edit the generated section to resolve duplicate definitions. NinjaTrader compiles NinjaScript files together; an error in another file can block this indicator from compiling.

## Recommended default settings

| Setting | Default | Meaning |
|---|---|---|
| Show Low Resistance Liquidity | On | Enables LRL line detection and display |
| Minimum Swing Points | 3 | Minimum qualifying swing members |
| Swing Detection | Medium | 5-bar left/right confirmed pivots |
| Active / Untapped Only | On | Consumed swings do not qualify |
| Allow Line Crossing | On | Does not require every candle to respect the connector |
| Line Fit Tolerance (ATR) | 0.50 | Maximum deviation of participating pivots from the fitted line |
| Max Bars Between Swings | 30 | Maximum spacing of accepted pivot members |
| Show Recent Only | On | Restricts displayed lines, not stored runs |
| Max LRL Lines | 2 | Number of recent distinct LRL lines shown |
| Show Text | On | Displays `LRL` near the line endpoint |
| Max Steepness | 0.50 ATR/bar | Caps fitted line slope |
| Max Break (ATR) | 0.15 | Applied when line crossing is disallowed |
| Swing Buffer | 40 | Recent confirmed swings held per side |
| Show Liquidity Draw | Off | Enables chart-corner contextual readout |
| Immediate Draw Range | 3.0 ATR | Distance used for nearby swing count |
| Minimum Swing Difference | 2 | Count difference threshold |
| Minimum Imbalance Ratio | 1.50 | High-to-low count imbalance threshold |
| Draw Qualification | Either | Difference **or** ratio can qualify |
| Show Untapped Swing Dots | On | Plots active swing markers |
| Swing Dot Offset | 10 ticks | Dot offset from the pivot high/low |

## How the LRL calculation works

1. A swing high or low is added **only when the required bars to its right have completed**. The dot is plotted at the original swing bar; this is a confirmed-pivot annotation, not advance knowledge at that bar.
2. Previously stored swings are marked consumed when the high trades **above** an active swing high or the low trades **below** an active swing low. Equal touches do not consume the level in this implementation.
3. Candidate start/end swings must slope in the qualifying direction. Intermediate pivots are included only when their deviation from the line and their spacing satisfy the settings; off-line pivots may be skipped.
4. Qualified candidates are ranked by number of swing members, line fit, endpoint recency, distance from current price, and slope.
5. Previously tracked runs are updated while qualified; new runs can form on newly confirmed pivots. The indicator retains a bounded history of 60 runs and displays the most recent non-overlapping runs according to the visibility settings.

## Liquidity Draw panel

The optional panel compares **active swing highs** with **active swing lows** within the configured buffer:

- **Structural draw:** all unresolved swings in the buffer.
- **Immediate draw:** unresolved swings within the selected ATR radius of current price.
- **Balanced:** neither side meets the required count difference/ratio, or counts are equal.
- **Nearest high/low:** point distance from price to the nearest active swing of each type, when enabled.

The NT8 port uses a **fixed text panel**, not the multi-cell TradingView table. The panel is contextual; it does not change LRL qualification or generate trade orders.

## Alerts

- **Bullish LRL Formed:** a qualifying LRL of highs forms above price (potential upside liquidity target).
- **Bearish LRL Formed:** a qualifying LRL of lows forms below price (potential downside liquidity target).
- **Any LRL Formed:** either formation event.
- **LRL Swept:** price trades beyond the previously tracked nearest active LRL endpoint, when enabled.

The port uses NT8 `Alert(...)` messages and the installation's `Alert1.wav`. Alerts are processed at **bar close** in this version; this is not a tick-by-tick intrabar notification engine.

## Appearance and known differences from TradingView

- TradingView's customizable LRL/dot colors are **fixed in this initial NT8 port**: dark gray LRLs, burgundy high dots and blue low dots.
- The **Liquidity Draw panel is text-based** rather than an exact TradingView table replica.
- Pivot labels appear at the pivot bar after confirmation, so historical plots must not be interpreted as signals known in real time on that bar.
- This is a **standalone indicator**, not an automated strategy. It does not place orders, define stops or targets, or guarantee that liquidity will be swept.

## Troubleshooting

**CS0246 / CS0103 — `DashStyleHelper` not found**  
Use the compile-fix source that includes `using NinjaTrader.Gui;` with the existing NinjaTrader chart/drawing namespaces.

**CS0229 — ambiguous `LRLTheTradingPharmD` references**  
Search indicators such as `MyCustomIndicator.cs` for a second `class LRLTheTradingPharmD`. Keep only one definition; don't remove an unrelated indicator accidentally. Do not edit NT8-generated cache/accessor code.

**Indicator not visible**  
Check that F5 completes without errors, `Show Low Resistance Liquidity` is enabled, sufficient bars have loaded, and at least three confirmed swing members satisfy the chosen filters. The default recent display cap is two lines.

**Different behavior from TradingView**  
Compare the **same instrument, timeframe, session template, price data and parameter values**. Pivot confirmation and session/history differences can change outcomes. Compare screenshots with bar-by-bar timestamps before assuming equivalent signals.

## Validation status

**Not yet verified as successfully compiled or visually parity-tested in a running NT8 installation.** The previous user screenshot showed duplicate-class compile errors (`CS0229`) involving `LRLTheTradingPharmD.cs` and `MyCustomIndicator.cs`. Resolve those duplicate definitions before functional testing.

Suggested validation sequence: compile; load chart; verify dots and pivot lag; verify formation and removal of LRLs; check panel counts; test formation/sweep alerts; compare output against TradingView on matching data.

## License and attribution

Original Pine Script copyright: **© TheTradingPharmD**, supplied under **Mozilla Public License 2.0**. Preserve the original attribution and applicable MPL 2.0 obligations when modifying or distributing derivatives. See https://www.mozilla.org/MPL/2.0/.

---

*Documentation for the initial NinjaTrader 8 port. Confirm platform compilation and performance before using it in a live trading workflow.*
