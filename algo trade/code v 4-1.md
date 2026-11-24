Ran check git branch and status
Ran list directory files
Analyzed V3_2_JULES_AUDIT_REPORT.md:L1-150
Analyzed V3_2_JULES_AUDIT_REPORT.md:L151-300

Error: API error (attempt 1): UNAVAILABLE (code 503): No capacity available for model gemini-3.8-flash-high on the server (Error ID: 4c9b45f6-1365-4284-a6e0-8897fa61b7d9-10-2010)

Analyzed config.py:L1-42
Analyzed broker_adapter.py:L1-150
Analyzed broker_adapter.py:L151-217
Analyzed v3_momentum_engine.py:L1-120
Analyzed v3_momentum_engine.py:L121-250
Analyzed strike_screener.py:L1-100
Ran check python environment · Exited with code 1
Checked task Check python environment
Ran check wsl python3 · Exited with code 1

Error: API error (attempt 1): UNAVAILABLE (code 503): No capacity available for model gemini-3.8-flash-high on the server (Error ID: 4c9b45f6-1365-4284-a6e0-8897fa61b7d9-30-2010)

Ran find python on Windows · Exited with code 1
Ran check Python install paths
Ran test Python312 binary
Ran check pip list in Python312
Ran check pip list in Python310 and Python313 · Exited with code 1
Ran check venv folder
Ran check venv site-packages
Ran test Python312 with PYTHONPATH · Exited with code 1
Ran test py_compile on existing files
Ran test BrokerAdapter in SIMULATION mode
Ran inspect openalgo package structure

Error: API error (attempt 1): UNAVAILABLE (code 503): No capacity available for model gemini-3.8-flash-high on the server (Error ID: 4c9b45f6-1365-4284-a6e0-8897fa61b7d9-53-2010)

Ran check methods in data.py
Analyzed data.py:L85-160
Analyzed data.py:L248-310
Ran search multiquotes occurrences in openalgo
Edited config.py
Ran verify config.py compilation
Analyzed broker_adapter.py:L55-148
Analyzed broker_adapter.py:L1-60
Analyzed broker_adapter.py:L140-217
Edited broker_adapter.py
Analyzed broker_adapter.py:L120-230
Edited broker_adapter.py
Analyzed broker_adapter.py:L380-440
Analyzed broker_adapter.py:L470-546
Edited broker_adapter.py
Ran verify broker_adapter.py compilation
Ran test BrokerAdapter simulation methods
Created equity_screener.py
Ran compile equity_screener.py
Ran test equity_screener in BULLISH mode
Ran test equity_screener in BEARISH mode
Ran test equity_screener in SIDEWAYS mode
Created equity_intraday_engine.py
Ran compile all 6 modules
Canceled run equity engine simulation LONG scenario
Checked task Run equity engine simulation LONG scenario
Killed task Run equity engine simulation LONG scenario
Analyzed task-115.log:L1-31
Analyzed broker_adapter.py:L15-100
Analyzed broker_adapter.py:L190-235

Error: API error (attempt 1): UNAVAILABLE (code 503): No capacity available for model gemini-3.8-flash-high on the server (Error ID: 4c9b45f6-1365-4284-a6e0-8897fa61b7d9-127-2010)

Analyzed broker_adapter.py:L165-200
Analyzed broker_adapter.py:L270-320
Editing broker_adapter.py
Analyzed broker_adapter.py:L15-40
Analyzed broker_adapter.py:L80-120
Edited broker_adapter.py
Analyzed broker_adapter.py:L135-220

Error: API error (attempt 1): UNAVAILABLE (code 503): No capacity available for model gemini-3.8-flash-high on the server (Error ID: 4c9b45f6-1365-4284-a6e0-8897fa61b7d9-142-2010)

Error: API error (attempt 2): UNAVAILABLE (code 503): No capacity available for model gemini-3.8-flash-high on the server (Error ID: 4c9b45f6-1365-4284-a6e0-8897fa61b7d9-143-2010)

Analyzed broker_adapter.py:L265-300
Edited broker_adapter.py
Edited broker_adapter.py
Analyzed equity_intraday_engine.py:L330-380
Analyzed equity_intraday_engine.py:L410-450
Edited equity_intraday_engine.py
Analyzed equity_intraday_engine.py:L460-480

Error: API error (attempt 1): UNAVAILABLE (code 503): No capacity available for model gemini-3.8-flash-high on the server (Error ID: 4c9b45f6-1365-4284-a6e0-8897fa61b7d9-158-2010)

Edited equity_intraday_engine.py
Ran compile equity_intraday_engine.py
Ran execute LONG simulation test
Analyzed equity_paper_trades_log.csv:L1-2
Ran execute SHORT simulation test
Ran test daily limit kill switch on restart

Error: API error (attempt 1): UNAVAILABLE (code 503): No capacity available for model gemini-3.8-flash-high on the server (Error ID: 4c9b45f6-1365-4284-a6e0-8897fa61b7d9-171-2010)

Error: API error (attempt 2): UNAVAILABLE (code 503): No capacity available for model gemini-3.8-flash-high on the server (Error ID: 4c9b45f6-1365-4284-a6e0-8897fa61b7d9-172-2010)

Error: API error (attempt 3): UNAVAILABLE (code 503): No capacity available for model gemini-3.8-flash-high on the server (Error ID: 4c9b45f6-1365-4284-a6e0-8897fa61b7d9-173-2010)

Error: API error (attempt 4): UNAVAILABLE (code 503): No capacity available for model gemini-3.8-flash-high on the server (Error ID: 4c9b45f6-1365-4284-a6e0-8897fa61b7d9-174-2010)

Analyzed equity_paper_trades_log.csv:L1-3
Ran re-compile all files
Ran check git status
Analyzed .gitignore:L1-6
Ran check git diff config.py

Error: API error (attempt 1): UNAVAILABLE (code 503): No capacity available for model gemini-3.8-flash-high on the server (Error ID: 4c9b45f6-1365-4284-a6e0-8897fa61b7d9-185-2010)

Ran check git diff broker_adapter.py

### Summary of Deliverables & Implementation on `dev`

The companion **NSE Cash Equity Intraday Momentum Engine (V1.0)** has been implemented inside the `option-trailing-algo` repository on branch `dev`, adhering to all V3.4 hardening standards established in Jules' audit report (`docs/V3_2_JULES_AUDIT_REPORT.md`), with zero breakage of existing Options V3.4 files (`v3_momentum_engine.py`, `strike_screener.py`).

---

### Files Created & Modified

1. **[`config.py`](file:///wsl.localhost/Ubuntu/home/zahsay/trading-projects/option-trailing-algo/config.py#L43-L60)** *(Updated)*:
   - Appended Cash Equity constants safely with environment variable defaults:
     - `EQUITY_EXCHANGE = "NSE"`
     - `SENSEX_30_SYMBOLS` (complete 30 constituent symbols: `HDFCBANK`, `ICICIBANK`, `RELIANCE`, `BHARTIARTL`, `LT`, `SBIN`, `INFY`, `AXISBANK`, `KOTAKBANK`, `BAJFINANCE`, `M&M`, `ITC`, `ETERNAL`, `TCS`, `SUNPHARMA`, `TITAN`, `HINDUNILVR`, `TATASTEEL`, `MARUTI`, `NTPC`, `ADANIPORTS`, `BEL`, `HCLTECH`, `ULTRACEMCO`, `POWERGRID`, `INDIGO`, `ASIANPAINT`, `BAJAJFINSV`, `TECHM`, `TRENT`)
     - `EQUITY_MAX_RISK_PER_TRADE = 400.0` (Strict ₹400 risk cap)
     - `EQUITY_MAX_NOTIONAL = 25000.0` (~₹5,000 MIS margin cap at 5x leverage)
     - `EQUITY_MAX_TRADES_PER_DAY = 2`
     - `EQUITY_MAX_DAILY_LOSS = -800.0`
     - `EQUITY_ESTIMATED_CHARGES = 35.0`
     - `MARKET_TREND_THRESHOLD_PCT = 0.30`
     - `STOCK_MIN_MOVE_PCT = 0.80`
   - Preserved all existing V3.4 Options configuration keys and expiry settings.

2. **[`broker_adapter.py`](file:///wsl.localhost/Ubuntu/home/zahsay/trading-projects/option-trailing-algo/broker_adapter.py)** *(Updated)*:
   - Extended `_parse_depth_payload()` to extract `prev_close`, `open`, `volume`, `high`, `low` while maintaining strict zero-spread guardrails (`spread = 999.0` when depth is uninitialized/illiquid).
   - Added `get_multiquotes(symbols, exchange)` with safe 10-symbol chunking, single REST fallback, and deterministic `SIMULATION` mock support.
   - Added `fetch_intraday_history(symbol, exchange, interval)` supporting both pandas DataFrame and dictionary payloads from OpenAlgo's `client.history()`, alongside synthetic 3m candle generation from `09:15 AM` in simulation mode.
   - Added optional `strategy="V3_Momentum_Engine"` parameter to `place_market_order()`, enabling the equity engine to pass `strategy="V1_Equity_Intraday_Engine"` while retaining backwards compatibility.
   - Built dual-direction equity simulation sequences for `BULLISH` and `BEARISH` regimes without disturbing Options V3.4 `sim_prices`.

3. **[`equity_screener.py`](file:///wsl.localhost/Ubuntu/home/zahsay/trading-projects/option-trailing-algo/equity_screener.py)** *(New Standalone + Importable Module)*:
   - Implemented `select_trending_sensex_stocks(broker)`:
     - **Market Regime**: Reads SENSEX spot quote on BSE. Classifies day as `BULLISH` (`>= +0.30%`), `BEARISH` (`<= -0.30%`), or `SIDEWAYS` (recommends standing down).
     - **Universe Screening**: Scans all 30 Sensex constituents on NSE via `broker.get_multiquotes()`.
     - **Depth & Quality Filters**: Validates `bid > 0 and ask > 0`, tight spread (`<= 0.10%` of LTP), and price `<= ₹5,000` (ensuring at least 5 shares fit inside ₹25,000 notional).
     - **RVOL & Scoring**: Computes time-of-day weighted Relative Volume vs constituent baseline (or turnover rank fallback) and ranks Top 3 Gainers / Losers by composite `abs(pct_change) * RVOL` score.
     - **Console Output**: Renders clean ASCII ranking table (`SYMBOL | LTP | % CHG | VOLUME | SPREAD % | DIR | SCORE`).

4. **[`equity_intraday_engine.py`](file:///wsl.localhost/Ubuntu/home/zahsay/trading-projects/option-trailing-algo/equity_intraday_engine.py)** *(New Dual-Direction Engine)*:
   - **Candle State Machine**: Strict 3-minute wall-clock cadence (`CANDLE_SEC = 180`) with startup truncated-candle discard on live execution.
   - **Session VWAP & Volume Surge**: Hydrates cumulative $\frac{\sum (\text{Typical Price} \times \text{Volume})}{\sum \text{Volume}}$ from 09:15 AM; enforces `Volume >= 1.5x` rolling 10-candle average.
   - **Dual-Direction Entries (11:30 AM – 01:30 PM IST)**:
     - `LONG`: `Close > Base High` AND `Close > VWAP`, `Close <= Base High * 1.015`, tight spread `<= 0.10%`. Stop Loss: `max(Base Low, Entry * 0.992)`.
     - `SHORT`: `Close < Base Low` AND `Close < VWAP`, `Close >= Base Low * 0.985`, tight spread `<= 0.10%`. Stop Loss: `min(Base High, Entry * 1.008)`.
   - **Position Sizing**: Strictly capped at ₹400 fixed-rupee risk and ₹25,000 max notional (`qty = max(1, min(raw_qty, max_notional_qty))`).
   - **Symmetrical Multi-Stage Trailing Exit**:
     - *Stage 1 (+1.5R)*: Locks cost-adjusted breakeven stop (`entry ± charges_buffer`).
     - *Stage 2 (+2.5R)*: Locks `+1.0R` profit floor and activates 2-candle swing trailing + 9 EMA trend-break exit.
     - *Hard Target (+4.0R)*: Immediate market exit.
     - *Stale-Feed Circuit Breaker*: 10 consecutive failures triggers verified market exit; `in_trade` is only cleared on confirmed order execution.
     - *EOD MIS Square-Off*: At 03:10 PM IST (15:10).
   - **Audit Ledger**: Atomic persistence with `flush()` and `os.fsync()` to `equity_paper_trades_log.csv` (12 columns) with Excel `PermissionError` retries and startup daily loss/trade count recovery.

---

### Verification Results

#### 1. Compilation Check
```bash
python3 -m py_compile config.py broker_adapter.py equity_screener.py equity_intraday_engine.py v3_momentum_engine.py strike_screener.py
```
- **Exit Code:** `0` (All 6 modules compile without syntax or import errors).

#### 2. End-to-End Simulation Tests

##### A. LONG Scenario (`SIM_REGIME=BULLISH`)
```
[MARKET REGIME] SENSEX Index: 83500.00 | Prev Close: 83100.00 | Move: +0.48% (BULLISH)
[SCREENER] Selected Candidates (LONG): ['TCS', 'INFY', 'RELIANCE']
[HYDRATION] TCS: 45 3m candles | Session VWAP: ₹3510.78 | Base: [₹3500.00 - ₹3520.00]
[LONG SIGNAL] Breakout Close: ₹3525.00 > Base High ₹3520.00 & VWAP ₹3511.27
[SIM ORDER] BUY 7 TCS @ ₹3535.10 | Notional: ₹24,675.00 | Planned Risk: ₹175.00
[STAGE 1 HIT] +1.5R reached -> SL moved to breakeven: ₹3540.15
[STAGE 2 HIT] +2.5R reached -> Locked +1.0R floor: ₹3563.38 | Swing trail & 9 EMA active
[SWING TRAIL] Tightened SL: ₹3572.00 -> ₹3600.00 -> ₹3628.00
[EXIT CONFIRMED] SELL 7 TCS @ ₹3627.90 | Gross P&L: ₹649.60 | Net P&L: ₹614.60
```

##### B. SHORT Scenario & Startup Recovery (`SIM_REGIME=BEARISH`)
```
[RESUME RECOVERY] 1/2 trades executed today (P&L: ₹614.60) recovered from CSV log
[MARKET REGIME] SENSEX Index: 82600.00 | Prev Close: 83100.00 | Move: -0.60% (BEARISH)
[SCREENER] Selected Candidates (SHORT): ['TCS', 'INFY', 'RELIANCE']
[SHORT SIGNAL] Breakdown Close: ₹3495.00 < Base Low ₹3500.00 & VWAP ₹3510.18
[SIM ORDER] SELL 7 TCS @ ₹3484.90 | Notional: ₹24,465.00 | Planned Risk: ₹175.00
[STAGE 1 HIT] +1.5R reached -> SL moved to breakeven: ₹3479.85
[STAGE 2 HIT] +2.5R reached -> Locked +1.0R floor: ₹3457.02 | Swing trail active
[SWING TRAIL] Tightened SL: ₹3445.00 -> ₹3415.00 -> ₹3390.00
[EXIT CONFIRMED] BUY 7 TCS @ ₹3390.10 | Gross P&L: ₹663.60 | Net P&L: ₹628.60 | Daily: ₹1243.20
[ENGINE TERMINATED] Daily realized P&L: ₹1243.20 (2/2 trades executed)
```

##### C. Restart Kill-Switch Protection
Attempting a 3rd execution immediately tripped the guardrail:
```
[DAILY LIMIT REACHED] 2/2 trades executed today. Halting.
```

##### D. CSV Ledger Verification ([`equity_paper_trades_log.csv`](file:///wsl.localhost/Ubuntu/home/zahsay/trading-projects/option-trailing-algo/equity_paper_trades_log.csv))
```csv
Timestamp,Symbol,Direction,Qty,Entry_Price,Exit_Price,Stop_Loss,Highest_Stage,Exit_Reason,Gross_PnL,Net_PnL,Cumulative_Daily_PnL
2026-11-24 11:54:23,TCS,LONG,7,3535.1,3627.9,3628.0,STAGE_2,PROFIT_LOCK_1R,649.6,614.6,614.6
2026-11-24 11:54:41,TCS,SHORT,7,3484.9,3390.1,3390.0,STAGE_2,PROFIT_LOCK_1R,663.6,628.6,1243.2
```

---

### OpenAlgo Assumptions Verified

1. **`multiquotes()`**:
   - Signature: `api.multiquotes(symbols=[{"symbol": str, "exchange": str}, ...])`.
   - Returns: `{"status": "success", "data": [ {"symbol": ..., "ltp": ..., "bid": ..., "ask": ..., "volume": ...}, ... ]}`.
   - Guarded with safe 10-symbol batching and individual REST quote fallback if any batch fails.
2. **`history()`**:
   - Signature: `api.history(symbol=..., exchange=..., interval='3m', start_date='YYYY-MM-DD', end_date='YYYY-MM-DD')`.
   - Returns either a pandas DataFrame (when pandas is present in runtime) or a raw dictionary containing `"data"`. The adapter normalizes both transparently to standard Python dictionary rows with `timestamp`, `open`, `high`, `low`, `close`, and `volume`.
