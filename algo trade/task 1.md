Two reasons why `strike_screener.py` hung:

1. OpenAlgo's local SQLite database locks under concurrent `ThreadPoolExecutor` requests (`max_workers=6`).
2. `client.search(query="SENSEX 84000")` runs a slow fuzzy query across the entire master contract table instead of hitting OpenAlgo's direct `optionchain` endpoint (or querying the local OpenAlgo SQLite database directly in <5 milliseconds!).

Let's test `client.optionchain()` and `client.quotes()` with a simple single-line diagnostic first, and make `strike_screener.py` run sequentially with live progress output (`flush=True`) so it never hangs silently.

### Step 1: Quick 1-Command Diagnostic (Check Which OpenAlgo Endpoint Responds Instantly)

Press `Ctrl+C` in your terminal, then run:

```bash
cd ~/trading-projects/option-trailing-algo
python3 -c '
import time, config
from openalgo import api
c = api(api_key=config.OPENALGO_API_KEY, host=config.OPENALGO_HOST)
t0 = time.time()
print("1. Testing quotes:", c.quotes(symbol="SENSEX26NOV2684100CE", exchange="BFO"), f"({time.time()-t0:.2f}s)")
t1 = time.time()
print("2. Testing optionchain:", str(c.optionchain(underlying="SENSEX", exchange="BFO"))[:250], f"({time.time()-t1:.2f}s)")
'

```

### Step 2: Fast Sequential `strike_screener.py` (Zero SQLite Lock + Instant Local Expiry Calculation)

Instead of calling slow `c.search()`, we can compute the exact OpenAlgo symbol prefix directly (or take the active expiry prefix once) and fetch only 8 targeted strikes sequentially with live console printing:

```bash
python3 - << 'PYEOF'
code = '''import datetime
import time
from openalgo import api
import config


def get_expiry_prefix(client, index_name, exchange, atm_strike, opt_type):
  # Try optionchain first (fast single call)
  try:
    oc = client.optionchain(underlying=index_name, exchange=exchange)
    if isinstance(oc, dict) and oc.get("status") == "success" and oc.get("data"):
      sample_sym = oc["data"][0].get("symbol", "")
      if sample_sym:
        # Extract prefix before digits of strike
        for i, ch in enumerate(sample_sym):
          if i >= len(index_name) + 7 and ch.isdigit():
            return sample_sym[:i]
  except Exception:
    pass

  # Fallback: single targeted search on ATM strike
  res = client.search(query=f"{index_name} {atm_strike} {opt_type}", exchange=exchange)
  if res.get("status") != "success" or not res.get("data"):
    return None

  today = datetime.date.today()
  valid = []
  for item in res["data"]:
    if item.get("name") != index_name or item.get("instrumenttype") != opt_type:
      continue
    try:
      exp_dt = datetime.datetime.strptime(item["expiry"], "%d-%b-%y").date()
    except ValueError:
      continue
    if exp_dt >= today:
      valid.append((exp_dt, item))

  if not valid:
    return None

  _, sample = min(valid, key=lambda x: x[0])
  strike_str = str(int(float(sample["strike"])))
  return sample["symbol"].split(strike_str)[0]


def run_strike_screener():
  if config.MODE != "OPENALGO":
    print("[INFO] Set MODE = 'OPENALGO' in config.py before running live screener.")
    return

  print("\\n=== FAST OPTION STRIKE SCREENER (V3.1) ===")
  idx_in = input("Index [SENSEX / NIFTY] (default SENSEX): ").strip().upper() or "SENSEX"
  dir_in = input("Directional Bias [CE / PE] (default CE): ").strip().upper() or "CE"
  spot_in = input(f"Approx {idx_in} Spot Price (e.g. 84000 or 25000): ").strip()

  if not spot_in:
    print("[ERROR] Spot price is required.")
    return

  spot = float(spot_in)
  exchange = "BFO" if idx_in == "SENSEX" else "NFO"
  step = 100 if idx_in == "SENSEX" else 50
  lot_size = 20 if idx_in == "SENSEX" else 65
  atm = int(round(spot / step) * step)

  min_prem = getattr(config, "PREMIUM_MIN_THRESHOLD", 150.0)
  max_prem = getattr(config, "PREMIUM_MAX_THRESHOLD", 330.0)

  client = api(api_key=config.OPENALGO_API_KEY, host=config.OPENALGO_HOST)
  print("Detecting active expiry prefix...", flush=True)
  prefix = get_expiry_prefix(client, idx_in, exchange, atm, dir_in)

  if not prefix:
    # Manual fallback prompt if search/optionchain is slow
    prefix = input("Enter symbol prefix manually (e.g. SENSEX26NOV26 or NIFTY26NOV26): ").strip().upper()
    if not prefix:
      return

  offsets = range(0, 9) if dir_in == "CE" else range(0, -9, -1)
  strikes_to_check = [atm + (i * step) for i in offsets]

  print(f"\\nPrefix: {prefix} | Lot: {lot_size} | Scanning 9 strikes sequentially...\\n", flush=True)
  print(f"{'SYMBOL':<24} | {'STRIKE':<8} | {'LTP':>7} | {'SPREAD':>6} | {'VOLUME':>9} | {'STATUS'}")
  print("-" * 78)

  recommended = []
  for st in strikes_to_check:
    sym = f"{prefix}{st}{dir_in}"
    try:
      q = client.quotes(symbol=sym, exchange=exchange)
      if q.get("status") != "success" or "data" not in q:
        print(f"{sym:<24} | {st:<8} | {'N/A':>7} | {'-':>6} | {'-':>9} | NO DATA", flush=True)
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

      print(f"{sym:<24} | {st:<8} | {ltp:>7.2f} | {spread:>6.2f} | {vol:>9} | {status}", flush=True)
    except Exception as e:
      print(f"{sym:<24} | {st:<8} | ERR: {e}", flush=True)
    time.sleep(0.15)

  print("\\n" + "=" * 78)
  if recommended:
    print("COPY-PASTE INTO config.py (Ordered Closest OTM -> Deeper OTM):")
    print(f"EXCHANGE = \\"{exchange}\\"")
    print(f"QTY = {lot_size}")
    print("CANDIDATE_STRIKES = [")
    for s in recommended[:5]:
      print(f"    \\"{s}\\",")
    print("]")
  else:
    print("[NOTE] No strikes fell inside ₹150–₹330 with <= ₹1.00 spread.")


if __name__ == "__main__":
  run_strike_screener()
'''
with open("strike_screener.py", "w") as f:
    f.write(code)
print("SEQUENTIAL strike_screener.py WRITTEN!")
PYEOF

```

Run Step 1 first so we see the exact latency of `quotes()` vs `optionchain()`, then run `python3 strike_screener.py`—it will print each strike row live in real time without blocking.
