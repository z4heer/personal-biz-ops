```python
# Verify lot size and option metrics from search results + screenshots
# Note: Quantsapp & Sahi show lot size = 825 (19,76,700 / 2396 = 825; 17,41,575 / 2111 = 825)
lot_size = 825
buy_1000ce = 14.60
sell_1020ce = 7.60
debit_spread_cost = (buy_1000ce - sell_1020ce) * lot_size
debit_spread_max_profit = (20 - (buy_1000ce - sell_1020ce)) * lot_size

# Credit spread alternative (Bull Put Spread: Sell 980 PE @ 9.40, Buy 960 PE @ 4.45)
sell_980pe = 9.40
buy_960pe = 4.45
net_credit = sell_980pe - buy_960pe
credit_max_profit = net_credit * lot_size
credit_max_loss = (20 - net_credit) * lot_size

# Intraday Long Call (1000 CE @ 14.60, 40% SL = 5.84/unit)
intraday_cost = buy_1000ce * lot_size
intraday_40pct_sl = intraday_cost * 0.40

print(f"Verified Lot Size: {lot_size}")
print(f"Bull Call Spread (1000/1020 CE): Debit ₹{debit_spread_cost:,.2f}, Max Profit ₹{debit_spread_max_profit:,.2f}, R:R 1:{debit_spread_max_profit/debit_spread_cost:.2f}")
print(f"Bull Put Spread (980/960 PE): Credit ₹{credit_max_profit:,.2f}, Max Loss ₹{credit_max_loss:,.2f}, R:R 1:{credit_max_profit/credit_max_loss:.2f}")
print(f"Intraday Long 1000 CE: Capital ₹{intraday_cost:,.2f}, 40% SL Loss ₹{intraday_40pct_sl:,.2f}")


```

```text
Verified Lot Size: 825
Bull Call Spread (1000/1020 CE): Debit ₹5,775.00, Max Profit ₹10,725.00, R:R 1:1.86
Bull Put Spread (980/960 PE): Credit ₹4,083.75, Max Loss ₹12,416.25, R:R 1:0.33
Intraday Long 1000 CE: Capital ₹12,045.00, 40% SL Loss ₹4,818.00


```

First, note one key correction from live NSE F&O option chain data: **SHRIRAMFIN's current lot size is `825**` (not `1,500`), which cuts every position's rupee risk by **45%**.

Evaluating **SHRIRAMFIN** (Spot **₹994.60**, **IVP 57%**, **6 DTE** to `23 Nov 2026`, **PCR 0.88**) across all **3 Market Regimes** and all **3 Strategy Structures** (Intraday Long Option, Debit Spread, Credit Spread):

---

### 1. Multi-Regime Fit Analysis (1H & 15M Chart Structure)

| Market Regime | Technical Criteria | SHRIRAMFIN Chart Reality | Regime Fit |
| --- | --- | --- | --- |
| **Regime 1: Impulse BOS** *(Break of Structure)* | Fresh breakout above major swing high (`₹1,004–₹1,005` on 1H chart) with high volume expansion. | Spot is at **₹994.60**, still **~₹10 below** the Nov 12–13 swing high (`₹1,004–₹1,005`), and the **1000 CE** strike holds the highest Call OI wall (**21.95 lakh** shares). | **NO** *(Not broken out yet)* |
| **Regime 2: Trend Pullback** | Higher highs/lows on 1H; pullback to 20/26 EMA & VWAP support followed by bullish 15M resumption close. | Spot pulled back from `₹1,005` to `₹976` (1H 20 EMA), reclaimed 15M VWAP (`₹989.07`), 9 EMA (`₹991.30`), and 26 EMA (`₹985.31`), and closed bullish at `12:15 PM`. | **YES — BEST FIT** |
| **Regime 3: Range Sweep** | Price trapped inside a horizontal box, sweeping liquidity below range low (`₹975`) or above range high (`₹1,005`) and snapping back inside. | Price is trending mid-range (`₹976` $\rightarrow$ `₹995`) with rising EMAs—not at a range boundary sweep. | **NO** |

---

### 2. Strategy Structure Evaluation (Lot Size = `825`)

Even though **Regime 2 (Trend Pullback)** is structurally valid, your **IVP (57% — High)** and **DTE (6 days)** dictate which instrument fits best:

#### A. Intraday Long Option (`Buy 1000 CE @ ₹14.60`)

* **Capital Required:** `₹14.60 × 825` = **₹12,045**
* **40% Max Stop-Loss:** `₹5.84/unit` = **₹4,818** (`12.0%` of ₹40,000 capital)
* **Verdict:** **CONDITIONAL PASS (Intraday Only — Exit by 3:15 PM).**
* **Why it works:** Intraday avoids overnight theta decay on a 6-DTE contract, and 1000 CE has ultra-tight liquidity (`₹0.10` spread) and strong momentum above 15M VWAP (`₹10.10`).
* **Catch:** Requires an immediate push through **₹1,000** today. If spot stalls at the ₹1,000 OI wall (`21.95L`), exit strictly at VWAP (`₹10.10`) or 40% SL (`₹8.75`).



#### B. Bull Call Debit Spread (`Buy 1000 CE @ ₹14.60` / `Sell 1020 CE @ ₹7.60`)

* **Net Debit (Max Loss):** `₹7.00/unit × 825` = **₹5,775** (`14.4%` of ₹40,000 capital)
* **Max Profit:** `₹13.00/unit × 825` = **₹10,725** (**1 : 1.86 R:R**)
* **Breakeven:** **₹1,007.00** (above the Nov 12–13 swing high of `₹1,005`)
* **Verdict:** **FAILS SWINGPILOT GATE (Paper Trade Only).**
* **Why:** Violates two frozen debit-spread rules (**6 DTE** vs `7–20 DTE` required, and **57% IVP** vs `35–55%` cap), and requires SHRIRAMFIN to break above `₹1,007` within 5 trading sessions against a `₹920` Max Pain pull.



#### C. Bull Put Credit Spread (`Sell 980 PE @ ₹9.40` / `Buy 960 PE @ ₹4.45`)

* **Net Credit (Max Profit):** `₹4.95/unit × 825` = **₹4,083.75**
* **Max Loss:** `₹15.05/unit × 825` = **₹12,416.25** (**1 : 0.33 R:R**, requires ~₹28,000–₹32,000 SPAN margin)
* **Breakeven:** **₹975.05** (protected below today's low of `₹976.00` and 1H 20 EMA `₹977.48`)
* **Verdict:** **STRUCTURALLY BEST FIT FOR 57% IVP & 6 DTE, BUT FAILS CAPITAL/R:R GATE.**
* **Why:** High IVP (`57%`) and short DTE (`6 days`) favor **option selling** (theta & IV crush work in your favor as long as SHRIRAMFIN holds above `₹980`, where Put OI grew `+10.19%` to `8.65L`).
* **Why to skip live:** Max loss (`₹12,416`) is **31% of your ₹40,000 account** with a `1 : 0.33` R:R (below your `>= 1 : 1.5` rule).



---

### Final Decision Matrix

| Structure | Regime Fit | Pilot Rule Check | Max Risk (825 Lot) | Action |
| --- | --- | --- | --- | --- |
| **Intraday Long 1000 CE** | Regime 2 (Pullback) | Best intraday fit (`₹0.10` spread) | **₹4,818** (at 40% SL) | **Only viable live play** if 1000 CE holds above `₹14.00`; square off intraday |
| **Bull Call Spread (1000/1020)** | Regime 2 (Pullback) | **FAIL** (6 DTE < 7; 57% IVP > 55%) | **₹5,775** | **Paper Trade / Log in CSV** (update `Lot_Size` to `825`) |
| **Bull Put Spread (980/960)** | Regime 2 (Pullback) | **FAIL** (R:R 1:0.33; Risk 31% of capital) | **₹12,416** | **Skip** (capital too small for ₹20-wide stock credit spread) |

Would you like me to update the CSV with the corrected **`825` lot size**, or would you prefer to take the **Intraday Long 1000 CE** setup with strict intraday exit levels?
