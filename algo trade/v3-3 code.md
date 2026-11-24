Yes—locking **V3.3** in right now while the market is closed for Guru Nanak Jayanti means tomorrow morning (Wednesday, Nov 25) is pure 5-minute launch-and-watch with zero coding distraction.

### What V3.3 Upgrades Across Your 4 Files

1. **Mid-Rotation Historical Base Sync (`v3_momentum_engine.py`):** If OTM rotation switches into a new strike mid-session (`10:30–11:30 AM` or after `11:30 AM`), the engine automatically queries OpenAlgo's 3-minute historical candles (`client.history()`) for today's `10:30–11:30 AM` window so the incoming strike inherits the **exact Freeze-Hour `base_high` and `base_low**` instead of starting blind or resetting warmup.
2. **Stage 1.5 (`2.5R` -> `+1R` Profit Lock) (`v3_momentum_engine.py`):** Bridges the wide gap between `1.5R` Breakeven and `+50%` Surge—once a trade reaches **`+2.5R`** (`+₹1,250` open gain), SL locks at **`+1.0R` (`entry + initial_risk_pts`, `+₹500` guaranteed gross profit)** so big open winners never round-trip back to zero.
3. **Actual Executed Fill Price via `orderbook()` (`broker_adapter.py`):** Queries `client.orderbook()` `0.35s` after `placeorder()` to pull the real `average_price` fill from OpenAlgo instead of estimating from `ask`/`bid`, plus increases first-tick WebSocket settle time to `0.8s` to eliminate startup REST calls.
4. **0-DTE Expiry Day Guardrail (`strike_screener.py`):** Automatically detects if the nearest expiry is **today (`0-DTE`)**—such as NIFTY on Tuesdays or SENSEX on Thursdays—warns about extreme 0-DTE gamma/theta decay, and lets you select **Next-Week Expiry** with one keystroke (`Y/n`).
5. **Local Secret Separation (`config.py` + `.gitignore`):** Reads `OPENALGO_API_KEY` from `config_local.py` (gitignored) or environment variable so you never accidentally commit your live key to GitHub.

---

### Complete V3.3 Files (Direct Copy-Paste for VS Code)

#### 1. `config.py` (V3.3 — Safe Git-Tracked Config)

*(Create a new file `config_local.py` in the same folder with just `OPENALGO_API_KEY = "your-actual-key"`, and add `config_local.py` to `.gitignore`.)*

```python
import os

# --- EXECUTION MODE ---
# "SIMULATION" = Local dry-run with synthetic ticks (0 API calls)
# "OPENALGO"   = Live/Paper routing via local OpenAlgo server (Analyze Mode)
MODE = "OPENALGO"

# --- OPENALGO LOCAL GATEWAY ---
OPENALGO_HOST = "http://127.0.0.1:5000"
try:
  from config_local import OPENALGO_API_KEY
except ImportError:
  OPENALGO_API_KEY = os.getenv("OPENALGO_API_KEY", "YOUR_OPENALGO_API_KEY_HERE")

# --- DEFAULT CONTRACT & MULTI-STRIKE WATCHLIST ---
EXCHANGE = "BFO"
SYMBOL = "SENSEX26NOV2682500PE"
QTY = 20

# Ordered from closest OTM to deeper OTM for automatic premium rotation
CANDIDATE_STRIKES = [
    "SENSEX26NOV2682500PE",
    "SENSEX26NOV2682400PE",
    "SENSEX26NOV2682200PE",
    "SENSEX26NOV2682000PE",
]

# --- DAILY RISK & DISCIPLINE GUARDRAILS ---
MAX_TRADES_PER_DAY = 3
MAX_DAILY_LOSS = -800.0
ESTIMATED_CHARGES_PER_TRADE = 55.0

# --- V3.3 CAPITAL-NORMALIZED POSITION SIZING (₹4,000 - ₹5,000 per lot) ---
# Automatically derives index-agnostic premium thresholds:
#   SENSEX (20 qty) -> ₹200.00 - ₹250.00
#   NIFTY  (65 qty) -> ₹61.55  - ₹76.90
MIN_CAPITAL_PER_LOT = 4000.0
MAX_CAPITAL_PER_LOT = 5000.0

```

---

#### 2. `broker_adapter.py` (V3.3 — WebSocket Streaming + Exact `orderbook()` Fill Price + Historical Base Sync)

```python
import datetime
import time
import config


class BrokerAdapter:
  """Unified adapter supporting SIMULATION and OpenAlgo WebSocket + REST modes."""

  def __init__(self):
    self.mode = config.MODE.upper()
    self.client = None
    self.ws_connected = False
    self.subscribed = set()
    self.sim_step = 0
    self.sim_prices = [
        180.0, 182.0, 181.0, 184.0,  # Candle 1 (Observe)
        185.0, 188.0, 186.0, 189.0,  # Candle 2 (Observe)
        188.0, 190.0, 187.0, 189.5,  # Candle 3 (Observe)
        189.0, 191.0, 188.5, 190.0,  # Candle 4 (Observe -> Base High ~ 191.0, Base Low ~ 180.0)
        190.5, 192.0, 194.0, 196.5,  # Candle 5 (Breakout > 191.0 -> ENTRY @ ~196.5)
        198.0, 205.0, 212.0, 222.0,  # Candle 6 (+1.5R Breakeven)
        235.0, 250.0, 265.0, 280.0,  # Candle 7 (+2.5R -> +1R Profit Lock)
        295.0, 315.0, 335.0, 340.0,  # Candle 8 (+50% Surge -> Stage 2 Trail)
        335.0, 315.0, 295.0, 285.0,  # Candle 9 (Pullback -> Trailing SL / 9 EMA Exit)
    ]

    if self.mode == "OPENALGO":
      from openalgo import api

      self.client = api(
          api_key=config.OPENALGO_API_KEY, host=config.OPENALGO_HOST
      )
      try:
        self.client.connect()
        self.ws_connected = True
        print("[BROKER] OpenAlgo WebSocket connected (Zero-429 real-time stream).")
      except Exception as e:
        self.ws_connected = False
        print(f"[BROKER WARNING] WebSocket connect fallback to REST: {e}")

  def ensure_subscribed(self, symbol, exchange):
    if not self.ws_connected or (symbol, exchange) in self.subscribed:
      return
    try:
      self.client.subscribe_quote([{"symbol": symbol, "exchange": exchange}])
      self.subscribed.add((symbol, exchange))
      time.sleep(0.8)  # Allow initial WebSocket snapshot to populate
    except Exception:
      pass

  def get_quote(self, symbol, exchange):
    if self.mode == "SIMULATION":
      price = self.sim_prices[min(self.sim_step, len(self.sim_prices) - 1)]
      self.sim_step += 1
      return {
          "ltp": price,
          "bid": round(price - 0.15, 2),
          "ask": round(price + 0.15, 2),
          "spread": 0.30,
      }

    # 1. Primary: Zero-REST OpenAlgo WebSocket cache
    if self.ws_connected:
      self.ensure_subscribed(symbol, exchange)
      try:
        ws_q = self.client.get_quotes(symbol=symbol, exchange=exchange)
        data = ws_q.get("data", ws_q) if isinstance(ws_q, dict) else None
        if isinstance(data, dict) and float(data.get("ltp", 0)) > 0:
          ltp = float(data["ltp"])
          bid = float(data.get("bid", ltp))
          ask = float(data.get("ask", ltp))
          spread = round(max(0.0, ask - bid), 2)
          return {"ltp": ltp, "bid": bid, "ask": ask, "spread": spread}
      except Exception:
        pass

    # 2. Fallback: Single REST quote check (no retry storms on 429)
    try:
      quote = self.client.quotes(symbol=symbol, exchange=exchange)
      if isinstance(quote, dict) and quote.get("status") == "success" and "data" in quote:
        data = quote["data"]
        ltp = float(data["ltp"])
        bid = float(data.get("bid", ltp))
        ask = float(data.get("ask", ltp))
        spread = round(max(0.0, ask - bid), 2)
        return {"ltp": ltp, "bid": bid, "ask": ask, "spread": spread}
    except Exception as e:
      print(f"[QUOTE WARNING] {symbol}: {e}")

    return None

  def fetch_freeze_hour_base(self, symbol, exchange):
    """Backfills 10:30-11:30 AM 3m candles if OTM rotation switches into a new strike mid-session."""
    if self.mode == "SIMULATION" or not self.client:
      return None, None, []
    try:
      today_str = datetime.date.today().strftime("%Y-%m-%d")
      hist = self.client.history(
          symbol=symbol,
          exchange=exchange,
          interval="3m",
          start_date=today_str,
          end_date=today_str,
      )
      rows = hist.get("data", []) if isinstance(hist, dict) else []
      candles = []
      base_high = None
      base_low = None
      for r in rows:
        ts_str = str(r.get("timestamp", r.get("datetime", "")))
        h = float(r.get("high", 0))
        l = float(r.get("low", 0))
        o = float(r.get("open", 0))
        c = float(r.get("close", 0))
        if h <= 0 or l <= 0:
          continue
        candles.append({"open": o, "high": h, "low": l, "close": c})
        if "10:3" in ts_str or "10:4" in ts_str or "10:5" in ts_str or "11:0" in ts_str or "11:1" in ts_str or "11:2" in ts_str:
          base_high = h if base_high is None else max(base_high, h)
          base_low = l if base_low is None else min(base_low, l)
      return base_high, base_low, candles[-20:]
    except Exception:
      return None, None, []

  def _fetch_executed_price(self, orderid):
    """Queries OpenAlgo orderbook once to retrieve exact executed average_price."""
    if not orderid or orderid == "UNKNOWN" or not self.client:
      return 0.0
    try:
      time.sleep(0.35)
      ob = self.client.orderbook()
      orders = []
      if isinstance(ob, dict):
        data = ob.get("data", {})
        if isinstance(data, dict):
          orders = data.get("orders", [])
        elif isinstance(data, list):
          orders = data
      for o in orders:
        if str(o.get("orderid", "")) == str(orderid):
          avg_px = float(o.get("average_price", o.get("price", 0)) or 0)
          if avg_px > 0:
            return avg_px
    except Exception:
      pass
    return 0.0

  def place_market_order(self, symbol, exchange, action, qty):
    if self.mode == "SIMULATION":
      q = self.get_quote(symbol, exchange)
      fill = q["ask"] if action == "BUY" else q["bid"]
      print(f"[SIM ORDER] {action} {qty} {symbol} @ {fill:.2f}")
      return {"status": "success", "orderid": f"SIM-{int(time.time())}", "fill_price": fill}

    try:
      resp = self.client.placeorder(
          strategy="V3_Momentum_Engine",
          symbol=symbol,
          action=action,
          exchange=exchange,
          price_type="MARKET",
          product="MIS",
          quantity=str(qty),
      )
      orderid = resp.get("orderid", "UNKNOWN") if isinstance(resp, dict) else "UNKNOWN"
      fill_exact = self._fetch_executed_price(orderid)
      if fill_exact <= 0:
        q = self.get_quote(symbol, exchange)
        fill_exact = (q["ask"] if action == "BUY" else q["bid"]) if q else 0.0
      print(f"[OPENALGO ORDER] {action} {qty} {symbol} @ ₹{fill_exact:.2f} -> {resp}")
      return {
          "status": "success",
          "orderid": orderid,
          "fill_price": fill_exact,
      }
    except Exception as e:
      print(f"[ORDER ERROR] {action} {symbol} failed: {e}")
      return None

  def shutdown(self):
    if self.ws_connected and self.client:
      try:
        if self.subscribed:
          self.client.unsubscribe_quote(
              [{"symbol": s, "exchange": e} for s, e in self.subscribed]
          )
        self.client.disconnect()
      except Exception:
        pass

```

---

#### 3. `strike_screener.py` (V3.3 — SQLite Screener + `₹4K–₹5K/lot` Band + 0-DTE Expiry Guard)

```python
import datetime
import os
import sqlite3
import time
import config
from openalgo import api

OPENALGO_DB = os.path.expanduser("~/trading-projects/openalgo/db/openalgo.db")


def parse_expiry(exp_str):
  for fmt in ("%d-%b-%y", "%d-%b-%Y", "%Y-%m-%d"):
    try:
      return datetime.datetime.strptime(exp_str.strip(), fmt).date()
    except ValueError:
      continue
  return None


def screen_strikes():
  print("\n=== V3.3 INSTANT PRE-MARKET STRIKE SCREENER ===")
  idx_in = input("Index [SENSEX / NIFTY] (default SENSEX): ").strip().upper() or "SENSEX"
  opt_in = input("Direction [CE / PE] (default PE): ").strip().upper() or "PE"
  spot_str = input("Current Spot Level (e.g., 83300 for SENSEX, 24150 for NIFTY): ").strip()

  if not spot_str:
    print("[ERROR] Spot level is required to filter nearby OTM strikes.")
    return
  spot = float(spot_str)

  exchange = "BFO" if idx_in == "SENSEX" else "NFO"
  min_cap = getattr(config, "MIN_CAPITAL_PER_LOT", 4000.0)
  max_cap = getattr(config, "MAX_CAPITAL_PER_LOT", 5000.0)
  lot_preview = 20 if idx_in == "SENSEX" else 65

  if not os.path.isfile(OPENALGO_DB):
    print(f"[ERROR] OpenAlgo SQLite master database not found at {OPENALGO_DB}")
    return

  t0 = time.time()
  conn = sqlite3.connect(OPENALGO_DB)
  cur = conn.cursor()
  cur.execute(
      """
      SELECT symbol, expiry, strike, lotsize
      FROM symtoken
      WHERE exchange = ? AND name = ? AND instrumenttype = 'OPTIDX' AND symbol LIKE ?
      """,
      (exchange, idx_in, f"%{opt_in}"),
  )
  rows = cur.fetchall()
  conn.close()

  if not rows:
    print(f"[ERROR] No {idx_in} {opt_in} contracts found in OpenAlgo master DB.")
    return

  today = datetime.date.today()
  valid_rows = []
  for sym, exp_str, strike, lotsize in rows:
    exp_dt = parse_expiry(exp_str)
    if exp_dt and exp_dt >= today:
      valid_rows.append((exp_dt, exp_str, sym, float(strike), int(lotsize)))

  if not valid_rows:
    print("[ERROR] No active/upcoming expiries found.")
    return

  distinct_expiries = sorted(set(r[0] for r in valid_rows))
  selected_exp = distinct_expiries[0]

  # 0-DTE Expiry-Day Gamma/Theta Guardrail
  if selected_exp == today and len(distinct_expiries) > 1:
    next_exp = distinct_expiries[1]
    print(f"\n[0-DTE EXPIRY WARNING] Nearest expiry ({selected_exp}) is TODAY!")
    print("  0-DTE contracts carry extreme gamma/theta spikes against a fixed ₹500 SL.")
    choice = input(f"  Switch to Next-Week Expiry ({next_exp})? [Y/n]: ").strip().lower()
    if choice != "n":
      selected_exp = next_exp

  expiry_contracts = [r for r in valid_rows if r[0] == selected_exp]
  exp_str = expiry_contracts[0][1]

  # Pick 10 strikes starting near ATM into OTM
  if opt_in == "CE":
    otm = sorted([r for r in expiry_contracts if r[3] >= spot - 100], key=lambda x: x[3])[:10]
  else:
    otm = sorted([r for r in expiry_contracts if r[3] <= spot + 100], key=lambda x: x[3], reverse=True)[:10]

  lot_size = otm[0][4] if otm else lot_preview
  min_prem = round(min_cap / lot_size, 2)
  max_prem = round(max_cap / lot_size, 2)

  print(
      f"\n[DB LOOKUP: {time.time()-t0:.3f}s] Expiry: {exp_str} | Lot: {lot_size}"
      f" | Band (₹{min_cap:.0f}-₹{max_cap:.0f}/lot): ₹{min_prem:.2f}-₹{max_prem:.2f}"
      f" | Checking {len(otm)} strikes...\n",
      flush=True,
  )

  client = api(api_key=config.OPENALGO_API_KEY, host=config.OPENALGO_HOST)
  sweet_spot = []

  print(f"{'SYMBOL':<24} | {'STRIKE':<8} | {'LTP':<8} | {'CAPITAL/LOT':<11} | {'SPREAD':<7} | {'STATUS'}")
  print("-" * 82)

  for _, _, sym, strike, _ in otm:
    time.sleep(1.1)
    try:
      q = client.quotes(symbol=sym, exchange=exchange)
      if isinstance(q, dict) and q.get("status") == "success" and "data" in q:
        d = q["data"]
        ltp = float(d["ltp"])
        bid = float(d.get("bid", ltp))
        ask = float(d.get("ask", ltp))
        spread = round(max(0.0, ask - bid), 2)
        cap_req = round(ltp * lot_size, 0)

        if min_prem <= ltp <= max_prem and spread <= 1.00:
          status = "SWEET SPOT"
          sweet_spot.append(sym)
        elif ltp > max_prem:
          status = "EXPENSIVE (>₹5K)"
        elif ltp < min_prem:
          status = "DEEP OTM (<₹4K)"
        else:
          status = "WIDE SPREAD"

        print(
            f"{sym:<24} | {strike:<8.0f} | ₹{ltp:<7.2f} | ₹{cap_req:<10.0f} | ₹{spread:<6.2f} | {status}",
            flush=True,
        )
    except Exception as e:
      print(f"{sym:<24} | {strike:<8.0f} | ERROR: {e}", flush=True)

  print("\n" + "=" * 82)
  if sweet_spot:
    print("RECOMMENDED CANDIDATE_STRIKES FOR config.py:")
    print(f"CANDIDATE_STRIKES = {sweet_spot[:5]}")
  else:
    print(
        f"[NOTE] No strikes fell inside ₹{min_prem:.2f}–₹{max_prem:.2f}"
        f" (₹{min_cap:.0f}-₹{max_cap:.0f}/lot) with <= ₹1.00 spread (or market closed)."
    )
  print("=" * 82 + "\n")


if __name__ == "__main__":
  screen_strikes()

```

---

#### 4. `v3_momentum_engine.py` (V3.3 — Mid-Rotation Historical Base Sync + Stage 1.5 `+1R` Profit Lock)

```python
import csv
import datetime
import os
import time
from broker_adapter import BrokerAdapter
import config

LOG_FILE = "v3_paper_trades_log.csv"
CANDLE_SEC = 180             # Strict 3-minute wall-clock candles
MAX_RISK_PER_TRADE = 500.0   # Dynamic ₹500 max risk per trade (25 pts SENSEX / 7.7 pts NIFTY)
CHASE_GUARD_PCT = 0.05       # Max 5% breakout extension above Base High
MAX_SPREAD_INDEX = 1.00      # Max ₹1.00 bid-ask spread for index options
MAX_SPREAD_STOCK = 0.15      # Max ₹0.15 bid-ask spread for stock options
MIN_OBSERVE_CANDLES = 4      # Minimum 4 candles (12 mins) if started late
MAX_QUOTE_FAILURES = 10      # Circuit breaker threshold (~20-30s dead feed)


def init_csv():
  if not os.path.isfile(LOG_FILE):
    with open(LOG_FILE, mode="w", newline="") as f:
      writer = csv.writer(f)
      writer.writerow([
          "Timestamp",
          "Symbol",
          "Qty",
          "Entry_Price",
          "Exit_Price",
          "Highest_Price",
          "Highest_Stage",
          "Exit_Reason",
          "Gross_PnL",
          "Net_PnL",
          "Cumulative_Daily_PnL",
      ])


def log_trade(row):
  with open(LOG_FILE, mode="a", newline="") as f:
    csv.writer(f).writerow(row)


def compute_ema(closes, period=9):
  if not closes:
    return 0.0
  k = 2.0 / (period + 1)
  ema = closes[0]
  for c in closes[1:]:
    ema = (c * k) + (ema * (1.0 - k))
  return ema


def current_candle_bucket():
  now = datetime.datetime.now()
  return (now.hour * 3600 + now.minute * 60 + now.second) // CANDLE_SEC


def sync_strike_history(broker, state, sym, exchange):
  """Hydrates Freeze-Hour (10:30-11:30 AM) base_high/base_low on OTM rotation."""
  st = state[sym]
  if st["base_high"] is not None:
    return
  bh, bl, hist_candles = broker.fetch_freeze_hour_base(sym, exchange)
  if bh is not None and bl is not None:
    st["base_high"] = bh
    st["base_low"] = bl
    if hist_candles:
      st["candles"] = hist_candles
    print(
        f"[BASE SYNCED | {sym}] Hydrated Freeze-Hour Base from history:"
        f" [{bl:.2f} - {bh:.2f}] ({len(hist_candles)} candles)"
    )


def run_v3_engine():
  init_csv()
  broker = BrokerAdapter()

  default_list = getattr(config, "CANDIDATE_STRIKES", [config.SYMBOL])
  default_str = ",".join(default_list)

  print("\n=== V3.3 MULTI-STRIKE MOMENTUM ENGINE SETUP ===")
  sym_in = input(f"Candidate Strikes [{default_str}]: ").strip().upper()
  strikes = [s.strip() for s in sym_in.split(",") if s.strip()] if sym_in else default_list

  exch_in = input(f"Exchange [{config.EXCHANGE}]: ").strip().upper()
  exchange = exch_in if exch_in else config.EXCHANGE

  qty_in = input(f"Quantity [{config.QTY}]: ").strip()
  qty = int(qty_in) if qty_in else int(config.QTY)

  # Dynamic ₹4,000 - ₹5,000 capital-normalized premium band & ₹500 max risk SL cap
  min_cap = getattr(config, "MIN_CAPITAL_PER_LOT", 4000.0)
  max_cap = getattr(config, "MAX_CAPITAL_PER_LOT", 5000.0)
  min_prem = round(min_cap / qty, 2)
  max_prem = round(max_cap / qty, 2)
  max_sl_points = round(MAX_RISK_PER_TRADE / qty, 2)

  max_spread = (
      MAX_SPREAD_INDEX
      if any(k in strikes[0] for k in ("NIFTY", "SENSEX", "BANKNIFTY"))
      else MAX_SPREAD_STOCK
  )

  # Per-strike independent state for seamless OTM auto-rotation
  state = {
      s: {
          "ticks": [],
          "candles": [],
          "base_high": None,
          "base_low": None,
          "base_locked": False,
          "last_quote": None,
      }
      for s in strikes
  }

  active_idx = 0
  active_sym = strikes[active_idx]
  started_late = datetime.datetime.now().time() >= datetime.time(11, 18)

  in_trade = False
  entry_price = 0.0
  initial_risk_pts = 0.0
  breakeven_trigger = 0.0
  profit_lock_trigger = 0.0
  highest_price = 0.0
  stop_loss = 0.0
  stage = "IDLE"
  climax_high = None

  daily_pnl = 0.0
  trades_today = 0
  consecutive_quote_failures = 0
  max_trades = getattr(config, "MAX_TRADES_PER_DAY", 3)
  max_daily_loss = getattr(config, "MAX_DAILY_LOSS", -800.0)

  last_bucket = current_candle_bucket()
  sim_ticks_in_bucket = 0

  print(
      f"\n>>> Starting V3.3 Engine [{config.MODE}] | {len(strikes)} Strikes Watched"
      f" | Band (₹{min_cap:.0f}-₹{max_cap:.0f}/lot): ₹{min_prem:.2f}-₹{max_prem:.2f}"
      f" | Max SL: {max_sl_points:.2f} pts | Trades: {trades_today}/{max_trades} <<<"
  )

  try:
    while True:
      now = datetime.datetime.now()
      now_t = now.time()

      # 1. Daily Kill-Switch Guard
      if trades_today >= max_trades:
        print(f"\n[DAILY LIMIT REACHED] {trades_today}/{max_trades} trades executed today. Stopping.")
        break
      if daily_pnl <= max_daily_loss:
        print(f"\n[KILL SWITCH TRIGGERED] Daily P&L ₹{daily_pnl:.2f} <= ₹{max_daily_loss:.2f}. Halting.")
        break

      # 2. Time Windows
      if config.MODE == "SIMULATION":
        window_start = datetime.time(0, 0)
        window_end = datetime.time(23, 59)
        eod_exit_time = datetime.time(23, 59)
      else:
        window_start = datetime.time(11, 30)
        window_end = datetime.time(13, 30)
        eod_exit_time = datetime.time(15, 15)

      if not in_trade and now_t >= window_end:
        print(f"\n[WINDOW CLOSED] Past {window_end.strftime('%H:%M')} entry cutoff with no open position.")
        break

      # 3. Poll active strike (WebSocket primary + REST fallback + Stale-Feed Circuit Breaker)
      q = broker.get_quote(active_sym, exchange)
      if q:
        consecutive_quote_failures = 0
        state[active_sym]["last_quote"] = q
        state[active_sym]["ticks"].append(q["ltp"])
        if not in_trade and now_t < window_start:
          print(
              f"  [LIVE TICK] {active_sym} LTP: ₹{q['ltp']:.2f} | Spread: ₹{q['spread']:.2f}   ",
              end="\r",
              flush=True,
          )
      else:
        consecutive_quote_failures += 1
        if consecutive_quote_failures >= MAX_QUOTE_FAILURES:
          print(f"\n[CRITICAL FEED ALERT] {consecutive_quote_failures} consecutive quote failures on {active_sym}!")
          if in_trade:
            print("[FAIL-SAFE EXIT] Closing open position immediately due to broken market data feed.")
            exit_resp = broker.place_market_order(active_sym, exchange, "SELL", qty)
            last_ltp = (
                state[active_sym]["last_quote"]["ltp"]
                if state[active_sym]["last_quote"]
                else entry_price
            )
            exit_px = (
                exit_resp["fill_price"]
                if (exit_resp and exit_resp["fill_price"] > 0)
                else last_ltp
            )
            gross = round((exit_px - entry_price) * qty, 2)
            net = round(gross - getattr(config, "ESTIMATED_CHARGES_PER_TRADE", 55.0), 2)
            daily_pnl = round(daily_pnl + net, 2)
            trades_today += 1
            log_trade([
                now.strftime("%Y-%m-%d %H:%M:%S"),
                active_sym,
                qty,
                round(entry_price, 2),
                round(exit_px, 2),
                round(highest_price, 2),
                stage,
                "FEED_FAILURE_FAILSAFE",
                gross,
                net,
                daily_pnl,
            ])
            in_trade = False
            state[active_sym]["base_locked"] = False
            state[active_sym]["candles"].clear()
            state[active_sym]["base_high"] = None
            state[active_sym]["base_low"] = None
          consecutive_quote_failures = 0
        time.sleep(2.0)
        continue

      # 4. Dynamic OTM Strike Auto-Rotation (Only when flat)
      if not in_trade and len(strikes) > 1 and state[active_sym]["last_quote"]:
        curr_ltp = state[active_sym]["last_quote"]["ltp"]
        if curr_ltp > max_prem and active_idx + 1 < len(strikes):
          for next_i in range(active_idx + 1, len(strikes)):
            cand_s = strikes[next_i]
            time.sleep(0.5)
            cand_q = broker.get_quote(cand_s, exchange)
            state[cand_s]["last_quote"] = cand_q
            if cand_q and cand_q["ltp"] <= max_prem and cand_q["spread"] <= max_spread:
              print(
                  f"\n>>> [OTM ROTATE UP] {active_sym} LTP ₹{curr_ltp:.2f} > ₹{max_prem:.2f}."
                  f" Switching to {cand_s} @ ₹{cand_q['ltp']:.2f} <<<"
              )
              active_idx = next_i
              active_sym = cand_s
              sync_strike_history(broker, state, active_sym, exchange)
              break
        elif curr_ltp < min_prem and active_idx - 1 >= 0:
          for prev_i in range(active_idx - 1, -1, -1):
            cand_s = strikes[prev_i]
            time.sleep(0.5)
            cand_q = broker.get_quote(cand_s, exchange)
            state[cand_s]["last_quote"] = cand_q
            if (
                cand_q
                and min_prem <= cand_q["ltp"] <= max_prem
                and cand_q["spread"] <= max_spread
            ):
              print(
                  f"\n>>> [OTM ROTATE DOWN] {active_sym} LTP ₹{curr_ltp:.2f} < ₹{min_prem:.2f}."
                  f" Switching back to {cand_s} @ ₹{cand_q['ltp']:.2f} <<<"
              )
              active_idx = prev_i
              active_sym = cand_s
              sync_strike_history(broker, state, active_sym, exchange)
              break

      # 5. Intra-Candle Tick Exit Management (Instant SL / 1.5R BE / 2.5R +1R Lock / +50% Trail / 2x Target / EOD)
      if in_trade and state[active_sym]["last_quote"]:
        ltp = state[active_sym]["last_quote"]["ltp"]
        if ltp > highest_price:
          highest_price = ltp

        # Stage 1: 1:1.5 Risk-Reward Breakeven Lock
        if stage == "INITIAL_BASE_SL" and ltp >= breakeven_trigger:
          stop_loss = max(stop_loss, entry_price)
          stage = "STAGE1_1.5R_BREAKEVEN"
          print(
              f"\n[STAGE 1: 1.5R BREAKEVEN] {active_sym} hit ₹{ltp:.2f} (>= ₹{breakeven_trigger:.2f})."
              f" SL locked at Cost: ₹{stop_loss:.2f}"
          )

        # Stage 1.5: 1:2.5 Risk-Reward -> Lock +1R Guaranteed Profit Floor
        if stage in ("INITIAL_BASE_SL", "STAGE1_1.5R_BREAKEVEN") and ltp >= profit_lock_trigger:
          one_r_floor = round(entry_price + initial_risk_pts, 2)
          stop_loss = max(stop_loss, one_r_floor)
          stage = "STAGE1.5_2.5R_LOCK_1R"
          print(
              f"\n[STAGE 1.5: 2.5R PROFIT LOCK] {active_sym} hit ₹{ltp:.2f} (>= ₹{profit_lock_trigger:.2f})."
              f" SL locked at +1R: ₹{stop_loss:.2f}"
          )

        # Stage 2: +50% Profit Surge -> Lock +25% Floor
        if ltp >= entry_price * 1.50 and stage != "STAGE2_PROFIT_TRAIL":
          stop_loss = max(stop_loss, round(entry_price * 1.25, 2))
          stage = "STAGE2_PROFIT_TRAIL"
          print(
              f"\n[STAGE 2: +50% SURGE] {active_sym} hit ₹{ltp:.2f}."
              f" Floor SL locked at +25%: ₹{stop_loss:.2f}"
          )

        exit_reason = None
        if ltp <= stop_loss:
          exit_reason = f"STOP_LOSS ({stage})"
        elif ltp >= entry_price * 2.00:
          exit_reason = "TARGET_2X_DOUBLED"
        elif now_t >= eod_exit_time:
          exit_reason = "EOD_SQUARE_OFF"

        if exit_reason:
          exit_resp = broker.place_market_order(active_sym, exchange, "SELL", qty)
          exit_px = (
              exit_resp["fill_price"]
              if (exit_resp and exit_resp["fill_price"] > 0)
              else ltp
          )
          gross = round((exit_px - entry_price) * qty, 2)
          net = round(gross - getattr(config, "ESTIMATED_CHARGES_PER_TRADE", 55.0), 2)
          daily_pnl = round(daily_pnl + net, 2)
          trades_today += 1
          print(
              f"\n[EXIT EXECUTED] {active_sym} @ ₹{exit_px:.2f} | Reason: {exit_reason}"
              f" | Net P&L: ₹{net:.2f} | Daily P&L: ₹{daily_pnl:.2f} | Trades: {trades_today}/{max_trades}"
          )
          log_trade([
              now.strftime("%Y-%m-%d %H:%M:%S"),
              active_sym,
              qty,
              round(entry_price, 2),
              round(exit_px, 2),
              round(highest_price, 2),
              stage,
              exit_reason,
              gross,
              net,
              daily_pnl,
          ])
          in_trade = False
          stage = "IDLE"
          climax_high = None
          state[active_sym]["base_locked"] = False
          state[active_sym]["candles"].clear()
          state[active_sym]["base_high"] = None
          state[active_sym]["base_low"] = None
          continue

      # 6. Check 3-Minute Candle Close Boundary
      sim_ticks_in_bucket += 1
      if config.MODE == "SIMULATION":
        candle_closed = sim_ticks_in_bucket >= 4
      else:
        curr_bucket = current_candle_bucket()
        candle_closed = curr_bucket != last_bucket
        if candle_closed:
          last_bucket = curr_bucket

      if candle_closed:
        sim_ticks_in_bucket = 0
        st = state[active_sym]
        if not st["ticks"]:
          continue

        c_open = st["ticks"][0]
        c_high = max(st["ticks"])
        c_low = min(st["ticks"])
        c_close = st["ticks"][-1]
        st["ticks"].clear()

        candle = {"open": c_open, "high": c_high, "low": c_low, "close": c_close}
        st["candles"].append(candle)
        closes = [c["close"] for c in st["candles"]]
        ema9 = compute_ema(closes, period=9)

        need_warmup = (
            st["base_high"] is None
            or (started_late and len(st["candles"]) <= MIN_OBSERVE_CANDLES)
        )

        # PHASE A: Freeze Hour / Base Observation (before 11:30 AM or late-start warmup)
        if now_t < window_start or need_warmup:
          st["base_high"] = (
              c_high if st["base_high"] is None else max(st["base_high"], c_high)
          )
          st["base_low"] = (
              c_low if st["base_low"] is None else min(st["base_low"], c_low)
          )
          print(
              f"\n[BASE OBSERVE #{len(st['candles'])} | {active_sym}]"
              f" Close: {c_close:.2f} | Base: [{st['base_low']:.2f} - {st['base_high']:.2f}]"
          )

        # PHASE B: Active Entry Hunting (11:30 AM - 01:30 PM)
        elif not in_trade:
          if not st["base_locked"]:
            st["base_locked"] = True
            print(
                f"\n>>> [BASE LOCKED | {active_sym}] High: {st['base_high']:.2f}"
                f" | Low: {st['base_low']:.2f} | Hunting Breakouts <<<"
            )

          spread_now = st["last_quote"]["spread"] if st["last_quote"] else 0.0
          print(
              f"[SCANNING 3M | {active_sym}] Close: {c_close:.2f}"
              f" | Base High: {st['base_high']:.2f} | 9 EMA: {ema9:.2f} | Spread: ₹{spread_now:.2f}"
          )

          if c_close > st["base_high"]:
            chase_limit = st["base_high"] * (1.0 + CHASE_GUARD_PCT)
            if c_close > chase_limit:
              print(
                  f"[SKIP BREAKOUT] Close {c_close:.2f} > 5% Chase Guard ({chase_limit:.2f})."
              )
            elif spread_now > max_spread:
              print(
                  f"[SKIP BREAKOUT] Spread ₹{spread_now:.2f} > Max ₹{max_spread:.2f}."
              )
            else:
              order_resp = broker.place_market_order(active_sym, exchange, "BUY", qty)
              if order_resp and order_resp["status"] == "success":
                in_trade = True
                entry_price = (
                    order_resp["fill_price"]
                    if order_resp["fill_price"] > 0
                    else c_close
                )
                highest_price = entry_price

                raw_sl_pts = entry_price - st["base_low"]
                if raw_sl_pts <= 0 or raw_sl_pts > max_sl_points:
                  stop_loss = round(entry_price - max_sl_points, 2)
                else:
                  stop_loss = round(st["base_low"], 2)

                initial_risk_pts = round(entry_price - stop_loss, 2)
                breakeven_trigger = round(entry_price + (1.5 * initial_risk_pts), 2)
                profit_lock_trigger = round(entry_price + (2.5 * initial_risk_pts), 2)
                stage = "INITIAL_BASE_SL"
                print(
                    f"\n>>> [ENTRY TRIGGERED] BUY {qty} {active_sym} @ ₹{entry_price:.2f}"
                    f" | SL: ₹{stop_loss:.2f} (Risk: {initial_risk_pts:.2f} pts / ₹{initial_risk_pts*qty:.0f})"
                    f" | 1.5R BE: ₹{breakeven_trigger:.2f} | 2.5R +1R Lock: ₹{profit_lock_trigger:.2f} <<<"
                )

        # PHASE C: Candle-Close Trailing & Momentum Exit
        else:
          candles = st["candles"]
          if stage == "STAGE2_PROFIT_TRAIL" and len(candles) >= 2:
            two_candle_low = min(candles[-1]["low"], candles[-2]["low"])
            if two_candle_low > stop_loss:
              stop_loss = round(two_candle_low, 2)
              print(f"[STAGE 2 TRAIL] 2-Candle Low SL raised to ₹{stop_loss:.2f}")

          body = abs(c_close - c_open)
          upper_wick = c_high - max(c_open, c_close)
          if body > 0 and upper_wick >= (2.0 * body) and c_close > entry_price * 1.25:
            climax_high = c_low
            print(f"[CLIMAX WICK ALERT] Exit armed if next candle breaks {climax_high:.2f}")
          elif climax_high is not None and c_close < climax_high:
            exit_resp = broker.place_market_order(active_sym, exchange, "SELL", qty)
            exit_px = (
                exit_resp["fill_price"]
                if (exit_resp and exit_resp["fill_price"] > 0)
                else c_close
            )
            gross = round((exit_px - entry_price) * qty, 2)
            net = round(gross - getattr(config, "ESTIMATED_CHARGES_PER_TRADE", 55.0), 2)
            daily_pnl = round(daily_pnl + net, 2)
            trades_today += 1
            print(f"\n[CLIMAX EXIT] {active_sym} @ ₹{exit_px:.2f} | Net P&L: ₹{net:.2f}")
            log_trade([
                now.strftime("%Y-%m-%d %H:%M:%S"),
                active_sym,
                qty,
                round(entry_price, 2),
                round(exit_px, 2),
                round(highest_price, 2),
                stage,
                "CLIMAX_WICK_EXIT",
                gross,
                net,
                daily_pnl,
            ])
            in_trade = False
            stage = "IDLE"
            climax_high = None
            state[active_sym]["base_locked"] = False
            state[active_sym]["candles"].clear()
            state[active_sym]["base_high"] = None
            state[active_sym]["base_low"] = None
            continue

          if stage == "STAGE2_PROFIT_TRAIL" and c_close < ema9:
            exit_resp = broker.place_market_order(active_sym, exchange, "SELL", qty)
            exit_px = (
                exit_resp["fill_price"]
                if (exit_resp and exit_resp["fill_price"] > 0)
                else c_close
            )
            gross = round((exit_px - entry_price) * qty, 2)
            net = round(gross - getattr(config, "ESTIMATED_CHARGES_PER_TRADE", 55.0), 2)
            daily_pnl = round(daily_pnl + net, 2)
            trades_today += 1
            print(
                f"\n[STAGE 3: 9 EMA EXIT] {active_sym} closed ₹{c_close:.2f} < 9 EMA ₹{ema9:.2f}"
                f" | Net P&L: ₹{net:.2f}"
            )
            log_trade([
                now.strftime("%Y-%m-%d %H:%M:%S"),
                active_sym,
                qty,
                round(entry_price, 2),
                round(exit_px, 2),
                round(highest_price, 2),
                stage,
                "9_EMA_MOMENTUM_EXIT",
                gross,
                net,
                daily_pnl,
            ])
            in_trade = False
            stage = "IDLE"
            climax_high = None
            state[active_sym]["base_locked"] = False
            state[active_sym]["candles"].clear()
            state[active_sym]["base_high"] = None
            state[active_sym]["base_low"] = None
            continue

      if in_trade and state[active_sym]["last_quote"]:
        ltp = state[active_sym]["last_quote"]["ltp"]
        unrealized = round((ltp - entry_price) * qty, 2)
        pct = ((ltp - entry_price) / entry_price) * 100.0
        print(
            f"[{stage} | {active_sym}] LTP: {ltp:.2f} ({pct:+.1f}%)"
            f" | SL: {stop_loss:.2f} | 1.5R BE: {breakeven_trigger:.2f} | P&L: ₹{unrealized:.1f}"
        )

      time.sleep(0.3 if config.MODE == "SIMULATION" else 1.5)

  except KeyboardInterrupt:
    print("\n[STOPPED] Engine manually terminated by user.")
  finally:
    broker.shutdown()


if __name__ == "__main__":
  run_v3_engine()

```

---

### Quick Setup & Push for V3.3

1. Save `config_local.py` with your `OPENALGO_API_KEY = "..."` and ensure `config_local.py` is listed in `.gitignore`.
2. Paste and save all 4 files above in VS Code, then compile and push:
```bash
cd ~/trading-projects/option-trailing-algo
echo "config_local.py" >> .gitignore
python3 -m py_compile broker_adapter.py config.py strike_screener.py v3_momentum_engine.py && echo "V3.3 COMPILED CLEANLY!"
git add .gitignore broker_adapter.py config.py strike_screener.py v3_momentum_engine.py
git commit -m "Feat: V3.3 historical base sync on rotation, 2.5R +1R profit lock, orderbook exact fill & 0-DTE guard"
git push

```



Let me know once V3.3 is pushed and we'll build **`equity_intraday_engine.py`**!
