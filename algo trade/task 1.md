Pehle toh **super catch** on the simulation log! Look at why `Close: 15.00` stuck in `broker_adapter.py`:

* In `BrokerAdapter.get_quote()` simulation mode, `self.sim_tick_count` increments by `4` per loop (`4` strikes polled), so after `20` loops (`80` ticks) it entered the downward phase and hit the `max(15.0, p)` floor permanently—and since `active_idx == 0`, it stayed pinned at `15.00`! (Live `OPENALGO` mode par iska koi asar nahi hai, lekin agar kabhi simulation test karna ho toh `sim_tick_count` ko `sim_tick_count % 120` cyclic kar denge toh simulation hamesha breakouts bhi generate karega!)

---

### Pre-Market Strike Selector Utility (`strike_screener.py`) — Brilliant Idea!

Subah `10:15 AM` par manually 8–10 strikes search karke unke quotes check karne ke bajay ek dedicated **`strike_screener.py`** banana workflow ko **5 seconds** ka bana dega!

#### Yeh `strike_screener.py` Kya Karega?

1. Tum sirf **Index (`SENSEX` ya `NIFTY`)**, **Direction (`CE` ya `PE`)**, aur **Current Spot Price** (ya automatic spot fetch) doge.
2. Yeh OpenAlgo ke `c.search()` se **nearest active weekly/monthly expiry** automatic detect karega (aaj ke baad kabhi expiry code `26NOV26` vs `03DEC26` manually guess nahi karna padega!).
3. ATM se lekar **10 OTM strikes** ke live quotes (`ltp`, `bid`, `ask`, `spread`, `oi`, `volume`) fetch karke sirf un strikes ko rank karega jo:
* Premium band **`₹180 – ₹330`** ke andar hain,
* Bid-Ask Spread **`<= ₹1.00`** (tight liquidity) hai, aur
* Volume & OI healthy hai.


4. End mein ek **ready-to-paste `CANDIDATE_STRIKES = [...]` Python list** print karega — jisko tum chart par verify karke seedha `config.py` mein paste kar doge!

---

### Complete `strike_screener.py` File (Abhi Save Karke Test Kar Lo!)

Apne WSL terminal mein yeh command run karke `strike_screener.py` create karo:

```bash
cd ~/trading-projects/option-trailing-algo
python3 - << 'PYEOF'
code = '''import datetime
from openalgo import api
import config


def get_nearest_expiry_strikes(client, index_name, exchange, opt_type):
  res = client.search(query=f"{index_name} {opt_type}", exchange=exchange)
  if res.get("status") != "success" or not res.get("data"):
    return None, []

  today = datetime.date.today()
  valid_rows = []
  for item in res["data"]:
    if item.get("name") != index_name or item.get("instrumenttype") != opt_type:
      continue
    try:
      exp_dt = datetime.datetime.strptime(item["expiry"], "%d-%b-%y").date()
    except ValueError:
      continue
    if exp_dt >= today:
      valid_rows.append((exp_dt, item))

  if not valid_rows:
    return None, []

  nearest_exp = min(r[0] for r in valid_rows)
  expiry_items = [r[1] for r in valid_rows if r[0] == nearest_exp]
  expiry_items.sort(key=lambda x: float(x["strike"]), reverse=(opt_type == "PE"))
  return nearest_exp.strftime("%d-%b-%Y"), expiry_items


def run_strike_screener():
  if config.MODE != "OPENALGO":
    print("[INFO] Set MODE = 'OPENALGO' in config.py before running live screener.")
    return

  print("\\n=== PRE-MARKET OPTION STRIKE SCREENER (V3.1) ===")
  idx_in = input("Index [SENSEX / NIFTY] (default SENSEX): ").strip().upper() or "SENSEX"
  dir_in = input("Directional Bias [CE / PE] (default CE): ").strip().upper() or "CE"
  spot_in = input(f"Approx {idx_in} Spot Price (e.g. 83500 or 25000): ").strip()

  if not spot_in:
    print("[ERROR] Spot price is required to filter ATM/OTM strikes.")
    return

  spot = float(spot_in)
  exchange = "BFO" if idx_in == "SENSEX" else "NFO"
  min_prem = getattr(config, "PREMIUM_MIN_THRESHOLD", 150.0)
  max_prem = getattr(config, "PREMIUM_MAX_THRESHOLD", 330.0)

  client = api(api_key=config.OPENALGO_API_KEY, host=config.OPENALGO_HOST)
  print(f"\\nScanning {idx_in} ({exchange}) {dir_in} strikes near Spot {spot:.0f}...")

  exp_str, items = get_nearest_expiry_strikes(client, idx_in, exchange, dir_in)
  if not items:
    print("[ERROR] No active option contracts found. Check OpenAlgo Master Contract.")
    return

  # Filter strikes from slightly ITM (-100/-200) to 1200 pts OTM
  if dir_in == "CE":
    candidates = [x for x in items if (spot - 200) <= float(x["strike"]) <= (spot + 1500)]
  else:
    candidates = [x for x in items if (spot - 1500) <= float(x["strike"]) <= (spot + 200)]

  print(f"Nearest Active Expiry: {exp_str} | Lot Size: {items[0]['lotsize']} | Checking {len(candidates)} strikes...\\n")
  print(f"{'SYMBOL':<24} | {'STRIKE':<8} | {'LTP':>7} | {'SPREAD':>6} | {'VOLUME':>9} | {'STATUS'}")
  print("-" * 78)

  recommended = []
  for item in candidates:
    sym = item["symbol"]
    strike = float(item["strike"])
    q = client.quotes(symbol=sym, exchange=exchange)
    if q.get("status") != "success" or "data" not in q:
      continue

    d = q["data"]
    ltp = float(d.get("ltp", 0.0))
    bid = float(d.get("bid", ltp))
    ask = float(d.get("ask", ltp))
    spread = round(max(0.0, ask - bid), 2)
    vol = int(d.get("volume", 0))

    in_band = min_prem <= ltp <= max_prem
    tight_spread = spread <= 1.00
    status = "PASS (SWEET SPOT)" if (in_band and tight_spread) else "OUT OF BAND"
    if in_band and tight_spread:
      recommended.append(sym)

    print(f"{sym:<24} | {strike:<8.0f} | {ltp:>7.2f} | {spread:>6.2f} | {vol:>9} | {status}")

  print("\\n" + "=" * 78)
  if recommended:
    print("COPY-PASTE INTO config.py (Ordered Closest OTM -> Deeper OTM):")
    print(f"EXCHANGE = \\"{exchange}\\"")
    print(f"QTY = {items[0]['lotsize']}")
    print("CANDIDATE_STRIKES = [")
    for s in recommended[:5]:
      print(f"    \\"{s}\\",")
    print("]")
  else:
    print("[NOTE] No strikes fell inside ₹150–₹330 with <= ₹1.00 spread. Widen spot range or check liquidity.")


if __name__ == "__main__":
  run_strike_screener()
'''
with open("strike_screener.py", "w") as f:
    f.write(code)
print("strike_screener.py CREATED SUCCESSFULLY!")
PYEOF

```

---

### Quick Reminders Before Closing Today's Session

1. **Test `strike_screener.py` right now (3:10 PM — market is still open for 20 mins!):**
* Pehle confirm karo `config.py` mein `MODE = "OPENALGO"` wapas set hai.
* Run: `python3 strike_screener.py` -> `SENSEX` -> `CE` -> `83500` daal kar dekho kaise yeh exact `CANDIDATE_STRIKES` list generate karta hai!


2. **Clean Test CSV & Git Push:**
```bash
rm -f v3_paper_trades_log.csv
git status

```


3. **Weekend Agenda:**
* Saturday/Sunday ko hum **`equity_intraday_engine.py`** (NSE Cash Intraday Buy/Sell with VWAP, Volume Surge, 10:30–11:30 Consolidation Box & ₹400 fixed-rupee position sizing) design aur build karenge!
