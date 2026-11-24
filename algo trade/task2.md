## Review
### V3.2 Technical Code & Quantitative Business Audit (`option-trailing-algo`)

Reviewing your four V3.2 files (`v3_momentum_engine.py`, `broker_adapter.py`, `strike_screener.py`, `config.py`) against live Indian F&O execution realities surfaces **five concrete findings** ranked by severity—including one critical mid-candle state bug when OTM rotation fires right before a 3-minute boundary.

---

### Track 1: Technical Code & Reliability Findings

#### 1. [HIGH] Mid-Candle OTM Rotation State Drift (`v3_momentum_engine.py`)

* **Issue:** During `10:30–11:30 AM` (`BASE OBSERVE`), `v3_momentum_engine.py` polls only `active_sym` (`symbols_to_poll = [active_sym]`). When `active_sym` exceeds `max_prem` and rotates to `cand_s` at, say, `10:48 AM`, `cand_s` enters with `candles = []` and `base_high = None`—missing the entire `10:30–10:48 AM` Freeze-Hour range. Worse, if `active_sym` rotates *after* `11:30 AM` while flat, `cand_s` has `len(st["candles"]) == 0 <= MIN_OBSERVE_CANDLES (4)`, which forces it back into a 12-minute `BASE OBSERVE` warmup using post-11:30 price action instead of the true `10:30–11:30 AM` Freeze-Hour range.
* **Fix:** Keep `MIN_OBSERVE_CANDLES = 4` only when the script itself is launched after `11:18 AM`; for strikes rotated into mid-session, initialize their `base_high` / `base_low` from the 3-minute historical candles (`client.history()`) covering `10:30–11:30 AM` so every strike shares the exact same structural Freeze-Hour reference.

#### 2. [MEDIUM] Hardcoded API Key in Tracked `config.py`

* **Issue:** `OPENALGO_API_KEY` lives directly inside `config.py`, which is tracked by Git (`git add config.py`). Even on a private GitHub repository, committing live broker gateway keys creates accidental-leakage risk if you ever share logs, screenshots, or make the repo public for portfolio showcase.
* **Fix:** Move `OPENALGO_API_KEY` into a gitignored `.env` or `config_local.py` (`OPENALGO_API_KEY = os.getenv("OPENALGO_API_KEY", ...)`).

#### 3. [MEDIUM] Order Fill Price Estimation Lag (`broker_adapter.py`)

* **Issue:** In `BrokerAdapter.place_market_order()`, `fill_est` is read from `self.get_quote()` immediately after `self.client.placeorder()`. On fast breakout candles, ask price can jump `1–3 pts` (`₹20–₹195`) during order routing, causing `entry_price` (and therefore `stop_loss` and `breakeven_trigger`) in the engine to diverge slightly from the actual OpenAlgo/Fyers fill.
* **Fix:** Query `self.client.orderbook()` / `positionbook()` once `0.3s` after `placeorder()` to read the exact executed `average_price` from OpenAlgo, falling back to `ask`/`bid` only if `average_price == 0`.

#### 4. [LOW] First-Tick WebSocket Warmup Race (`broker_adapter.py`)

* **Issue:** `ensure_subscribed()` sleeps `0.4s` after `subscribe_quote()`. If Fyers' first tick takes `0.6s` to arrive, `get_quotes()` falls through once to the REST `self.client.quotes()` fallback on startup. This is safe (only 1 REST call), though bumping the initial subscription settle delay to `0.8s` eliminates even that single startup REST call.

---

### Track 2: Quantitative Business & Risk Logic Findings

#### 1. [HIGH] `₹4,000–₹5,000/lot` Band Tightness on Expiry vs. Non-Expiry Days

* **Strength:** Holding capital deployed per lot constant at `₹4,000–₹5,000` (`₹200–₹250` on SENSEX `20 qty`; `₹61.55–₹76.90` on NIFTY `65 qty`) normalizes your `₹500` max risk to an identical **10%–12.5% structural SL** across both indices—making forward-test expectancy cleanly comparable.
* **Risk Watch:** A `₹50` premium window on SENSEX (`200–250`) and `₹15.35` window on NIFTY (`61.55–76.90`) is narrow (`~1–2 strikes` wide). On **expiry days** (Tuesday for NIFTY, Thursday for SENSEX), ATM/near-OTM options inside `₹4K–₹5K/lot` carry extreme gamma and theta acceleration—a single 3-minute wick can swing `15–20 pts` (`₹300–₹400`) and clip your `25 pt` / `7.7 pt` SL before trending.
* **Rule Adjustment:** Keep `₹4,000–₹5,000/lot` on **non-expiry days** (`Mon, Wed, Fri`), and on **expiry days** either skip 0-DTE contracts (trade next-week expiry) or widen the SL buffer slightly while cutting `MAX_TRADES_PER_DAY` from `3` to `2`.

#### 2. [MEDIUM] Gap Between `1:1.5 R:R` Breakeven and `+50%` Stage 2 Surge

* **Math Check:** On a `₹220` SENSEX option with a `25 pt` SL (`₹500` risk):
* **Stage 1 (`1.5R` Breakeven)** triggers at `₹220 + 37.5 = ₹257.50` (`+17%`).
* **Stage 2 (`+50%` Surge)** does not trigger until `₹330.00` (`+50%`, or `+4.4R`).
* If a trade rallies to `₹315` (`+43%`, `+₹1,900` unrealized) and reverses sharply, Stage 1 leaves the SL all the way down at cost (`₹220`), giving back the entire `+₹1,900` open gain.


* **Refinement:** Add an intermediate **Stage 1.5 (`2.5R` / `+30%` move)** that locks **`+1R` (`+₹500` net profit floor)** at `₹282.50` (`+28%`), bridging the gap between breakeven and the `+50%` trail.

#### 3. [STRENGTH] What Is Already Working Exceptionally Well

* **Zero-`429` WebSocket + Fail-Safe Circuit Breaker:** Eliminates yesterday's Fyers `/data/depth` rate-limit freeze while guaranteeing an automatic market exit if ticks ever stall for `> 20s`.
* **5% Chase Guard + Spread Filter:** Prevents buying exhausted vertical spikes and illiquid wide-spread strikes—the two biggest killers of retail option buyers.

---

### Recommended Next Step

You can save those exact 3 refinements (`next-week expiry on 0-DTE`, `2.5R +1R profit lock`, and `config_local.py` secret separation) for your next minor patch (`V3.3`) after running tomorrow's live market session on V3.2.

Now that V3.2 is audited and pushed, would you like to proceed with building **`equity_intraday_engine.py`** (NSE Cash Intraday Buy/Sell engine with VWAP, Volume Surge, `10:30–11:30` Consolidation Box, and `₹400` fixed-rupee risk sizing)?
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
