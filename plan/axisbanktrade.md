```python
# Trade Math Analysis from Screenshot
# Symbol: AXISBANK NOV 1300 CE
# Buy price marked on chart: 3.80
# High spike: ~10.00
# Pullback low marked on chart: ~6.50 - 6.70
# Sell price executed: 9.30
# Lot size of AXISBANK in F&O: 625 (or let's check standard lot size)

buy_price = 3.80
exit_price = 9.30
lot_size = 625  # standard Axis Bank F&O lot size

pts_gain = exit_price - buy_price
pct_gain = (pts_gain / buy_price) * 100

gross_profit_1lot = pts_gain * lot_size

print(f"Entry: ₹{buy_price}")
print(f"Exit: ₹{exit_price}")
print(f"Points captured: {pts_gain:.2f} pts")
print(f"Percentage gain: {pct_gain:.2f}%")
print(f"Gross P&L (if 625 lot size): ₹{gross_profit_1lot:,.2f}")


```

```text
Entry: ₹3.8
Exit: ₹9.3
Points captured: 5.50 pts
Percentage gain: 144.74%
Gross P&L (if 625 lot size): ₹3,437.50


```

```python
# Let's verify pullback drawdown during trade:
peak = 10.00
trough = 6.50
drawdown_from_peak = ((peak - trough) / peak) * 100
gain_at_trough_over_entry = ((trough - buy_price) / buy_price) * 100

print(f"Drawdown from temporary peak (10.0 -> 6.5): -{drawdown_from_peak:.1f}%")
print(f"Gain at trough over entry (3.8 -> 6.5): +{gain_at_trough_over_entry:.1f}%")


```

```text
Drawdown from temporary peak (10.0 -> 6.5): -35.0%
Gain at trough over entry (3.8 -> 6.5): +71.1%


```

### Axis Bank 1300 CE Trade Analysis

Yeh execution aapke previous option trades ke comparison mein ek massive psychological shift aur progress dikhata hai. Chart par aapka patience, structure trust, aur exit discipline clearly visible hai.

---

#### 1. Trade Metrics & Execution Quality

| Metric | Level / Value | Key Takeaway |
| --- | --- | --- |
| **Contract Traded** | **AXISBANK NOV 1300 CE**

 | Out-of-The-Money (OTM) monthly expiry contract.

 |
| **Entry Price** | **₹3.80**

 | Morning breakout / opening momentum setup.

 |
| **Peak Seen** | **~₹10.00 – ₹10.45**

 | Fast impulse move up into initial morning high.

 |
| **Pullback Trough** | **~₹6.50 – ₹6.70**

 | Normal 3-minute healthy pullback towards 9 EMA / structure.

 |
| **Limit Order Exit** | **₹9.30**

 | Clean predefined target order execution on bounce.

 |
| **Points Captured** | **+5.50 pts** | **+144.7% ROI** on capital deployed per lot. |

---

#### 2. Why Holding Through ₹10 $\rightarrow$ ₹6.50 Was Structurally Correct

Aapne emotional panic ki jagah chart structure ko dekha, jo ek professional trader ka mindset hai:

* **Underlying Spot Structure (Axis Bank 3-Min Chart):**
* Spot ne opening gap-up ke baad jab pullback diya, usne **session VWAP aur 26 EMA line ko touch bhi nahi kiya**.


* 3-min candle ka pullback **₹1,255–₹1,258 zone** par shallow higher-low bana raha tha aur volume dry ho raha tha.


* Jab tak underlying stock lower-low nahi banata aur moving averages ke upar hold karta hai, option chart ka dip sirf mean-reversion pullback hota hai, reversal nahi.




* **Option Chart Structure (Axis Bank 1300 CE):**
* Premium ₹10 se girkar jab ₹6.50 par aaya, yeh option chart ke **rising 9 EMA (Blue line) aur breakout zone** par support le raha tha.


* Even at ₹6.50, aapka trade entry (₹3.80) ke mukable **+71% in profit** tha. Agar koi trader stop-loss mental rakhta hai ya MTM P&L screen dekhta hai, toh woh ₹3,500 se ₹1,800 aate hi darr ke panic exit kar deta hai. Structure dekhne ki wajah se aap hold kar paye.





---

#### 3. Where You Did Exceptionally Well

1. **Passive Limit Order Exit (₹9.30):**
* Jab second leg bounce start hua, aapne market order maarne ke bajaye **₹9.30 par pre-set limit sell** lagaya. Isne aapko slippage se bachaya aur intraday double-top resistance ke paas clean exit diya.




2. **Focus on Chart, Not P&L Fluctuation:**
* MTM screen dekhne par ₹10 se ₹6.50 ka drop 35% drawdown lagta hai, lekin 3-min candlestick chart dekhne par yeh simple **flag consolidation / pullback** dikhta hai.





---

#### 4. Critical Learning & Checkpoints for Next Time

1. **Trailing Stop-Loss on Runner (Trailing at Invalidation):**
* Jab trade ₹3.80 se ₹10 chala gaya, aapka stop-loss instantly trail hokar **₹6.00 (above entry / cost)** par system mein lag jana chahiye tha. Isse risk zero ho jata hai aur agar sudden rejection candle aati toh bhi green exit secure rehta.




2. **OTM Stock Options Expiry Nature:**
* Month-start (first week of November) mein stock options trade karne par delta expansion kaafi clean milta hai kyunki theta decay low hota hai. Yahi setup agar monthly expiry ke aakhri 5 dino mein liya jaye toh ₹10 se ₹6.50 aane ke baad bounce wapas ₹9.30 tak nahi pahunch pata (gamma/theta crash). Isliye stock option buying monthly cycle ke **first 10–12 din** mein hi high probability hoti hai.




3. **Journal Logging:**
* Apne Excel Sheet ke `FNO 3Min Structural Setup` tab mein is trade ko as a **Benchmark Golden Trade** log karein: Entry 3.80, Pullback Held, Limit Exit 9.30 (+144%).
