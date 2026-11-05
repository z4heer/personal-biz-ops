# BTST (Buy Today, Sell Tomorrow) Execution Cheat Sheet

### 1. The 6-Point Filter (Must Score $\ge 5/6$ at 3:15 PM)

| Check | Parameter | Rule Requirement | Disqualification Trigger |
| --- | --- | --- | --- |
| **1. Candle Anatomy** | Daily Bar | Closing in the **top 10%–15%** of the day’s range (near High of Day). | Upper wick $> 20\%$ of the body (indicates rejection). |
| **2. Volume Expansion** | Volume vs. 20 SMA | Volume $\ge \mathbf{1.5\times}$ the 20-day Volume SMA. | Below-average or declining volume into close. |
| **3. Structure & Breakout** | Chart Pattern | Fresh breakout or daily close **above a multi-day resistance level** (5–20 day high). | Stalling right *under* resistance (like AXISBANK at ₹1,290).

 |
| **4. Higher Timeframe Momentum** | Daily RSI (14) | **58 to 68** (strong directional momentum, not exhausted). | RSI $> 75$ (extreme overbought) or $< 50$ (bearish). |
| **5. Universe Filter** | Liquidity & Segment | **F&O eligible large-cap or mid-cap stocks** (zero circuit trap risk). | T2T, SME, or illiquid non-F&O cash counters. |
| **6. Market Bias** | Benchmark Index | Nifty / Sensex trading above its own session VWAP and closing green. | Nifty down $> 1\%$ or breaking major support into 3:00 PM. |

---

### 2. Execution Timing & Routine

* **2:45 PM – 3:05 PM (Screening):** Run your EOD breakout screener. Identify stocks printing a daily expansion candle near day's high.
* **3:10 PM – 3:18 PM (Structure Check):** Verify the stock is not bumping directly into a major weekly resistance level. Check that the 15-minute chart shows sustained closing strength into the final session.
* **3:20 PM – 3:25 PM (Order Placement):**
* Product: **CNC (Cash & Carry)**.
* Order Type: **Limit Order** placed 0.05% below the current market price or at the best bid (never use Market orders into the 3:20 PM volatility spike).


* **Next Morning 9:15 AM – 9:30 AM (Exit Leg):**
* **Gap Up (+1.5% to +3%):** Book profit on open or set a tight trailing stop on the 1-minute chart. Do not turn an overnight gain into a multi-day hope trade.
* **Flat Open (+0.0% to +0.3%):** Give the 9:15–9:20 AM candle a chance to push; if it breaks below yesterday's closing price, exit flat.
* **Gap Down:** Hard stop-loss triggers if price breaks below yesterday's low or $-1.5\%$ from entry.



---

### 3. Sizing & Friction Rules (The Zerodha Reality)

* **DP Charge Impact:** When you sell a CNC position on T+1 morning before final settlement, brokers typically treat it as a delivery sale (subject to delivery STT of 0.1% each side and standard DP charges if applicable).
* **Minimum Allocation Rule:**
* Do not take BTST on small quantities (e.g., 2–5 shares worth ₹1,000–₹2,000) where a ₹15–₹40 friction charge wipes out 25%–50% of the overnight gain.
* Keep BTST position allocation between **₹15,000 and ₹30,000 per setup** so a 1.5% overnight pop (+₹225 to +₹450) absorbs the ₹15.34 DP charge and exchange STT while leaving over 85% of net profit intact.



---

### 4. Immediate Veto Checklist (Instant Skip)

Skip any potential BTST candidate if:

1. Earnings or quarterly financial results are scheduled after 3:30 PM today or before 9:00 AM tomorrow.
2. The stock rallied purely because of an intraday pump without breaking its prior 10-day high.
3. The stock left a massive upper wick on the daily candle by 3:15 PM (sellers pushed it down from highs).
4. The stock is in the Trade-to-Trade (T2T) category (BTST is strictly banned by exchanges for T2T stocks).
