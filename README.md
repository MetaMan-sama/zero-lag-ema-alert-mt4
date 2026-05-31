# Zero-Lag EMA Alert — MQL4 Script

A MetaTrader 4 script that computes a **Zero-Lag EMA (ZLEMA)** each cycle using a two-EMA lag-correction formula — `ZLEMA = EMA₁ + (EMA₁ − EMA₂)` — and fires two independent alert types: a **price-ZLEMA crossover** when the current close transitions through the ZLEMA level using persistent `PrevZLEMA` and `PrevPrice` globals, and a **trend flattening** signal when the ZLEMA's bar-to-bar change falls within `Point × 10` — detecting momentum exhaustion before the actual crossover.

---

## Overview

Standard Exponential Moving Averages introduce lag because they are computed as a weighted average of historical prices — the more historical data used, the smoother but more delayed the result. The Zero-Lag EMA, introduced by John Ehlers and popularized in his work on digital signal processing for trading, addresses this by estimating the lag component and subtracting it from the EMA. The lag estimate is computed as the difference between the current EMA and the prior bar's EMA: `lag = EMA(bar 0) − EMA(bar 1)`. Adding this lag back to the current EMA produces an output that tracks price more closely and turns earlier than a standard EMA of the same period. This makes ZLEMA crossovers more timely than standard EMA crossovers — firing closer to the actual price inflection point rather than lagging behind it. The flattening detector adds a second dimension: when the ZLEMA's cycle-to-cycle change falls below `Point × 10`, momentum is decelerating even if no crossover has yet occurred — providing an early warning of a potential trend change before the crossover confirms it.

---

## Features

- **Native ZLEMA construction** — `CalculateZLEMA()` fetches `ema1 = iMA(..., MODE_EMA, PRICE_CLOSE, 0)` and `ema2 = iMA(..., MODE_EMA, PRICE_CLOSE, 1)`; computes `lag = ema1 − ema2`; returns `ema1 + lag` — a direct implementation of Ehlers' lag-correction formula
- **`iBars() < period` sufficient-data guard** — `CalculateZLEMA()` returns `0.0` if insufficient bars are available, preventing invalid data access during terminal warm-up
- **Price-ZLEMA crossover detection** — `AlertCrossovers` gate with `PrevZLEMA != 0` first-cycle guard; `PrevPrice < PrevZLEMA && currentPrice > currentZLEMA` → **Bullish Crossover**; `PrevPrice > PrevZLEMA && currentPrice < currentZLEMA` → **Bearish Crossover**
- **Flattening detection** — `AlertFlattening` gate; `MathAbs(currentZLEMA − PrevZLEMA) < Point × 10` → **Flattening Trend Detected** — fires when the ZLEMA has barely moved between cycles, signaling momentum exhaustion
- **Independent alert gating** — `AlertCrossovers` and `AlertFlattening` boolean inputs evaluated in separate `if` blocks; both active simultaneously without interference
- **Persistent dual-state tracking** — `PrevZLEMA` and `PrevPrice` globals updated unconditionally at cycle end
- **Three notification channels:** sound alert, email, and mobile push
- **Lightweight loop** — polls once per minute (`Sleep(60000)`)

---

## How It Works

1. Every minute, `CalculateZLEMA()` fetches two EMA values and returns `ema1 + (ema1 − ema2)`; `currentPrice = iClose(..., 0)` fetched
2. If `AlertCrossovers && PrevZLEMA != 0`: prior/current price-ZLEMA side comparison evaluated for crossover
3. If `AlertFlattening && PrevZLEMA != 0`: `MathAbs(currentZLEMA − PrevZLEMA) < Point × 10` evaluated for momentum exhaustion
4. `PrevZLEMA = currentZLEMA` and `PrevPrice = currentPrice` updated at cycle end

---

## Input Parameters

| Parameter           | Type            | Default     | Description                                                     |
|---------------------|-----------------|-------------|-----------------------------------------------------------------|
| `TradeSymbol`       | string          | `EURUSD`    | Symbol for analysis                                             |
| `Timeframe`         | ENUM_TIMEFRAMES | `PERIOD_H1` | Timeframe for analysis                                          |
| `ZLEMAPeriod`       | int             | `21`        | Period for the Zero-Lag EMA calculation                         |
| `AlertCrossovers`   | bool            | `true`      | Fire alerts on price-ZLEMA crossovers                           |
| `AlertFlattening`   | bool            | `true`      | Fire alerts when ZLEMA change falls below `Point × 10`          |
| `EnableAlerts`      | bool            | `true`      | Fire an on-screen/sound alert                                   |
| `EnableEmail`       | bool            | `false`     | Send an email notification                                      |
| `EnablePush`        | bool            | `false`     | Send a mobile push notification                                 |

---

## Alert Message Format

```
Bullish Crossover Detected detected on EURUSD (Timeframe: PERIOD_H1)
ZLEMA Value: 1.08392
```

---

## Installation

1. Copy `Zero-Lag_EMA_001.mq4` to `MQL4/Scripts/` in your MT4 data folder
2. Compile in MetaEditor (F7)
3. Drag onto any chart from Navigator → Scripts
4. Configure inputs and click **OK**

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
