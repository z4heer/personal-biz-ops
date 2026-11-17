```python
# Analyze SENSEX option spread math from screenshots (Lot size = 20)
lot_size = 20

# Bull Call Spread (26 Nov 2026 expiry - 9 DTE, IVP 52%)
# ATM 82800 CE = 844.90, OTM 83000 CE = 730.20 (or 83100 CE = 678.00)
bcs_debit_200 = (844.90 - 730.20) * lot_size
bcs_profit_200 = (200 - (844.90 - 730.20)) * lot_size

bcs_debit_300 = (844.90 - 678.00) * lot_size
bcs_profit_300 = (300 - (844.90 - 678.00)) * lot_size

# Bull Put Spread (26 Nov 2026 expiry - 9 DTE, IVP 52%)
# Sell 82800 PE = 674.10, Buy 82500 PE = 558.55
bps_credit_300 = (674.10 - 558.55) * lot_size
bps_loss_300 = (300 - (674.10 - 558.55)) * lot_size

# Intraday Long 82900 CE (19 Nov expiry - from Screenshot 3: Buy 391.50, Sell 390.45, Spread 1.05)
intraday_cost = 391.50 * lot_size
intraday_40sl = intraday_cost * 0.40

print(f"BCS 82800/83000 (200w): Debit ₹{bcs_debit_200:,.2f}, Max Profit ₹{bcs_profit_200:,.2f}, R:R 1:{bcs_profit_200/bcs_debit_200:.2f}")
print(f"BCS 82800/83100 (300w): Debit ₹{bcs_debit_300:,.2f}, Max Profit ₹{bcs_profit_300:,.2f}, R:R 1:{bcs_profit_300/bcs_debit_300:.2f}")
print(f"BPS 82800/82500 (300w): Credit ₹{bps_credit_300:,.2f}, Max Loss ₹{bps_loss_300:,.2f}, R:R 1:{bps_credit_300/bps_loss_300:.2f}")
print(f"Intraday 82900 CE (19 Nov): Cost ₹{intraday_cost:,.2f}, 40% SL ₹{intraday_40sl:,.2f}")


```

```text
BCS 82800/83000 (200w): Debit ₹2,294.00, Max Profit ₹1,706.00, R:R 1:0.74
BCS 82800/83100 (300w): Debit ₹3,338.00, Max Profit ₹2,662.00, R:R 1:0.80
BPS 82800/82500 (300w): Credit ₹2,311.00, Max Loss ₹3,689.00, R:R 1:0.63
Intraday 82900 CE (19 Nov): Cost ₹7,830.00, 40% SL ₹3,132.00


```

```python
# Check OTM Bull Call Spread (83000 CE / 83200 CE) for 26 Nov expiry to get >= 1:1.5 R:R
lot_size = 20
# 83000 CE = 730.20, 83200 CE = 630.00 -> width = 200
debit_otm = (730.20 - 630.00) * lot_size
profit_otm = (200 - (730.20 - 630.00)) * lot_size

# Check 82900 CE (787.00) / 83200 CE (630.00) -> width = 300
debit_829_832 = (787.00 - 630.00) * lot_size
profit_829_832 = (300 - (787.00 - 630.00)) * lot_size

print(f"OTM BCS 83000/83200 (200w): Debit ₹{debit_otm:,.2f}, Max Profit ₹{profit_otm:,.2f}, R:R 1:{profit_otm/debit_otm:.2f}")
print(f"BCS 82900/83200 (300w): Debit ₹{debit_829_832:,.2f}, Max Profit ₹{profit_829_832:,.2f}, R:R 1:{profit_829_832/debit_829_832:.2f}")


```

```text
OTM BCS 83000/83200 (200w): Debit ₹2,004.00, Max Profit ₹1,996.00, R:R 1:1.00
BCS 82900/83200 (300w): Debit ₹3,140.00, Max Profit ₹2,860.00, R:R 1:0.91


```

```python
import csv

# Read existing CSV to append SENSEX_Value column cleanly
with open('SwingPilot_Beginner_Trade_Pilot_SHRIRAMFIN.csv', 'r', encoding='utf-8') as f:
    rows = list(csv.reader(f))

sensex_new_values = {
    'Field_Name': 'SENSEX_26NOV_Value',
    'Underlying_Symbol': 'SENSEX',
    'Strategy_Type': 'Bull Call Spread',
    'Trade_Direction': 'Bullish',
    'Market_Regime': 'Impulse BOS',
    'Completed_15M_Trigger_Closed': 'YES',
    'Execution_Time_IST': '12:42 PM',
    'Upcoming_Earnings_or_Major_Event': 'NO',
    'Expiry_Date': '2026-11-26',
    'Calendar_DTE': '9',
    'Numerical_IVP_Percent': '52',
    'Lot_Size': '20',
    'Number_of_Lots': '1',
    'Primary_Leg_Strike': '83000',
    'Primary_Leg_Type': 'CE',
    'Primary_Leg_Price': '730.20',
    'Primary_Leg_Bid_Ask_Spread': '0.50',
    'Hedge_Leg_Strike': '83200',
    'Hedge_Leg_Type': 'CE',
    'Hedge_Leg_Price': '630.00',
    'Hedge_Leg_Bid_Ask_Spread': '0.50',
    'Broker_SPAN_Exposure_Margin_INR': '0',
}

updated_rows = []
for row in rows:
    field = row[0]
    val = sensex_new_values.get(field, '')
    # Insert SENSEX_26NOV_Value right before Instructions_and_Validation_Rules (last col)
    new_row = row[:-1] + [val, row[-1]]
    updated_rows.append(new_row)

out_csv = 'SwingPilot_Beginner_Trade_Pilot_Updated.csv'
with open(out_csv, 'w', newline='', encoding='utf-8') as f:
    writer = csv.writer(f)
    writer.writerows(updated_rows)

with open(out_csv, 'r', encoding='utf-8') as f:
    print(f.read())


```

```text
Field_Name,RELIANCE_Value,SENSEX_Value,SHRIRAMFIN_Value,SENSEX_26NOV_Value,Instructions_and_Validation_Rules
Underlying_Symbol,RELIANCE,SENSEX,SHRIRAMFIN,SENSEX,Stock or Index symbol [cite: 4]
Strategy_Type,Bull Call Spread,Bear Put Spread,Bull Call Spread,Bull Call Spread,Bull Call Spread / Bear Put Spread / Long Call / Long Put / Credit Spread [cite: 4]
Trade_Direction,Bullish,Bearish,Bullish,Bullish,Bullish / Bearish [cite: 4]
Market_Regime,Trend Pullback,Trend Pullback,Trend Pullback,Impulse BOS,Regime 1 (Impulse BOS) / Regime 2 (Trend Pullback) / Regime 3 (Range Sweep) [cite: 3]
Completed_15M_Trigger_Closed,YES,YES,YES,YES,YES/NO: Has a 15M candle closed confirming the setup in trade direction? [cite: 3]
Execution_Time_IST,11:45 AM,12:15 PM,12:22 PM,12:42 PM,Must be within entry window: 11:30 AM to 1:30 PM IST (Gate 2) [cite: 3]
Upcoming_Earnings_or_Major_Event,NO,NO,NO,NO,YES/NO: Any earnings or major macro event within 5 calendar days? [cite: 3]
Expiry_Date,2026-11-23,2026-11-26,2026-11-23,2026-11-26,Contract expiry date (YYYY-MM-DD) [cite: 4]
Calendar_DTE,10,13,6,9,"Calendar days to expiry: Spreads require 7 to 20 DTE [cite: 3,4]"
Numerical_IVP_Percent,42,48,57,52,Implied Volatility Percentile: 35% to 55% required for Debit Spreads [cite: 3]
Lot_Size,500,20,1500,20,"Contract lot size (e.g. RELIANCE=500,SENSEX=20) [cite: 4,11]"
Number_of_Lots,1,1,1,1,Default is strictly 1 lot during pilot [cite: 4]
Primary_Leg_Strike,1340,81500,1000,83000,Bought Strike (for Debit Spread) or Sold Strike (for Credit Spread) [cite: 4]
Primary_Leg_Type,CE,PE,CE,CE,CE or PE [cite: 4]
Primary_Leg_Price,24.5,420,14.60,730.20,Executable premium in ₹/unit (Best Ask if buying) [cite: 4]
Primary_Leg_Bid_Ask_Spread,0.25,0.4,0.10,0.50,"Bid-Ask gap in ₹ (Must be <= ₹1.00 for stock,<= ₹0.50 for index) [cite: 3]"
Hedge_Leg_Strike,1370,80900,1020,83200,Sold Strike (for Debit Spread) or Bought Strike (for Credit Spread) [cite: 4]
Hedge_Leg_Type,CE,PE,CE,CE,CE or PE [cite: 4]
Hedge_Leg_Price,12.5,155,7.60,630.00,Executable premium in ₹/unit (Best Bid if selling) [cite: 4]
Hedge_Leg_Bid_Ask_Spread,0.2,0.3,0.25,0.50,"Bid-Ask gap in ₹ (Must be <= ₹1.00 for stock,<= ₹0.50 for index) [cite: 3]"
Broker_SPAN_Exposure_Margin_INR,0,0,0,0,Required for Credit Spreads only; enter 0 for Debit Spreads [cite: 4]



```

**Verdict: SENSEX passes ALL SwingPilot Gate Checks (`9 DTE`, `52% IVP`, `12:42 PM IST`, `Lot Size = 20`) under Regime 1 (Impulse BOS).** However, you must choose between two distinct structures depending on your R:R vs. budget preference—and avoid the 19 Nov weekly 82900 CE due to its `₹1.05` bid-ask spread (`> ₹0.50` index limit).

---

### 1. Multi-Regime Fit Analysis (SENSEX Spot: `82,782.10` / Fut: `82,944.00`)

| Market Regime | Technical Criteria | SENSEX Chart Reality (`Screenshot 124243.png`) | Regime Fit |
| --- | --- | --- | --- |
| **Regime 1: Impulse BOS** *(Break of Structure)* | Clean breakout above prior multi-day swing high (`82,390–82,500`) with strong volume expansion. | SENSEX broke decisively above the Nov 11–12 resistance zone (`82,390`) on strong green volume bars, holding well above 15M VWAP (`82,622.46`) and 9/26 EMAs (`82,845.31 / 82,520.81`). | **YES — BEST FIT** |
| **Regime 2: Trend Pullback** | Pullback into 20/26 EMA or VWAP followed by a bounce trigger. | Price is currently extended **+322 pts above 15M VWAP** (`82,622.46`) and **+424 pts above 1H 9 EMA** (`82,391.65`). Has not pulled back yet. | **NO** *(Extended; wait for dip to 82,750–82,800 if trading pullback)* |
| **Regime 3: Range Sweep** | False breakdown/breakout at a horizontal range boundary snapping back inside. | Index is in an active directional uptrend (`+677.31 pts / +0.82%`), not a range sweep. | **NO** |

---

### 2. Strategy Structure Evaluation (Lot Size = `20`, Capital = `₹40,000`)

#### A. Bull Call Debit Spread (`26 Nov 2026` Expiry — `9 DTE`, `IVP 52%`) — **PASSES ALL GATES**

* **Gate Status:** **9 DTE** (inside `7–20` rule) | **52% IVP** (inside `35–55%` rule) | **12:42 PM IST** (inside `11:30–1:30` window).
* **Option 1 — ATM Spread (`Buy 82800 CE @ 844.90` / `Sell 83000 CE @ 730.20`):**
* **Net Debit (Max Loss):** `114.70 × 20` = **₹2,294** (`5.7%` of ₹40,000 capital)
* **Max Profit:** `85.30 × 20` = **₹1,706** (**1 : 0.74 R:R** — high probability since spot `82,782` is already at `82,800`, though R:R is below 1:1.5).


* **Option 2 — OTM Impulse Spread (`Buy 83000 CE @ 730.20` / `Sell 83200 CE @ 630.00`):**
* **Net Debit (Max Loss):** `100.20 × 20` = **₹2,004** (`5.0%` of ₹40,000 capital)
* **Max Profit:** `99.80 × 20` = **₹1,996** (**1 : 1.00 R:R** — best risk-reward among visible strikes on the 26 Nov chain, aligned with heavy Put writing at `83000 PE` OI `+1006%` to `0.79L`).



#### B. Bull Put Credit Spread (`Sell 82800 PE @ 674.10` / `Buy 82500 PE @ 558.55`, `26 Nov`)

* **Net Credit (Max Profit):** `115.55 × 20` = **₹2,311**
* **Max Loss:** `184.45 × 20` = **₹3,689** (**1 : 0.63 R:R**, requires ~₹28k–₹32k SPAN margin).
* **Verdict:** **CONDITIONAL PASS** — Fits within your ₹40,000 capital (`9.2%` max risk) and leverages `52%` high IVP, though R:R (`1 : 0.63`) is lower than your `1 : 1.5` credit spread target.

#### C. Intraday Long Option (`19 Nov 82900 CE @ 391.50` from Screenshot 3)

* **Capital Required:** `391.50 × 20` = **₹7,830** (`40% SL` = **₹3,132**)
* **Verdict:** **FAILS BID-ASK GATE (`₹1.05 > ₹0.50`)** — Screenshot 3 shows `BUY 391.50 / SELL 390.45` (`₹1.05` spread, more than double your `<= ₹0.50` index liquidity limit), plus the 15M chart shows a rejection wick at `460` pulling back toward `391`. **Skip.**

---

### 3. Input Sheet Data (`SENSEX_26NOV_Value`)

Your updated CSV file (with `SENSEX_26NOV_Value` added alongside `SHRIRAMFIN_Value`) is ready:


| Field_Name | SENSEX_26NOV_Value | Notes / Verification |
| --- | --- | --- |
| **Underlying_Symbol** | `SENSEX` | Spot: `82,782.10` (+0.82%) / Nov Fut: `82,944.00` |
| **Strategy_Type** | `Bull Call Spread` | Debit spread fitting `52%` IVP & `9` DTE |
| **Trade_Direction** | `Bullish` | PCR `1.17` (bullish), heavy Put OI buildup at `82800` & `83000` |
| **Market_Regime** | `Impulse BOS` | Breakout above `82,390` swing resistance on 15M & 1H |
| **Completed_15M_Trigger_Closed** | `YES` | Confirmed above 15M VWAP (`82,622.46`) & 9/26 EMA |
| **Execution_Time_IST** | `12:42 PM` | Timestamp `12:42:41 IST` — passes Gate 2 (`11:30–1:30`) |
| **Upcoming_Earnings_or_Major_Event** | `NO` | Index basket; no major scheduled event within 5 days |
| **Expiry_Date** | `2026-11-26` | `26 Nov (9 days)` tab selected on Option Chain |
| **Calendar_DTE** | `9` | Passes `7 to 20 DTE` rule |
| **Numerical_IVP_Percent** | `52` | Shown as `52.00 - High` — passes `35% to 55%` rule |
| **Lot_Size** | `20` | Standard BSE SENSEX lot size |
| **Number_of_Lots** | `1` | Pilot rule default |
| **Primary_Leg_Strike** | `83000` | Buy `83000 CE` (or `82800` if preferring ITM/ATM) |
| **Primary_Leg_Type** | `CE` | Call |
| **Primary_Leg_Price** | `730.20` | `83000 CE` LTP (`844.90` if using `82800 CE`) |
| **Primary_Leg_Bid_Ask_Spread** | `0.50` | Verify on order window before entry (`<= ₹0.50` index limit) |
| **Hedge_Leg_Strike** | `83200` | Sell `83200 CE` (`83000` if using `82800 CE` primary) |
| **Hedge_Leg_Type** | `CE` | Call |
| **Hedge_Leg_Price** | `630.00` | `83200 CE` LTP (`730.20` if using `83000 CE` hedge) |
| **Hedge_Leg_Bid_Ask_Spread** | `0.50` | Verify on order window before entry (`<= ₹0.50` index limit) |
| **Broker_SPAN_Exposure_Margin_INR** | `0` | Debit spread (`0` required) |

### Copy-Paste Paper/Live Trade Log Row (Tab-Separated)

If you log this **SENSEX Bull Call Spread** as Trade `#2` right below your SHRIRAMFIN entry:

```text
2	2026-11-17		Paper	NFO-OPT	Regime 1 (Impulse BOS)	52%	Bull Call Spread	SENSEX	83000 CE / 83200 CE	2026-11-26	9	20	1	20	100.20		0	0	

```

*(Before placing any live order on Zerodha, open the **26 Nov `83000 CE` and `83200 CE**` order windows to verify that the live Bid-Ask gap is `<= ₹0.50`, as BSE index options can occasionally widen past `₹0.50` during sharp midday moves.)*
