Save this file as **`docs/BEGINNER_HINGLISH_EXECUTION_GUIDE.md`** inside `~/trading-projects/option-trailing-algo/docs/` on your `dev` branch. It explains both **Options V3.4** and **Cash Equity V1.0** in clear, beginner-friendly Hinglish with exact commands, logic behind every step, and emergency controls.

```markdown
# Complete Execution Flow – Step-by-Step Beginner Hinglish Guide
**(Options Momentum Engine V3.4 + NSE Cash Equity Intraday Engine V1.0)**

Ye guide specially is tarah banayi gayi hai ki live market ke dauran aapko bilkul dimaag na lagana pade — bas timing follow karein, commands chalayein, aur bot ko apna kaam karne dein.

---

## Golden Rules (Sabse Pehle Ye 4 Baatein Yaad Rakhein)

1. **Both Engines Run on `dev` Branch:** Aapki `dev` branch mein **Options V3.4** aur **Cash Equity V1.0** dono ka latest, tested code maujood hai. Branch switch karne ki zaroorat nahi hai.
2. **Analyze Mode Check (Zaroori Safety):** Subah OpenAlgo start karne ke baad dashboard par **Purple Pill (`Analyze Mode` / Paper Trading)** ON hona zaroori hai — jab tak forward-testing complete na ho, real paisa risk nahi hoga.
3. **Strict Capital & Loss Cap:**
   - **Options Engine:** Per lot `₹4,000 – ₹5,000` capital | Per trade max loss `₹500` | Daily max loss `-₹800` | Max `3` trades/day.
   - **Cash Equity Engine:** Per trade max `₹25,000` MIS notional (`~₹5,000` margin) | Per trade fixed risk `₹400` | Daily max loss `-₹800` | Max `2` trades/day.
4. **No Manual Panic Exit:** Jab bot trade le le, toh terminal ya broker app se manual exit mat karein (sirf emergency mein hi karein), kyunki bot khud **Breakeven, Profit Lock, Trailing SL, aur EOD Square-Off** handle karta hai.

---

## Step-by-Step Daily Timeline (Subah 09:05 AM se Shaam 03:20 PM Tak)

### Phase 1: Morning Setup (09:05 AM – 09:15 AM) — OpenAlgo & Fyers Login

1. Apna WSL Ubuntu terminal open karein aur OpenAlgo server start karein:
   ```bash
   cd ~/trading-projects/openalgo
   uv run app.py

```

*(Ya aap jo normal start command use karte hain woh chalayein.)*
2. Browser mein **`http://127.0.0.1:5000`** kholein aur **Fyers** account se login karein.
3. Top bar par check karein ki **Analyze Mode (Purple Pill)** **ON** hai.
4. Is terminal ko minimize kar dein (band mat karein, ye background mein chalta rahega).

---

### Phase 2: Options Momentum Engine V3.4 Setup (10:15 AM – 10:25 AM)

#### Step 2.1: Sahi Strike Chunna (`strike_screener.py`)

10:15 AM tak morning ki pehli 1 ghante ki volatility shaant ho jaati hai. Ab naya terminal kholein aur screener chalayein:

```bash
cd ~/trading-projects/option-trailing-algo && source venv/bin/activate
git checkout dev
python3 strike_screener.py

```

* Screen par aapse 3 cheezein poochega:
1. **Index:** `SENSEX` ya `NIFTY` type karein (default `SENSEX`).
2. **Direction:** Market agar niche ja raha hai toh `PE`, upar ja raha hai toh `CE`.
3. **Current Spot Level:** TradingView / Kite par jo current index price chal raha hai woh daalein (jaise `83300`).


* **0-DTE Expiry Warning:** Agar aaj expiry day hai, toh screener warning dega aur poochega: `Switch to Next-Week Expiry? [Y/n]`. Hamesha **`Y`** dabayein taaki theta decay se bacha ja sake.
* Output table mein jiske aage **`SWEET SPOT`** likha ho (`₹200–₹250` SENSEX / `₹61.55–₹76.90` NIFTY), neeche di gayi **`CANDIDATE_STRIKES = [...]`** list copy kar lein.

#### Step 2.2: `config.py` Mein Strikes Paste Karein

VS Code mein `config.py` kholein aur `CANDIDATE_STRIKES = [...]` ko nayi list se replace karke **`Ctrl+S`** (Save) karein.

#### Step 2.3: `tmux` Session Banakar Options Engine Start Karein (10:25 AM)

`tmux` isliye use karte hain taaki agar VS Code ya terminal window galti se close bhi ho jaaye, toh bot background mein chalta rahe:

```bash
tmux new -s algo_sess0
cd ~/trading-projects/option-trailing-algo && source venv/bin/activate
python3 v3_momentum_engine.py

```

* Prompts par bas **`Enter`**, **`Enter`**, **`Enter`** dabate jaayein (default values `config.py` se le lega).
* Screen par dikhega:
* `[BROKER] OpenAlgo WebSocket connected (Zero-429 real-time stream).`
* `[LIVE TICK] SENSEX... LTP: ₹224.50 | Spread: ₹0.40`


* **Options Engine ab 100% live hai!**

---

### Phase 3: Cash Equity Intraday Engine V1.0 Setup (11:20 AM – 11:25 AM)

10:30 AM se 11:30 AM ke beech market apna **Consolidation Box** banata hai. 11:20 AM par hum **usi `tmux` session ke andar ek nayi window (`Window 1`)** kholenge:

1. Keyboard par pehle **`Ctrl+B`** dabayein, phir chhod kar **`C`** dabayein (ek nayi terminal tab khul jayegi).
2. Cash Equity Engine start karein:
```bash
cd ~/trading-projects/option-trailing-algo && source venv/bin/activate
python3 equity_intraday_engine.py

```


3. **Ye Engine Khud Kya Karega (Automatic Magic):**
* **Market Regime Check:** Pehle `SENSEX` (`BSE_INDEX`) ka trend dekhega:
* Agar SENSEX **`+0.30%` ya zyada upar** hai $\rightarrow$ **BULLISH Mode** (`LONG` / Buy only) aur Sensex 30 ke **Top 3 Gainers** chusega.
* Agar SENSEX **`-0.30%` ya zyada niche** hai $\rightarrow$ **BEARISH Mode** (`SHORT` / Sell only) aur Sensex 30 ke **Top 3 Losers** chusega.
* Agar SENSEX **`-0.30%` se `+0.30%` ke beech sideways** hai $\rightarrow$ **`SIDEWAYS (Standing Down)`** bolkar ruk jayega taaki chop market mein loss na ho.


* **VWAP + Box Hydration:** Subah `09:15 AM` se ab tak ka Session VWAP aur `10:30–11:30 AM` ka `Base High / Base Low` automatic calculate karega.



---

### Phase 4: Live Trading Window (11:30 AM – 01:30 PM) — Aapko Kya Karna Hai?

**Kuch nahi! Sirf monitor karein.** Dono engines ke internal rules is tarah kaam karenge:

| Feature | Options Engine (`v3_momentum_engine.py`) | Cash Equity Engine (`equity_intraday_engine.py`) |
| --- | --- | --- |
| **Entry Condition** | 3-min candle `Base High` ke upar close ho + Spread `<= ₹1.00` + `<= 5%` Chase Guard. | **LONG:** Close `> Base High` & `> VWAP` + `1.5x` Vol Surge.<br>

<br>**SHORT:** Close `< Base Low` & `< VWAP` + `1.5x` Vol Surge. |
| **Position Sizing** | Fixed **1 Lot** (`₹4,000 – ₹5,000` capital). | Dynamic Qty: `int(400 / risk_per_share)` (Max `₹25,000` notional). |
| **Initial Stop-Loss** | `Base Low` (Max `₹500` risk cap: `25 pts` SENSEX / `7.7 pts` NIFTY). | `Base Low/High` (Max `0.8%` risk cap = strictly `₹400` risk). |
| **Stage 1 (`1.5R`)** | SL automatic **Net Cost (Entry + Charges)** par aa jayega (Zero-loss trade). | SL automatic **Net Cost (Entry ± Charges)** par aa jayega. |
| **Stage 2 (`2.5R`)** | SL **`+1R` (`+₹500` profit)** par lock ho jayega. | SL **`+1R` (`+₹400` profit)** par lock + 2-candle swing & `9 EMA` trail chalu. |
| **Big Move Trail** | `+50%` profit par 2-candle low trail + `9 EMA` exit + 1-candle Climax Wick. | `4.0R` target par direct profit booking. |
| **No-Trade Cutoff** | `01:30 PM` tak breakout nahi aaya toh engine khud band ho jayega. | `01:30 PM` tak breakout nahi aaya toh engine khud band ho jayega. |
| **EOD Square-Off** | Open position **`03:15 PM`** par automatic close. | Open position **`03:10 PM`** par automatic close (MIS penalty se pehle). |

---

### Phase 5: Post-Market Review (03:20 PM Ke Baad)

Market band hone ke baad dono CSV logs check karein ki aaj ke trades kaise rahe:

```bash
cd ~/trading-projects/option-trailing-algo
cat v3_paper_trades_log.csv
cat equity_paper_trades_log.csv

```

Saath hi OpenAlgo browser dashboard par **Positions** tab mein confirm karein ki koi bhi open position baaki nahi hai.

---

## Quick `tmux` Cheat-Sheet (Keyboard Shortcuts)

* **Options Window (`Window 0`) par jaane ke liye:** `Ctrl+B` dabayein, phir `0` dabayein.
* **Equity Window (`Window 1`) par jaane ke liye:** `Ctrl+B` dabayein, phir `1` dabayein.
* **Terminal se bahar nikalna (bots ko background mein chalta chhod kar):** `Ctrl+B` dabayein, phir `D` dabayein (`[detached]`).
* **Wapas andar aane ke liye:** `tmux attach -t algo_sess0`
* **Emergency Stop (kisi bhi bot ko turant rokne ke liye):** Us window mein jaakar **`Ctrl+C`** dabayein (WebSocket cleanly disconnect ho jayega).

```

---

### Commit & Push This Final Doc on `dev`

Save `docs/BEGINNER_HINGLISH_EXECUTION_GUIDE.md` in VS Code and push it to `dev`:

```bash
cd ~/trading-projects/option-trailing-algo
git add docs/BEGINNER_HINGLISH_EXECUTION_GUIDE.md
git commit -m "Docs: Add beginner-friendly step-by-step Hinglish execution guide for Options V3.4 & Equity V1.0"
git push

```

Have a great rest of the Guru Nanak Jayanti holiday—everything is clean, tested, and ready for tomorrow's 09:05 AM start!
