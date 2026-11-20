That SQLite + 3s-timeout `strike_screener.py` is running like clockwork—**0.244s** DB lookup and instant parallel-safe quotes for both `SENSEX` (`BFO`) and `NIFTY` (`NFO`)!

### Two Subtle Quant Upgrades Spotted in Your NIFTY Output

Look closely at the NIFTY table you just generated (`Lot: 65`, `Expiry: 23-Nov-2026`):

1. **NIFTY Premium Band Calibration (`₹80 – ₹180` vs SENSEX `₹150 – ₹330`):**
* Because NIFTY's lot size is **65** (vs SENSEX **20**), a ₹150 NIFTY option carries `₹9,750` notional value—and on a Monday expiry (`23-Nov-2026`), ATM `25800CE` is `₹99.85` and `25850CE` is `₹78.90`.
* Using SENSEX's `₹150–₹330` band on NIFTY filtered out `25750CE` (`₹125.10`) and `25800CE` (`₹99.85`), leaving only ITM `25700CE` (`₹154.00`).
* **Fix:** Auto-scale the sweet-spot band in `strike_screener.py` by index:
* **SENSEX (`Lot 20`):** `₹150 – ₹330` (Max SL cap `25 pts` = `₹500` risk)
* **NIFTY (`Lot 65`):** `₹75 – ₹180` (Max SL cap `8 pts` = `₹520` risk — keeping your `₹500` per-trade risk identical across both indices!)




2. **Dynamic SL Cap in `v3_momentum_engine.py` (`₹500` Risk Budget Across Both Indices):**
* Right now `MAX_SL_POINTS = 25.0` is hardcoded in `v3_momentum_engine.py`. On `20 qty` SENSEX, `25 pts = ₹500` risk—but on `65 qty` NIFTY, `25 pts` would equal `₹1,625` risk!
* If we define `MAX_RISK_RUPEES_PER_TRADE = 500.0` and compute `max_sl_pts = round(MAX_RISK_RUPEES_PER_TRADE / qty, 2)` dynamically, then:
* **SENSEX (`qty = 20`):** Automatically caps SL at **25.00 pts** (`₹500` risk).
* **NIFTY (`qty = 65`):** Automatically caps SL at **7.70 pts** (`₹500.50` risk).





---

### Quick 10-Second Update to Lock Both Improvements In

Run these two clean commands so Monday morning works seamlessly for **both SENSEX and NIFTY**:

#### 1. Update `strike_screener.py` with Index-Aware Premium Bands

```bash
cd ~/trading-projects/option-trailing-algo
python3 - << 'PYEOF'
with open("strike_screener.py", "r") as f:
    src = f.read()

old_band = """  min_prem = getattr(config, "PREMIUM_MIN_THRESHOLD", 150.0)
  max_prem = getattr(config, "PREMIUM_MAX_THRESHOLD", 330.0)"""

new_band = """  if idx_in == "NIFTY":
    min_prem, max_prem = 75.0, 180.0
  else:
    min_prem = getattr(config, "PREMIUM_MIN_THRESHOLD", 150.0)
    max_prem = getattr(config, "PREMIUM_MAX_THRESHOLD", 330.0)"""

assert old_band in src
with open("strike_screener.py", "w") as f:
    f.write(src.replace(old_band, new_band, 1))
print("strike_screener.py updated with NIFTY (₹75-₹180) & SENSEX (₹150-₹330) bands!")
PYEOF

```

#### 2. Update `v3_momentum_engine.py` with Dynamic `₹500` Risk-Per-Trade SL Cap

```bash
python3 - << 'PYEOF'
with open("v3_momentum_engine.py", "r") as f:
    src = f.read()

src = src.replace(
    "MAX_SL_POINTS = 25.0         # Max ₹500 risk on 20 qty SENSEX (caps wide bases)",
    "MAX_RISK_PER_TRADE = 500.0   # Dynamic ₹500 max risk per trade (25 pts SENSEX / 7.7 pts NIFTY)"
)
src = src.replace(
    "  max_spread = (",
    "  max_sl_points = round(MAX_RISK_PER_TRADE / qty, 2)\n  max_spread = ("
)
src = src.replace("MAX_SL_POINTS", "max_sl_points")

with open("v3_momentum_engine.py", "w") as f:
    f.write(src)
print("v3_momentum_engine.py updated with dynamic ₹500 risk-normalized SL cap!")
PYEOF

```

---

### Final Friday Wrap-Up & Git Push

1. Verify `MODE = "OPENALGO"` in `config.py` and remove any stray CSV test rows (`rm -f v3_paper_trades_log.csv`).
2. Commit and push the complete **V3.1 Multi-Strike Engine + SQLite Screener**:
```bash
git add broker_adapter.py config.py v3_momentum_engine.py strike_screener.py DAILY_LAUNCH_CHEATSHEET.md
git commit -m "Feat: V3.1 multi-strike OTM rotation, ₹500 risk-normalized SL cap, 1:1.5 RR BE & SQLite strike screener"
git push

```



Have a great Friday evening! Ping me whenever you're ready over the weekend to build the companion **`equity_intraday_engine.py`** for NSE cash stocks.
