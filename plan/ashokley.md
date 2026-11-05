```python
# Holdings details from Screenshot 2:
# ASHOKLEY: Qty = 11, Avg cost = 157.73, LTP = 169.04, Invested = 1735.05, Cur. val = 1859.44, P&L = +124.39 (+7.17%), Day chg = +9.74%
# Zerodha DP charges: 13.5 + 18% GST = 15.34 (charged per ISIN sold on a given day regardless of quantity)

qty = 11
avg_cost = 157.73
ltp = 169.04
invested = 1735.05
cur_val = 1859.44
gross_pnl = cur_val - invested
dp_charge = 15.34

net_pnl = gross_pnl - dp_charge
dp_drag_pct = (dp_charge / gross_pnl) * 100

print(f"Gross P&L: ₹{gross_pnl:.2f} (+{(gross_pnl/invested)*100:.2f}%)")
print(f"Net P&L after ₹15.34 DP charge: ₹{net_pnl:.2f}")
print(f"DP charge drag on profit: {dp_drag_pct:.2f}%")

# If partial exit vs full exit:
# If selling partial (say 6 shares today, 5 shares later):
# Today's sell incurs ₹15.34 DP charge. Later day sell also incurs ₹15.34 DP charge (Total DP = ₹30.68)!
print(f"Two-day split DP charge: ₹{dp_charge * 2:.2f} (which would be {(dp_charge * 2 / gross_pnl) * 100:.2f}% of profit)")


```

```text
Gross P&L: ₹124.39 (+7.17%)
Net P&L after ₹15.34 DP charge: ₹109.05
DP charge drag on profit: 12.33%
Two-day split DP charge: ₹30.68 (which would be 24.66% of profit)


```

### Current Trade Metrics (ASHOKLEY)

* **Holding:** 11 shares @ Average Cost **₹157.73** (Invested: ₹1,735.05).


* **LTP:** **₹169.04** (+7.17% total gain, +9.74% today's expansion).


* **Gross Profit:** **+₹124.39**.


* **DP Charge Impact (Zerodha):** Ek single sell execution par flat **₹15.34** (inclusive of GST) lagega.
* **Single-shot exit:** Net Profit = **₹109.05** (DP drag = 12.3%).
* **Split exit (do alag din bechna):** Agar aaj 6 shares aur kal/parso 5 shares beche, toh **₹30.68** DP charges lagenge, jo profit ka seedha **24.7%** kha jayega. Isliye is position ko do alag dinon mein split karke nahi bechna hai; **sell order single execution mein hi lagana hai**.



---

### Chart Structure & Momentum (5-Min Chart)

* **Opening Expansion:** Stock subah ₹154 se gap-up karke ₹169 tak vertical expansion mein gaya hai.


* **Indicators Alignment:** Price **Session VWAP (₹166.48)** aur **9 EMA (₹167.11)** ke comfortably upar consolidate kar raha hai, aur volume surge healthy hai.


* **Round Number Resistance:** **₹170.00** ek immediate psychological ceiling hai.



---

### Recommended Exit Strategy

Aapke paas do disciplined paths hain:

#### Option 1: Goal Met — Book Full Profit Near ₹170 (Conservative)

* **Rationale:** Aapka 7% gain target achieve ho chuka hai (+7.17%). Subah ka +9.7% move vertically stretch ho chuka hai aur ₹170 par supply aane ki probability rehti hai.


* **Execution:** Terminal par limit sell order place karein **₹169.50 – ₹170.00** par all 11 shares ke liye.


* **Outcome:** Clean +7.5% to +7.8% gain lock, zero tracking stress, and single DP deduction of ₹15.34.

#### Option 2: Trail Stop to Ride Towards ₹174–₹175 (Trend Follower)

* **Rationale:** Stock mein volume strong hai aur VWAP (₹166.50) / 9 EMA (₹167.10) se niche koi breakdown candle nahi bani hai.


* **Trailing Level:** Apna mental/system stop-loss **₹166.00** (5-min session VWAP ke just niche) set karein.


* *Agar price ₹170 ke upar 5-min candle close karta hai:* Target open hota hai **₹174.00 – ₹175.00** (+10.5% gain).


* *Agar price reverse hokar ₹166.00 break karta hai:* Turant market exit karein; fir bhi aapka trade **+5.2% net green** mein close hoga.



Agar aap breakout swing trades mein 7% rule ko strict mandate mante hain, toh **₹169.50–₹170.00 ke limit order par pura exit lena** sabse clean approach hai.
