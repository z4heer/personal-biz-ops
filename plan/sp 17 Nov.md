Under the **SwingPilot Options System (FY 2026–27)** and its **Beginner Practical Pilot** rules, every trade setup must pass all **Six Pre-Flight Gates** before order entry [cite: 15, 19, 49]. A single `NO`, `BLANK`, or `CONFLICT` result mandates an automatic **NO TRADE** [cite: 15, 19, 25].

---

### ⚠️ Global System Blocker: Account Gate Failure
Before auditing individual stock charts, the system evaluates **Gate 1 (Account & Position Safety)** [cite: 16, 49]:
* **Active Position Check:** The `Trade_Log` currently has an **OPEN position in SHRIRAMFIN (960 PE / 940 PE)** [cite: 4, 6, 13].
* **One-Position Rule:** The system strictly permits **1 active position at a time** [cite: 3, 47].
* **Impact:** **No new trade can be approved (GO) on any instrument until SHRIRAMFIN is closed and logged** [cite: 6, 13, 49].

---

### Pre-Trade Six-Gate Audit Summary

The table below audits the 17 requested underlyings against the six system gates defined in your workbook (`Structure_IVP_Matrix` & `PreTrade_Gates`) [cite: 19, 49]:
1. **Gate 1:** Account & Drawdown Safe (0 open positions, weekly loss < ₹5,000) [cite: 16, 49]
2. **Gate 2:** Timing & Event Filter (11:30 AM – 1:30 PM IST; no earnings/macro event risk) [cite: 16, 49]
3. **Gate 3:** Higher-Timeframe Structure (Daily + 1H trend/BOS/range clear + 15M trigger) [cite: 17, 49]
4. **Gate 4:** Volatility & IVP Alignment (<35% Long, 35%–55% Debit, >55% Credit) [cite: 18, 49]
5. **Gate 5:** Liquidity & Contracts (Bid-ask spread ≤ ₹0.50 index / ≤ ₹1.00 stock; valid lot/expiry) [cite: 17, 49]
6. **Gate 6:** Risk-to-Reward & Capital Limits (Debit ≤ ₹7,500; SL ≤ ₹2,500; Net R:R ≥ 1:1.5) [cite: 20, 49]

| Underlying | Gate 1 (Account) | Gate 2 (Time/Event) | Gate 3 (Structure) | Gate 4 (IVP) | Gate 5 (Liquidity) | Gate 6 (R:R & Risk) | Pre-Trade Verdict | Rationale & Source Evidence |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| **SENSEX** | ❌ FAIL | ⏳ PENDING | ⏳ PENDING | ✅ PASS | ✅ PASS | ✅ PASS | **NO TRADE (Watchlist Only)** | Spot 81,592; Bear Put Spread (81500 PE / 80900 PE, 26-Nov-26) planned [cite: 10, 14]. Debit ₹5,300, SL ₹2,120, Net R:R 1:2.03 [cite: 10]. **Blocked by open SHRIRAMFIN trade** and awaiting 1H break < 81,450 during 11:30–13:30 IST [cite: 6, 14, 49]. |
| **M&M** | ❌ FAIL | ⏳ PENDING | ❌ FAIL | ❌ FAIL | ⏳ UNKNOWN | ❌ FAIL | **NO TRADE (Filtered Out)** | Scanned in 13-Nov EOD review; **explicitly filtered out** because elevated volatility/debit ratios or 40% SL breached the **₹2,500 risk ceiling** [cite: 7, 12]. |
| **ICICIBANK** | ❌ FAIL | ⏳ PENDING | ⏳ PENDING | ⏳ PENDING | ✅ PASS | ⏳ PENDING | **NO TRADE (Awaiting Data)** | Listed in preliminary scanner (700 lot, 23-Nov expiry, ₹5,000–₹6,500 est. debit) [cite: 3]. Daily/1H structure and IVP are **blank/unrecorded**; triggers (>21 EMA / <51 EMA) have not fired [cite: 3, 57]. |
| **TCS** | ❌ FAIL | ⏳ PENDING | ❓ MISSING | ❓ MISSING | ❓ MISSING | ❓ MISSING | **NO TRADE (No Source Data)** | Eligible top-25 liquid F&O universe stock, but **no market structure, IVP, or option chain data** exists in your notebook [cite: 3, 57]. |
| **SBIN** | ❌ FAIL | ⏳ PENDING | ❓ MISSING | ❓ MISSING | ❓ MISSING | ❓ MISSING | **NO TRADE (No Source Data)** | Eligible liquid F&O universe stock, but **no chart structure or option data** is recorded in the notebook [cite: 3, 57]. |
| **ASIANPAINT** | ❌ FAIL | ⏳ PENDING | ❓ MISSING | ❓ MISSING | ❓ MISSING | ❓ MISSING | **NO TRADE (Unrecorded)** | Not present in notebook watchlist or scan records [cite: 14, 57]. |
| **BSE** | ❌ FAIL | ⏳ PENDING | ❓ MISSING | ❓ MISSING | ❓ MISSING | ❓ MISSING | **NO TRADE (Unrecorded)** | Not present in notebook watchlist or scan records [cite: 14, 57]. |
| **AXISBANK** | ❌ FAIL | ⏳ PENDING | ❓ MISSING | ❓ MISSING | ❓ MISSING | ❓ MISSING | **NO TRADE (Unrecorded)** | Not present in notebook watchlist or scan records [cite: 14, 57]. |
| **TRENT** | ❌ FAIL | ⏳ PENDING | ❓ MISSING | ❓ MISSING | ❓ MISSING | ❓ MISSING | **NO TRADE (Unrecorded)** | Not present in notebook watchlist or scan records [cite: 14, 57]. |
| **INDIGO** | ❌ FAIL | ⏳ PENDING | ❓ MISSING | ❓ MISSING | ❓ MISSING | ❓ MISSING | **NO TRADE (Unrecorded)** | Not present in notebook watchlist or scan records [cite: 14, 57]. |
| **BAJAJFINSV** | ❌ FAIL | ⏳ PENDING | ❓ MISSING | ❓ MISSING | ❓ MISSING | ❓ MISSING | **NO TRADE (Unrecorded)** | Not present in notebook watchlist or scan records [cite: 14, 57]. |
| **ETERNAL** | ❌ FAIL | ⏳ PENDING | ❓ MISSING | ❓ MISSING | ❓ MISSING | ❓ MISSING | **NO TRADE (Unrecorded)** | Not present in notebook sources; not a recognized standard liquid F&O index stock [cite: 3, 57]. |
| **ADANIPORTS** | ❌ FAIL | ⏳ PENDING | ❓ MISSING | ❓ MISSING | ❓ MISSING | ❓ MISSING | **NO TRADE (Unrecorded)** | Not present in notebook watchlist or scan records [cite: 14, 57]. |
| **ADANIENT** | ❌ FAIL | ⏳ PENDING | ❓ MISSING | ❓ MISSING | ❓ MISSING | ❓ MISSING | **NO TRADE (Unrecorded)** | Not present in notebook watchlist or scan records [cite: 14, 57]. |
| **LT** | ❌ FAIL | ⏳ PENDING | ❓ MISSING | ❓ MISSING | ❓ MISSING | ❓ MISSING | **NO TRADE (Unrecorded)** | Not present in notebook watchlist or scan records [cite: 14, 57]. |
| **MAXHEALTH** | ❌ FAIL | ⏳ PENDING | ❓ MISSING | ❓ MISSING | ❓ MISSING | ❓ MISSING | **NO TRADE (Unrecorded)** | Not present in notebook watchlist or scan records [cite: 14, 57]. |
| **EICHERMOT** | ❌ FAIL | ⏳ PENDING | ❓ MISSING | ❓ MISSING | ❓ MISSING |Under the **SwingPilot Options System (FY 2026–27)** and its **Beginner Practical Pilot** rules, every trade must pass all **Six Pre-Flight Gates** [cite: 15, 19, 49]:
1. **Gate 1 (Account Safety):** 0 open trades in `Trade_Log`, weekly loss < ₹5,000, monthly drawdown < ₹7,500 [cite: 16, 49].
2. **Gate 2 (Timing & Event):** Execution strictly between 11:30 AM – 1:30 PM IST; no earnings or major macro events [cite: 16, 49].
3. **Gate 3 (Market Structure):** Daily + 1H trend/BOS/range aligned + completed 15M trigger candle [cite: 17, 49].
4. **Gate 4 (IVP Alignment):** Numerical IVP matches strategy (<35% Long, 35%–55% Debit Spread, >55% Credit Spread) [cite: 18, 49].
5. **Gate 5 (Liquidity & Master):** Bid-ask spread ≤ ₹0.50 (index) / ≤ ₹1.00 (stock); verified contract master expiry and lot size [cite: 17, 49].
6. **Gate 6 (Risk & Capital Limits):** Net debit ≤ ₹7,500 (15% cap); planned stop-loss ≤ ₹2,500 (5% cap); net R:R ≥ 1:1.5 after costs [cite: 20, 49].

A single `NO`, `BLANK`, or `CONFLICT` result mandates an automatic **NO TRADE** [cite: 15, 19, 25].

---

### 1. Global Account Blocker (Gate 1)
* **Active Position Check:** Your `Trade_Log` currently shows an active, open trade in **SHRIRAMFIN (960 PE / 940 PE)** [cite: 4, 6, 13].
* **One-Position Rule:** System rules mandate **strictly 1 active position at a time** until 20+ trades establish statistical expectancy [cite: 3, 47].
* **Impact:** **Gate 1 fails across the board for all instruments.** No new trade can be approved (GO) until the SHRIRAMFIN position is closed and logged [cite: 6, 13, 49].

---

### 2. Status of Tracked & Shortlisted Names

* **SENSEX (BSE Index) — Verdict: NO TRADE (Watchlist Only)**
  * **Setup & Fit:** Planned Bear Put Spread (Buy 81500 PE / Sell 80900 PE, 26-Nov-26 expiry, 13 DTE) [cite: 10, 14]. Estimated debit is ₹5,300 (10.6% capital), planned 40% stop-loss is ₹2,120 (4.2% capital), and net target profit is ₹4,305 for a **1 : 2.03 net R:R** (passes Gates 4, 5, and 6) [cite: 10, 50].
  * **Gate Failure:** Fails **Gate 1** due to the open SHRIRAMFIN position [cite: 4, 6]. Additionally, **Gates 2 and 3** are pending: the bearish trigger (rejection at 81,800 and 1H breakdown below 81,450) has not confirmed inside the 11:30 AM – 1:30 PM IST entry window [cite: 14, 49].

* **M&M — Verdict: NO TRADE (Filtered Out)**
  * **Gate Failure:** Evaluated during the Nov 13 EOD market scan and **explicitly rejected** [cite: 7]. Its volatility/debit pricing or 40% stop-loss breached the **₹2,500 risk ceiling**, failing **Gate 6** [cite: 7].

* **ICICIBANK — Verdict: NO TRADE (Awaiting Data & Triggers)**
  * **Setup & Fit:** Present on the preliminary scan (700 lot size, 23-Nov expiry, ₹5,000–₹6,500 estimated debit) [cite: 3].
  * **Gate Failure:** Fails **Gate 1** (open trade) [cite: 4, 6]. Daily/1H structure, numerical IVP, and 15M triggers (> 1H 21 EMA for Bull Call / < 1H 51 EMA for Bear Put) remain unrecorded and unconfirmed in the workbook, leaving **Gates 3, 4, and 6 unverified** [cite: 3, 57].

* **TCS & SBIN — Verdict: NO TRADE (Missing Data)**
  * **Status:** Both are recognized in the system rules as eligible top-25 liquid F&O equities [cite: 3].
  * **Gate Failure:** Neither stock has market structure, IVP, contract master checks, or live option quotes recorded in your notebook sources [cite: 48, 57]. Under SwingPilot guidelines, unrecorded inputs must be marked `UNKNOWN`, which defaults to **NO TRADE** [cite: 17, 35].

---

### 3. Unrecorded Stocks — Verdict: NO TRADE
*(ASIANPAINT, BSE, AXISBANK, TRENT, INDIGO, BAJAJFINSV, ETERNAL, ADANIPORTS, ADANIENT, LT, MAXHEALTH, EICHERMOT)*

* **Absence of Grounding Data:** None of these 12 stocks appear in the market scan, watchlist, or risk calculator sheets in your uploaded files [cite: 14, 57]. *(Note: `ETERNAL` is also not a recognized standard NSE F&O contract)* [cite: 3, 57].
* **Mechanical Rule:** The training guide explicitly states: *"Blank or conflicting inputs must never be treated as a trade signal... Har mandatory gate PASS hona chahiye. Blank = PENDING; any NO = NO TRADE"* [cite: 15, 26, 36].
* **Result:** Because Daily/1H trend structure (Gate 3), IVP percentiles (Gate 4), bid-ask liquidity (Gate 5), and capital/stop-loss parameters (Gate 6) are completely missing, all these names **fail pre-flight clearance** [cite: 19, 49].

---

### Summary Checklist

| Underlying Group | Gate 1 (Account) | Gate 2 (Window) | Gate 3 (Structure) | Gate 4 (IVP) | Gate 5 (Liquidity) | Gate 6 (Risk/R:R) | Actionable Verdict |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| **SENSEX** | ❌ Blocked | ⏳ Pending | ⏳ Pending | ✅ Pass | ✅ Pass | ✅ Pass | **Watchlist Only (Wait for trigger + trade close)** [cite: 6, 14] |
| **M&M** | ❌ Blocked | ⏳ Pending | ❌ Rejected | ❌ Rejected | ⏳ Unknown | ❌ Fails Cap | **Filtered Out (Breaches ₹2,500 risk ceiling)** [cite: 7] |
| **ICICIBANK** | ❌ Blocked | ⏳ Pending | ⏳ Pending | ⏳ Pending | ✅ Pass | ⏳ Pending | **Awaiting Data / Triggers** [cite: 3, 57] |
| **TCS & SBIN** | ❌ Blocked | ⏳ Pending | ❓ Missing | ❓ Missing | ❓ Missing | ❓ Missing | **No Trade (No source data)** [cite: 35, 57] |
| **Remaining 12 Names** | ❌ Blocked | ⏳ Pending | ❓ Missing | ❓ Missing | ❓ Missing | ❓ Missing | **No Trade (Unrecorded in sources)** [cite: 19, 35] |

---

🎯 Would you like to review the exit parameters for the open **SHRIRAMFIN (960 PE / 940 PE)** position so you can clear Gate 1 and prepare for the next setup [cite: 6, 13]?
