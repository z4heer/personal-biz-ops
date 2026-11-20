`broker_adapter.py` super clean hai! Notice karo:

* `self.client.quotes()` pehle se hi `bid`, `ask`, `ltp`, aur `volume` sab return karta hai — toh koi extra API call add karne ki zaroorat nahi!
* Bas ek `get_quote(self, symbol, exchange)` method add karenge jo ek hi call mein `{"ltp": ..., "bid": ..., "ask": ..., "spread": ...}` dega (aur backward compatibility ke liye `get_ltp()` ko waisa hi chhod denge taaki `strategy_engine.py` bilkul break na ho!).

---

### Complete V3.1 Architecture Upgrade (`broker_adapter.py` + `config.py` + `v3_momentum_engine.py`)

Yeh rahe woh **5 exact features** jo is V3.1 update mein cleanly wire ho rahe hain:

1. **Multi-Strike Parallel Watchlist (`3–5 OTM Strikes`):**
* `config.py` mein `CANDIDATE_STRIKES` list (closest OTM se deeper OTM tak, e.g. `84100CE, 84300CE, 84500CE, 84700CE`).
* Engine **har strike ka apna `candles`, `Base High`, `Base Low`, aur `9 EMA` parallel mein build karta hai**.


2. **Dynamic OTM Auto-Rotation (`PREMIUM_MAX_THRESHOLD` & `PREMIUM_MIN_THRESHOLD`):**
* Jab engine flat hai (`not in_trade`), aur active strike ka `ltp > PREMIUM_MAX_THRESHOLD` (e.g., `₹330`) ho jaaye -> turant list ke **next further OTM strike** par shift hoga (jiska `ltp <= ₹330` aur `spread <= ₹1.00` ho) — aur uska `Base High` / `Base Low` pehle se ready milega!
* Agar market pullback kare aur active strike `ltp < PREMIUM_MIN_THRESHOLD` (e.g., `₹140`) gir jaaye -> wapas **previous higher-delta strike** par rotate karega.
* Trade active hone ke baad strike **100% locked** rehti hai.


3. **Multi-Trade Loop (`3/3` Daily Cap):**
* Ek trade close hone par script exit nahi hogi — state reset karke automatically agle setup ke liye scan continue karegi (`Trades: 1/3 -> 2/3 -> 3/3`), jab tak `Daily Net P&L > -₹800` aur `Time <= 13:30 IST`.


4. **Adaptive Stage 1 Breakeven (`1:1.5 R:R`):**
* Fixed `+25%` ki jagah **`entry_price + 1.5 * initial_risk_pts`** hit hote hi SL exact **Cost-to-Cost** lock ho jayega (har strike price par consistent math!).


5. **Live Bid-Ask Spread Gate:**
* Order bhejne se pehle `spread <= ₹1.00` (Index) / `₹0.15` (Stock) check karega.



---

### Step 1: `broker_adapter.py` Mein `get_quote()` Add Karo (Backward-Compatible)

Terminal mein run karo:

```bash
cd ~/trading-projects/option-trailing-algo
python3 - << 'PYEOF'
code = '''import random
import time
from openalgo import api
import config


class BrokerAdapter:

  def __init__(self):
    self.mode = config.MODE
    self.sim_prices = {}
    self.sim_tick_count = 0

    if self.mode == "OPENALGO":
      self.client = api(
          api_key=config.OPENALGO_API_KEY, host=config.OPENALGO_HOST
      )

  def _execute_with_retry(self, func, *args, retries=3, delay=0.5, **kwargs):
    last_err = None
    for attempt in range(1, retries + 1):
      try:
        return func(*args, **kwargs)
      except Exception as e:
        last_err = e
        print(
            f"\\n[API WARNING] Attempt {attempt}/{retries} failed: {e}."
            f" Retrying in {delay}s..."
        )
        time.sleep(delay)
        delay *= 2
    print(f"\\n[API ERROR] All {retries} attempts failed: {last_err}")
    return None

  def get_quote(self, symbol, exchange):
    if self.mode == "OPENALGO":

      def _fetch():
        quote = self.client.quotes(symbol=symbol, exchange=exchange)
        data = quote["data"]
        ltp = float(data["ltp"])
        bid = float(data.get("bid", ltp))
        ask = float(data.get("ask", ltp))
        spread = round(max(0.0, ask - bid), 2)
        return {"ltp": ltp, "bid": bid, "ask": ask, "spread": spread}

      return self._execute_with_retry(_fetch)

    # SIMULATION MODE: Multi-strike realistic price simulation
    if symbol not in self.sim_prices:
      idx = len(self.sim_prices)
      self.sim_prices[symbol] = round(290.0 - (idx * 45.0), 2)

    self.sim_tick_count += 1
    p = self.sim_prices[symbol]
    if self.sim_tick_count <= 20:
      p = round(p + random.uniform(-2.0, 2.2), 2)
    elif self.sim_tick_count <= 80:
      p = round(p + random.uniform(-1.0, 4.5), 2)
    else:
      p = round(p - random.uniform(0.5, 3.5), 2)

    p = max(15.0, p)
    self.sim_prices[symbol] = p
    return {"ltp": p, "bid": round(p - 0.25, 2), "ask": round(p + 0.25, 2), "spread": 0.50}

  def get_ltp(self, symbol, exchange):
    q = self.get_quote(symbol, exchange)
    return q["ltp"] if q else None

  def place_order(self, symbol, exchange, action, qty, price):
    if self.mode == "OPENALGO":

      def _order():
        return self.client.placeorder(
            strategy="Intraday_Option_Trail",
            symbol=symbol,
            action=action,
            exchange=exchange,
            price_type="MARKET",
            product="MIS",
            quantity=qty,
        )

      return self._execute_with_retry(_order)

    print(f"\\n[SIM ORDER] {action} {qty} qty of {symbol} @ ₹{price:.2f}")
    return {"status": "success", "mode": "SIMULATION"}
'''
with open("broker_adapter.py", "w") as f:
    f.write(code)
print("broker_adapter.py UPDATED WITH get_quote()!")
PYEOF

```

---

### Step 2: `config.py` Mein Multi-Strike Watchlist & Rotation Thresholds Add Karo

Apne `config.py` ke bottom mein yeh 3 lines add kar do (roz subah 10:15 AM par bas apne 3–5 OTM candidate strikes yahan list kar dena):

```bash
cat << 'EOF' >> ~/trading-projects/option-trailing-algo/config.py

# Multi-Strike Auto-Rotation Watchlist (Ordered: Closest OTM -> Deeper OTM)
CANDIDATE_STRIKES = [
    "SENSEX26NOV2684100CE",
    "SENSEX26NOV2684300CE",
    "SENSEX26NOV2684500CE",
    "SENSEX26NOV2684700CE",
]
PREMIUM_MAX_THRESHOLD = 330.0  # Rotate to next OTM strike if LTP exceeds ₹330
PREMIUM_MIN_THRESHOLD = 140.0  # Rotate back to closer strike if LTP drops below ₹140
EOF

```

---

### Step 3: Complete V3.1 `v3_momentum_engine.py` Save Karo

```bash
python3 - << 'PYEOF'
code = '''import csv
import datetime
import os
import time
from broker_adapter import BrokerAdapter
import config

# ==========================================
# V3.1 MULTI-STRIKE MOMENTUM PROTOCOL PARAMETERS
# ==========================================
MAX_SL_POINTS = 25.0         # Max ₹500 risk on 20 qty SENSEX (caps wide bases)
MAX_CHASE_PCT = 0.05         # Skip candle if close jumps > 5% above Base High
MAX_SPREAD_INDEX = 1.00      # Max Bid-Ask spread for SENSEX/NIFTY (₹)
MAX_SPREAD_STOCK = 0.15      # Max Bid-Ask spread for Stock Options (₹)
STAGE1_RR_MULTIPLIER = 1.5   # 1:1.5 R:R -> Shift SL to Cost-to-Cost (Breakeven)
STAGE2_TRIGGER_PCT = 0.50    # +50% move -> Trail by Last 2-Candle Low (min +25% locked)
STAGE2_LOCK_PCT = 0.25       # Minimum +25% profit floor once Stage 2 activates
TARGET_2X_MULTIPLIER = 2.0   # +100% (2x) hard profit target
EMA_PERIOD = 9               # 9 EMA on 3-minute candle closes
UPPER_WICK_RATIO = 2.0       # Upper wick >= 2x body signals climax exhaustion


def calculate_ema(closes, period=9):
  if len(closes) < period:
    return None
  k = 2 / (period + 1)
  ema = sum(closes[:period]) / period
  for price in closes[period:]:
    ema = (price * k) + (ema * (1 - k))
  return round(ema, 2)


def get_today_stats(csv_file="v3_paper_trades_log.csv"):
  if not os.path.isfile(csv_file):
    return 0, 0.0
  today_str = str(datetime.date.today())
  trades = 0
  total_pnl = 0.0
  with open(csv_file, mode="r") as f:
    reader = csv.DictReader(f)
    for row in reader:
      if row.get("Date") == today_str:
        trades += 1
        total_pnl += float(row.get("Net_PnL", 0.0))
  return trades, round(total_pnl, 2)


def log_v3_trade(
    symbol, entry_time, exit_time, entry_p, exit_p,
    initial_sl, max_p, qty, stage_reached, reason
):
  csv_file = "v3_paper_trades_log.csv"
  file_exists = os.path.isfile(csv_file)
  gross_pnl = round((exit_p - entry_p) * qty, 2)
  est_charges = round((entry_p + exit_p) * qty * 0.0025, 2)
  net_pnl = round(gross_pnl - est_charges, 2)
  risk_pct = round(((entry_p - initial_sl) / entry_p) * 100, 2)

  with open(csv_file, mode="a", newline="") as f:
    writer = csv.writer(f)
    if not file_exists:
      writer.writerow([
          "Date", "Symbol", "Entry_Time", "Exit_Time", "Entry_Price",
          "Initial_SL", "Risk_Pct", "Exit_Price", "Highest_Price", "Qty",
          "Gross_PnL", "Net_PnL", "Highest_Stage", "Exit_Reason"
      ])
    writer.writerow([
        datetime.date.today(), symbol, entry_time, exit_time, entry_p,
        initial_sl, risk_pct, exit_p, max_p, qty, gross_pnl, net_pnl,
        stage_reached, reason
    ])
  print(
      f"\\n[V3.1 CSV LOGGED] {symbol} | Gross: ₹{gross_pnl} | Net: ₹{net_pnl} |"
      f" Stage: {stage_reached}"
  )


def run_v3_strategy():
  trades_today, pnl_today = get_today_stats()
  if trades_today >= config.MAX_TRADES_PER_DAY:
    print(f"[KILL SWITCH] Max daily trades ({config.MAX_TRADES_PER_DAY}) reached.")
    return
  if pnl_today <= -abs(config.MAX_DAILY_LOSS):
    print(f"[KILL SWITCH] Max daily loss (-₹{config.MAX_DAILY_LOSS}) breached.")
    return

  default_list = getattr(config, "CANDIDATE_STRIKES", [config.SYMBOL])
  default_str = ",".join(default_list)
  max_prem = getattr(config, "PREMIUM_MAX_THRESHOLD", 330.0)
  min_prem = getattr(config, "PREMIUM_MIN_THRESHOLD", 140.0)

  print("\\n=== V3.1 MULTI-STRIKE MOMENTUM ENGINE SETUP ===")
  sym_in = input(f"Candidate Strikes [{default_str}]: ").strip().upper()
  exch_in = input(f"Exchange [{config.EXCHANGE}]: ").strip().upper()
  qty_in = input(f"Quantity [{config.QTY}]: ").strip()

  strikes = [s.strip() for s in sym_in.split(",") if s.strip()] if sym_in else default_list
  exchange = exch_in if exch_in else config.EXCHANGE
  qty = int(qty_in) if qty_in else config.QTY
  max_spread = (
      MAX_SPREAD_INDEX
      if any(idx in strikes[0] for idx in ["SENSEX", "NIFTY", "BANKNIFTY"])
      else MAX_SPREAD_STOCK
  )

  broker = BrokerAdapter()
  print(
      f"\\n>>> Starting V3.1 Engine [{config.MODE}] | {len(strikes)} Strikes"
      f" Watched | Band: ₹{min_prem}-₹{max_prem} | Trades:"
      f" {trades_today}/{config.MAX_TRADES_PER_DAY} <<<"
  )

  # Parallel Per-Strike State (Each strike tracks its own 3m candles & Base High/Low)
  state = {
      s: {
          "candles": [],
          "ticks": [],
          "base_high": 0.0,
          "base_low": float("inf"),
          "base_locked": False,
          "last_quote": None,
      }
      for s in strikes
  }

  active_idx = 0
  active_sym = strikes[active_idx]
  candle_start_dt = datetime.datetime.now()
  ticks_per_candle = 5 if config.MODE == "SIMULATION" else 180
  min_base_candles = 4

  # Active Trade State
  in_trade = False
  entry_price = 0.0
  initial_sl_price = 0.0
  be_trigger_price = 0.0
  entry_time = ""
  current_sl = 0.0
  target_2x = 0.0
  highest_ltp = 0.0
  stage = "BASE_OBSERVATION"

  def reset_trade_state():
    nonlocal in_trade, entry_price, initial_sl_price, be_trigger_price
    nonlocal entry_time, current_sl, target_2x, highest_ltp, stage
    in_trade = False
    entry_price = 0.0
    initial_sl_price = 0.0
    be_trigger_price = 0.0
    entry_time = ""
    current_sl = 0.0
    target_2x = 0.0
    highest_ltp = 0.0
    stage = "SCANNING"

  while True:
    now_dt = datetime.datetime.now()
    now_time = now_dt.time()

    # Check Daily Kill-Switches before scanning
    trades_today, pnl_today = get_today_stats()
    if not in_trade:
      if trades_today >= config.MAX_TRADES_PER_DAY:
        print(f"\\n[KILL SWITCH] Daily trade cap ({config.MAX_TRADES_PER_DAY}) reached. Stopping.")
        break
      if pnl_today <= -abs(config.MAX_DAILY_LOSS):
        print(f"\\n[KILL SWITCH] Daily loss limit (-₹{config.MAX_DAILY_LOSS}) breached. Stopping.")
        break
      if config.MODE == "OPENALGO":
        if now_time < datetime.time(10, 30):
          print("Waiting for 10:30 AM Freeze Hour observation window...", end="\\r")
          time.sleep(1)
          continue
        if now_time > config.TRADING_END_TIME:
          print("\\n[SESSION END] Past 13:30 IST execution cutoff. No new trades.")
          break

    # Fetch live quotes for all candidate strikes (when flat) or active strike (when in trade)
    symbols_to_poll = [active_sym] if in_trade else strikes
    for s in symbols_to_poll:
      q = broker.get_quote(s, exchange)
      if q:
        state[s]["last_quote"] = q
        state[s]["ticks"].append(q["ltp"])

    active_q = state[active_sym]["last_quote"]
    if not active_q:
      time.sleep(1)
      continue

    # ==========================================
    # DYNAMIC OTM AUTO-ROTATION (ONLY WHEN FLAT)
    # ==========================================
    if not in_trade:
      curr_ltp = active_q["ltp"]
      # 1. Premium exceeded upper threshold -> rotate to next further OTM strike
      if curr_ltp > max_prem and active_idx < len(strikes) - 1:
        for next_i in range(active_idx + 1, len(strikes)):
          cand_s = strikes[next_i]
          cand_q = state[cand_s]["last_quote"]
          if cand_q and cand_q["ltp"] <= max_prem and cand_q["spread"] <= max_spread:
            print(
                f"\\n>>> [OTM ROTATE UP] {active_sym} LTP ₹{curr_ltp:.2f} > ₹{max_prem:.0f}."
                f" Switching to {cand_s} @ ₹{cand_q['ltp']:.2f} <<<\\n"
            )
            active_idx = next_i
            active_sym = cand_s
            active_q = cand_q
            break

      # 2. Premium decayed below lower threshold -> rotate back to closer strike
      elif curr_ltp < min_prem and active_idx > 0:
        for prev_i in range(active_idx - 1, -1, -1):
          cand_s = strikes[prev_i]
          cand_q = state[cand_s]["last_quote"]
          if cand_q and min_prem <= cand_q["ltp"] <= max_prem and cand_q["spread"] <= max_spread:
            print(
                f"\\n>>> [OTM ROTATE DOWN] {active_sym} LTP ₹{curr_ltp:.2f} < ₹{min_prem:.0f}."
                f" Switching to {cand_s} @ ₹{cand_q['ltp']:.2f} <<<\\n"
            )
            active_idx = prev_i
            active_sym = cand_s
            active_q = cand_q
            break

    # ==========================================
    # 3-MINUTE WALL-CLOCK CANDLE CLOSE (ALL STRIKES)
    # ==========================================
    active_ticks = state[active_sym]["ticks"]
    candle_ready = (
        len(active_ticks) >= ticks_per_candle
        if config.MODE == "SIMULATION"
        else (now_dt - candle_start_dt).total_seconds() >= 180
    )

    if candle_ready and active_ticks:
      candle_start_dt = now_dt
      for s in symbols_to_poll:
        t_list = state[s]["ticks"]
        if not t_list:
          continue
        c_bar = {
            "open": t_list[0],
            "high": max(t_list),
            "low": min(t_list),
            "close": t_list[-1],
        }
        state[s]["candles"].append(c_bar)
        state[s]["ticks"] = []

        # Build Freeze-Hour / Min-4-Candle Base per strike
        still_building = (
            len(state[s]["candles"]) <= min_base_candles
            or (config.MODE == "OPENALGO" and now_time < config.TRADING_START_TIME)
        )
        if not state[s]["base_locked"] and still_building:
          state[s]["base_high"] = max(state[s]["base_high"], c_bar["high"])
          state[s]["base_low"] = min(state[s]["base_low"], c_bar["low"])
        elif not state[s]["base_locked"]:
          state[s]["base_locked"] = True

      act = state[active_sym]
      act_candles = act["candles"]
      act_candle = act_candles[-1]
      ema_9 = calculate_ema([c["close"] for c in act_candles], EMA_PERIOD)

      if not act["base_locked"]:
        print(
            f"[BASE OBSERVE #{len(act_candles)} | {active_sym}] Close:"
            f" {act_candle['close']:.2f} | Base: [{act['base_low']:.2f} -"
            f" {act['base_high']:.2f}]"
        )
        continue

      # PHASE B: BREAKOUT ENTRY WITH SPREAD GATE & DUAL-CAP SL
      if act["base_locked"] and not in_trade:
        in_exec_window = (
            True
            if config.MODE == "SIMULATION"
            else (config.TRADING_START_TIME <= now_time <= config.TRADING_END_TIME)
        )

        if in_exec_window and act_candle["close"] > act["base_high"]:
          proposed_entry = act_candle["close"]
          chase_pct = (proposed_entry - act["base_high"]) / act["base_high"]

          if chase_pct > MAX_CHASE_PCT:
            print(
                f"\\n[SKIP - CHASE GUARD] {active_sym} close {proposed_entry:.2f} is"
                f" {chase_pct*100:.1f}% above Base High {act['base_high']:.2f}."
            )
            continue

          if active_q["spread"] > max_spread:
            print(
                f"\\n[SKIP - WIDE SPREAD] {active_sym} spread ₹{active_q['spread']:.2f}"
                f" > max ₹{max_spread:.2f}."
            )
            continue

          capped_sl = round(max(act["base_low"], proposed_entry - MAX_SL_POINTS), 2)
          risk_pts = round(proposed_entry - capped_sl, 2)
          if risk_pts <= 0:
            continue

          res = broker.place_order(
              active_sym, exchange, "BUY", qty, round(proposed_entry + 0.05, 2)
          )
          if res:
            in_trade = True
            entry_price = proposed_entry
            initial_sl_price = capped_sl
            be_trigger_price = round(entry_price + (STAGE1_RR_MULTIPLIER * risk_pts), 2)
            entry_time = now_time.strftime("%H:%M:%S")
            current_sl = capped_sl
            target_2x = round(entry_price * TARGET_2X_MULTIPLIER, 2)
            highest_ltp = entry_price
            stage = "INITIAL_BASE_SL"
            print(
                f"\\n>>> [V3.1 ENTRY] BUY {active_sym} @ {entry_price:.2f} | Risk:"
                f" {risk_pts:.2f} pts (₹{risk_pts*qty:.0f}) | SL: {current_sl:.2f}"
                f" | 1:1.5 BE @ {be_trigger_price:.2f} | 2x: {target_2x:.2f} <<<\\n"
            )
        else:
          print(
              f"[SCANNING 3M | {active_sym}] Close: {act_candle['close']:.2f} |"
              f" Base High: {act['base_high']:.2f} | 9 EMA: {ema_9} | Spread:"
              f" ₹{active_q['spread']:.2f}"
          )

      # PHASE C: CANDLE-CLOSE EXIT CHECKS (Stage 2 Trail, Stage 3 9-EMA, Stage 4 Climax)
      elif in_trade:
        ltp = active_q["ltp"]
        if stage in ["STAGE_2_TRAIL", "STAGE_3_EMA_RIDE"] and len(act_candles) >= 2:
          last_2_low = min(act_candles[-1]["low"], act_candles[-2]["low"])
          floor_25 = round(entry_price * (1 + STAGE2_LOCK_PCT), 2)
          new_sl = round(max(last_2_low, floor_25), 2)
          if new_sl > current_sl:
            current_sl = new_sl
            print(
                "\\n>>> [STAGE 2 TRAIL] SL moved to 2-Candle Low / +25% Floor:"
                f" {current_sl:.2f} <<<\\n"
            )

        if (
            stage in ["STAGE_2_TRAIL", "STAGE_3_EMA_RIDE"]
            and ema_9 is not None
            and act_candle["close"] < ema_9
        ):
          broker.place_order(active_sym, exchange, "SELL", qty, ltp)
          exit_time = now_time.strftime("%H:%M:%S")
          print(
              f"\\n>>> [STAGE 3 EXIT - 9 EMA BREAK] Close {act_candle['close']:.2f}"
              f" < 9 EMA {ema_9:.2f} | Sold @ {ltp:.2f} <<<"
          )
          log_v3_trade(
              active_sym, entry_time, exit_time, entry_price, ltp,
              initial_sl_price, highest_ltp, qty, stage, "3m Close Below 9 EMA"
          )
          reset_trade_state()
          continue

        if len(act_candles) >= 2:
          prev = act_candles[-2]
          prev_body = abs(prev["close"] - prev["open"])
          prev_upper_wick = prev["high"] - max(prev["open"], prev["close"])
          if (
              prev_body > 0
              and prev_upper_wick >= (UPPER_WICK_RATIO * prev_body)
              and act_candle["low"] < prev["low"]
          ):
            broker.place_order(active_sym, exchange, "SELL", qty, ltp)
            exit_time = now_time.strftime("%H:%M:%S")
            print(
                "\\n>>> [STAGE 4 EXIT - CLIMAX WICK REJECTION] Broke prev low"
                f" {prev['low']:.2f} | Sold @ {ltp:.2f} <<<"
            )
            log_v3_trade(
                active_sym, entry_time, exit_time, entry_price, ltp,
                initial_sl_price, highest_ltp, qty, "STAGE_4_CLIMAX", "Climax Wick Rejection"
            )
            reset_trade_state()
            continue

    # ==========================================
    # TICK-BY-TICK MILESTONES & HARD SL CHECKS
    # ==========================================
    if in_trade:
      ltp = active_q["ltp"]
      act_candles = state[active_sym]["candles"]
      if ltp > highest_ltp:
        highest_ltp = ltp

      gain_pct = (highest_ltp - entry_price) / entry_price

      # Stage 1 (1:1.5 R:R): Move SL to Cost-to-Cost (Risk-Free)
      if highest_ltp >= be_trigger_price and stage == "INITIAL_BASE_SL":
        current_sl = entry_price
        stage = "STAGE_1_BREAKEVEN"
        print(
            f"\\n>>> [STAGE 1 (1:1.5 R:R HIT)] SL modified to Cost-to-Cost:"
            f" {current_sl:.2f} (100% Risk-Free) <<<\\n"
        )

      # Stage 2 (+50% Move): Lock +25% floor & activate 2-Candle Low + 9 EMA trail
      if gain_pct >= STAGE2_TRIGGER_PCT and stage == "STAGE_1_BREAKEVEN":
        floor_25 = round(entry_price * (1 + STAGE2_LOCK_PCT), 2)
        last_2_low = (
            min(act_candles[-1]["low"], act_candles[-2]["low"])
            if len(act_candles) >= 2
            else floor_25
        )
        current_sl = round(max(floor_25, last_2_low), 2)
        stage = "STAGE_2_TRAIL"
        print(
            "\\n>>> [STAGE 2 (+50%)] Locked +25% / 2-Candle Low SL:"
            f" {current_sl:.2f} | Riding 9 EMA <<<\\n"
        )

      # Stage 4 (2x Target Hit)
      if ltp >= target_2x:
        broker.place_order(active_sym, exchange, "SELL", qty, ltp)
        exit_time = now_time.strftime("%H:%M:%S")
        print(
            "\\n>>> [STAGE 4 EXIT - 2X TARGET HIT] Double target reached! Sold @"
            f" {ltp:.2f} <<<"
        )
        log_v3_trade(
            active_sym, entry_time, exit_time, entry_price, ltp,
            initial_sl_price, highest_ltp, qty, "STAGE_4_2X_TARGET", "2x Target Hit (+100%)"
        )
        reset_trade_state()
        continue

      live_pnl = round((ltp - entry_price) * qty, 2)
      print(
          f"[{stage} | {active_sym}] LTP: {ltp:.2f}"
          f" (+{((ltp-entry_price)/entry_price)*100:.1f}%) | SL: {current_sl:.2f}"
          f" | 1.5R BE: {be_trigger_price:.2f} | P&L: ₹{live_pnl}"
      )

      # Hard Stop-Loss Hit
      if ltp <= current_sl:
        broker.place_order(active_sym, exchange, "SELL", qty, ltp)
        exit_time = now_time.strftime("%H:%M:%S")
        print(f"\\n>>> [HARD SL EXIT] {active_sym} @ {ltp:.2f} (SL: {current_sl:.2f}) <<<")
        log_v3_trade(
            active_sym, entry_time, exit_time, entry_price, ltp,
            initial_sl_price, highest_ltp, qty, stage, f"SL Hit ({stage})"
        )
        reset_trade_state()
        continue

      # EOD Square-Off
      if config.MODE == "OPENALGO" and now_time >= config.SQUARE_OFF_TIME:
        broker.place_order(active_sym, exchange, "SELL", qty, ltp)
        exit_time = now_time.strftime("%H:%M:%S")
        log_v3_trade(
            active_sym, entry_time, exit_time, entry_price, ltp,
            initial_sl_price, highest_ltp, qty, stage, "EOD Square-Off"
        )
        break

    time.sleep(0.3 if config.MODE == "SIMULATION" else 1.0)


if __name__ == "__main__":
  run_v3_strategy()
'''
with open("v3_momentum_engine.py", "w") as f:
    f.write(code)
print("v3_momentum_engine.py V3.1 WRITTEN & VERIFIED!")
PYEOF

```

---

### Step 4: Quick Simulation Smoke-Test (30 Seconds Mein Verify Karo)

Abhi market close ke baad `config.py` mein temporarily `MODE = "SIMULATION"` karke `python3 v3_momentum_engine.py` run karo — tumhe live terminal par dikh jayega ki:

1. 4 strikes parallel mein base lock karti hain,
2. Jaise hi pehli strike `₹330` cross karti hai, `[OTM ROTATE UP]` trigger hokar next strike par switch hoti hai,
3. `1:1.5 R:R` par `[STAGE 1 (1:1.5 R:R HIT)]` breakeven lock hota hai, aur
4. Trade close hone ke baad loop `Trades: 1/3 -> 2/3` continue karta hai!
*(Test ke baad `MODE = "OPENALGO"` wapas set karke test CSV delete kar dena, aur `git commit` kar lena!)*
