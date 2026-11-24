Here is a structured, ready-to-paste prompt you can assign directly to **Jules** on your `option-trailing-algo` repository (`main` branch). It directs Jules to perform both a rigorous **Technical Code Audit** and a **Quantitative Business & Strategy Logic Review** without modifying production code prematurely (generating an audit report first so you stay in control of merges).

---

### Copy-Paste Task Prompt for Jules (`option-trailing-algo`)

```markdown
Perform a comprehensive **Technical Code Review** and **Business/Trading Logic Audit** of the `option-trailing-algo` repository (specifically the V3.2 build across `v3_momentum_engine.py`, `broker_adapter.py`, `strike_screener.py`, and `config.py`).

Do NOT modify core trading logic yet—create a detailed markdown audit report at `docs/V3_2_JULES_AUDIT_REPORT.md` (and open a PR or summary) covering the following two tracks:

### Track 1: Technical Code & Reliability Review
1. **WebSocket & REST Fallback (`broker_adapter.py`):**
   - Audit `BrokerAdapter.get_quote()` and `ensure_subscribed()` for thread-safety, race conditions on first-tick subscription (`time.sleep(0.4)`), and handling of partial/stale WebSocket payloads (`bid`/`ask` missing or zero).
   - Verify that fallback to `self.client.quotes()` cannot trigger Fyers `429 Too Many Requests` rate-limit loops during OTM strike rotation.
2. **State Machine & Edge Cases (`v3_momentum_engine.py`):**
   - Check 3-minute wall-clock candle aggregation (`current_candle_bucket()`), mid-candle OTM strike rotation (ensure ticks/candles from the outgoing strike do not pollute the incoming strike's `base_high`/`base_low` or `9 EMA`), and `SIMULATION` vs `OPENALGO` mode parity.
   - Audit the Stale-Feed Circuit Breaker (`MAX_QUOTE_FAILURES = 10`), order fill price estimation (`fill_price` fallback when `get_quote()` returns `None` immediately after `placeorder()`), and CSV logging atomicity (`v3_paper_trades_log.csv`).
3. **SQLite Screener & Security (`strike_screener.py` & `config.py`):**
   - Review SQLite read-only safety (`OPENALGO_DB`), expiry date parsing across weekly/monthly SENSEX (`BFO`) and NIFTY (`NFO`) contracts, and confirm no sensitive credentials (`OPENALGO_API_KEY`) are hardcoded or exposed in tracked files (recommend `.env` / `config_local.py` gitignore pattern if needed).

### Track 2: Quantitative Business & Risk Logic Review
1. **Capital-Normalized Position Sizing (`₹4,000 – ₹5,000` per lot):**
   - Evaluate the `min_prem = 4000 / qty` to `max_prem = 5000 / qty` band (`₹200–₹250` on SENSEX 20 qty; `₹61.55–₹76.90` on NIFTY 65 qty) paired with the `₹500` max risk per trade (`10%–12.5%` structural SL). Identify any liquidity, theta-decay, or gap-slippage risks on expiry vs. non-expiry days.
2. **Entry & 4-Stage Trailing Exit Math:**
   - Audit the `10:30–11:30 AM` Freeze-Hour Base Observation + `11:30 AM–01:30 PM` breakout window, `5%` Chase Guard, `1:1.5 R:R` Breakeven trigger, `+50%` surge -> `+25%` floor + 2-candle trailing SL, `9 EMA` momentum exit, and Climax Wick logic for potential premature stop-outs or missed runners.
3. **Actionable Recommendations:**
   - Rank findings by severity (**Critical**, **High**, **Medium**, **Low**) and provide concrete, minimal-diff code snippets for any recommended hardening.

```

---

### Quick Tip When Assigning in Jules

* Point Jules to your **`option-trailing-algo`** repository on branch **`main`**.
* Let Jules generate `docs/V3_2_JULES_AUDIT_REPORT.md` in the background while we build **`equity_intraday_engine.py`**—that way you get an independent second-pair-of-eyes audit on your options engine in parallel!

Whenever you've kicked off the Jules task, let me know and we'll jump straight into building **`equity_intraday_engine.py`**.
