# V3.1 Multi-Strike Momentum Engine
## Complete Execution Flow – Step-by-Step Hinglish Guide

**Script:** `v3_momentum_engine.py`  
**Version:** V3.1  
**Engine Type:** Multi-Strike Momentum Trading Engine

Is document mein V3.1 Multi-Strike Momentum Engine ka poora execution flow simple, step-by-step Hinglish mein explain kiya gaya hai.

Objective hai ki har stage ka exact **kyun (Why), kab (When), aur kaise (How)** crystal-clear rahe, taaki trading execution systematic, disciplined aur rule-based ho.

---

# 1. Pre-Market & Safety Gates
### (Engine Shuru Hone Se Pehle)

Engine trading start karne se pehle kuch mandatory safety checks perform karta hai.

### 1.1 Daily Kill-Switch Check

Script sabse pehle check karti hai ki aaj ke trading day mein:

- Maximum **3 trades** se zyada toh nahi ho gaye?
- Maximum daily loss **-₹800** cross toh nahi hua?

**Decision:**

- Agar dono conditions safe hain, toh engine aage proceed karega.
- Agar maximum trades complete ho gaye hain, toh new trades stop ho jayenge.
- Agar daily loss limit `-₹800` hit ho gayi hai, toh engine trading permanently stop kar dega for the day.

### 1.2 Sweet-Spot Band & Auto OTM Rotation

**Premium Range: ₹140 – ₹330**

`config.py` mein tumne 3–5 OTM strikes ki list configure ki hai, closest OTM se deeper OTM tak.

Jab tak koi trade active nahi hai, engine premium ke basis par strike selection manage karta hai.

| Premium Condition | Engine Action |
|---|---|
| Premium > ₹330 | Agli deeper OTM strike par shift |
| Premium < ₹140 | Pichhli closer OTM strike par shift |
| Premium ₹140–₹330 | Current strike maintain |

**Objective:**

- Expensive premium avoid karna.
- Low-premium, suitable momentum opportunities identify karna.
- Market movement ke according automatically strike rotate karna.

**Important:** Active trade ke dauran selected strike lock rahegi. Rotation sirf new trade selection ke liye hogi.

---

# 2. Phase A: Freeze Hour / Base Observation
### (10:30 AM – 11:30 AM)

Is phase ka objective hai market ki initial movement observe karna aur ek reliable breakout range establish karna.

### 2.1 No-Trade Zone

`10:30 AM` se `11:30 AM` tak:

- Engine koi BUY order place nahi karega.
- Sirf market observe karega.
- 3-minute candles collect karega.
- Base High aur Base Low calculate karega.

### 2.2 3-Minute Candles & Range Box

Engine har 3-minute interval par candle data calculate karta hai:

- Open
- High
- Low
- Close
- 9 EMA

Poore observation period mein:

**Base High:** Sabhi observed candles ka highest price.

**Base Low:** Sabhi observed candles ka lowest price.

Isse ek imaginary range box create hota hai.

Yahi box future breakout entry ka reference banega.

### 2.3 Late Start Safety Rule

Agar kisi din engine `11:30 AM` ke baad late start hota hai:

- Engine blindly entry nahi karega.
- Minimum 4 complete candles observe karega.
- 4 candles × 3 minutes = 12 minutes minimum observation.

Iske baad hi breakout scanning start hogi, subject to the configured entry window.

### 2.4 11:30 AM Sharp – BASE LOCKED

Jaise hi `11:30 AM` hota hai:

- Base High lock.
- Base Low lock.
- Observation phase complete.
- Engine Hunting Mode mein enter karta hai.

**Engine Status:** `SCANNING 3M`

Ab engine breakout opportunity search karega.

---

# 3. Phase B: Breakout Entry & 4 Protection Shields
### (11:30 AM – 01:30 PM)

Jab 11:30 AM se 01:30 PM ke beech koi 3-minute candle Base High ke upar close hoti hai, toh engine immediately BUY nahi karta.

Pehle 4 mandatory protection filters evaluate karta hai.

## Shield 1: 5% Chase Guard

Engine check karta hai ki breakout candle ka closing price Base High se kitna upar hai.

**Condition:**

`Breakout Close <= Base High × 1.05`

Agar candle Base High se 5% se zyada upar close karti hai:

- Trade reject.
- Entry skip.
- Next setup ka wait.

**Logic:** Bhaagti hui train pakadne ki koshish nahi karni.

## Shield 2: Bid-Ask Spread Gate

Engine live Bid aur Ask prices ka difference calculate karta hai.

**Formula:**

`Spread = Ask - Bid`

Maximum acceptable spread:

| Instrument | Maximum Spread |
|---|---:|
| Index | ₹1.00 |
| Stock | ₹0.15 |

Agar spread defined limit se zyada hai:

- Entry reject.
- Illiquid market conditions mein trade avoid.

**Objective:** Poor execution aur unnecessary slippage se bachna.

## Shield 3: Dynamic ₹500 Risk-Normalized Stop-Loss

Initial Stop-Loss reference:

`Initial SL = Base Low`

Lekin agar Base Low bahut door hai, toh engine maximum ₹500 risk per trade maintain karne ke liye stop distance cap karta hai.

**Maximum Risk: ₹500 per Trade**

| Instrument | Lot Quantity | Maximum Risk Distance |
|---|---:|---:|
| SENSEX | 20 | 25 points |
| NIFTY | 65 | Approx. 7.7 points |

Risk calculation:

`Maximum SL Distance = ₹500 / Quantity`

**Important:** Actual SL calculation ko Base Low aur maximum permitted risk distance dono consider karne chahiye. Agar valid stop placement possible nahi hai, toh trade reject hona chahiye.

## Shield 4: Order Execution

Jab saare mandatory filters pass ho jaate hain:

1. Selected strike confirm hoti hai.
2. Strike lock hoti hai.
3. One-lot `MIS MARKET BUY` order fire hota hai.
4. Broker order status verify hota hai.
5. Actual executed price ko Entry Price record kiya jaata hai.
6. Position monitoring start hoti hai.

**Important:** Order submission aur successful execution alag events hain. Engine ko confirmed filled quantity aur actual average fill price ke baad hi position active mark karni chahiye.

---

# 4. Phase C: 4-Stage Smart Trailing Exit
### (Profit Protection & Momentum Management)

Trade execute hone ke baad engine position ko manage karta hai:

- Every 1.5 seconds: Live price monitoring.
- Every 3-minute candle close: Technical exit conditions evaluation.

Objective hai:

1. Initial risk control.
2. Profit protect karna.
3. Momentum continue hone tak position hold karna.
4. Momentum weak hone par exit karna.

## Stage 0 – Initial Base Stop-Loss

Trade execute hone ke baad:

- Initial SL establish hota hai.
- Maximum planned risk ₹500 hai.
- Stop-loss ko risk limit ke andar maintain kiya jaata hai.

Yeh trade ka initial protection layer hai.

## Stage 1 – 1:1.5 R:R Breakeven (Risk-Free Lock)

Jab premium initial risk ka 1.5 times profit achieve karta hai:

`Trigger = Entry Price + (1.5 × Initial Risk)`

Example:

- Entry Price: ₹200
- Initial Risk: 20 points
- Trigger: ₹230

Jaise hi premium ₹230 reach karta hai:

- Stop-Loss Entry Price par shift ho jaata hai.
- Initial capital risk substantially protect ho jaata hai.

**Objective:** Trade ko cost-to-cost protection tak le jaana.

## Stage 2 – +50% Profit Surge (2-Candle Trailing)

Jab option premium entry price se 50% increase karta hai:

`Trigger = Entry Price × 1.50`

Engine:

- Minimum +25% gross profit lock karne ka target rakhta hai.
- Stop-Loss ko previous 2 completed candles ke Low ke neeche trail karta hai.
- Har new completed candle ke saath trailing SL update karta hai.

**Example:**

Entry Price = ₹200

| Parameter | Price |
|---|---:|
| Entry | ₹200 |
| +50% Trigger | ₹300 |
| +25% Profit Lock | ₹250 |

Trailing SL ko kabhi previous SL se neeche nahi le jaana chahiye.

## Stage 3 – 9 EMA Momentum Exit

Stage 2 activate hone ke baad:

Agar koi completed 3-minute candle 9 EMA ke neeche close hoti hai:

- Momentum weakening signal.
- Position exit.
- Profit book.

**Objective:** Strong momentum ke dauran position hold karna aur momentum loss hone par exit karna.

## Stage 4 – Climax Rejection / 2x Target / EOD Square-Off

Is stage mein 3 independent exit conditions monitor hoti hain.

### A. Climax Wick Rejection

Engine shooting-star type rejection identify karta hai.

Condition:

- Upper wick candle body se 2 times badi ho.
- Next candle previous candle ka Low break kare.

Agar dono conditions satisfy hoti hain:

- Exit trigger.
- Momentum exhaustion ke near profit booking.

### B. Double Target – 2x

Agar premium entry price ka double ho jaata hai:

`Target = Entry Price × 2`

Example:

- Entry Price: ₹200
- Target Price: ₹400
- Gross Premium Gain: 100%

Target hit hone par hard exit.

### C. End-of-Day Safety Exit

**Mandatory Exit Time: 03:15 PM**

Agar position abhi bhi open hai:

- Engine 03:15 PM par exit order execute karega.
- Broker auto-square-off se pehle position close karne ka objective rahega.

---

# 5. Loop Reset & Next Trade Selection
### (Trades: 1/3 → 2/3 → 3/3)

Ek trade close hone ke baad engine post-trade processing karta hai.

### 5.1 Trade Logging

Saara trade data `v3_paper_trades_log.csv` mein save hota hai.

Important fields:

- Entry Time
- Exit Time
- Instrument
- Strike
- Entry Price
- Exit Price
- Gross P&L
- Net P&L
- Highest Stage Achieved
- Exit Reason

### 5.2 State Reset

Trade close hone ke baad:

- Active position state reset.
- Previous trade ke trailing variables clear.
- Current trade status update.
- Trade counter increment.
- Next setup ke liye engine ready.

### 5.3 Next Trade Eligibility

Engine next setup scan tabhi karega jab:

- Current time 01:30 PM se pehle ho.
- Daily trade count 3 se kam ho.
- Daily loss limit `-₹800` hit na hui ho.
- Koi active position na ho.

Agar sabhi conditions satisfy hoti hain:

**Engine Status:** `SCANNING 3M`

Next eligible breakout setup ka wait karega.

Agar koi mandatory condition fail hoti hai:

**Engine Status:** `TRADING STOPPED`

---

# 6. Complete Execution Flow – Quick Reference

| Phase | Time | Activity | Key Rule |
|---|---|---|---|
| Pre-Market | Before Trading | Safety checks | Max 3 trades / -₹800 |
| Strike Selection | Before Entry | OTM rotation | ₹140–₹330 |
| Phase A | 10:30–11:30 | Base observation | 3M candles |
| Base Lock | 11:30 AM | Lock range | Base High / Low |
| Phase B | 11:30–01:30 | Breakout scanning | 4 protection shields |
| Stage 0 | After Entry | Initial SL | Max ₹500 risk |
| Stage 1 | +1.5R | Breakeven | Entry price SL |
| Stage 2 | +50% | Profit protection | +25% lock / 2-candle trail |
| Stage 3 | Stage 2 Active | Momentum exit | 9 EMA |
| Stage 4 | Any Active Trade | Special exits | Wick / 2x / EOD |
| Loop Reset | After Exit | Logging & reset | Next eligible setup |

---

# 7. Final Trading Philosophy

V3.1 Multi-Strike Momentum Engine ka core objective hai:

1. **Capital Protection:** Daily loss aur per-trade risk limits strictly follow karna.
2. **No Emotional Trading:** Entry aur exit decisions predefined rules se lena.
3. **Momentum Selection:** Sirf valid breakout opportunities trade karna.
4. **Premium Discipline:** Suitable premium range mein OTM strike select karna.
5. **Profit Protection:** Breakeven aur dynamic trailing SL use karna.
6. **Controlled Frequency:** Maximum 3 trades per day.
7. **Time Discipline:** 01:30 PM ke baad new entry nahi aur 03:15 PM par mandatory exit.

**Golden Rule:**

> No valid setup = No Trade.  
> No confirmation = No Entry.  
> Risk limit breached = Stop Trading.  
> Momentum intact = Hold.  
> Momentum broken = Exit.

**SwingPilot V3.1 – Trade the Setup, Protect the Capital, Follow the System.**

---

**Document Status:** Execution Flow Reference  
**Script:** `v3_momentum_engine.py`  
**Version:** V3.1

**Technical Note:** Yeh document supplied strategy description ka formatted version hai, live script ka independently verified code audit nahi. Actual implementation mein stop-loss calculation, candle-close confirmation, order fills, partial fills, slippage, broker-side protective orders, trailing SL monotonicity aur emergency square-off handling verify karna zaroori hai. Paper trading validation ke baad hi live deployment consider karein.
