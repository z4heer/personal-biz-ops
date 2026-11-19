Yes, we are completely set for today! Your **Python $\leftrightarrow$ OpenAlgo $\leftrightarrow$ Fyers** pipeline is live, verified, and running in **Analyze Mode**.

### Quick Checklist for Tomorrow Morning (Friday, Nov 20)

1. **Before 10:30 AM IST:** Start OpenAlgo (`python app.py`), log in via **Login with Fyers** at `[http://127.0.0.1:5000](http://127.0.0.1:5000)`, and verify the purple **Analyze** badge is active.
2. **Confirm WSL Timezone & Option Symbol:** Double-check `date` shows IST and grab your exact `BFO`/`NFO` option contract symbol via `c.search(...)`.
3. **Launch & Observe:** Run `python v3_momentum_engine.py` (at 10:30 AM for 4-candle base lock + 11:30 AM–1:30 PM execution) or `python strategy_engine.py` (at 11:30 AM).

See you tomorrow during the trading session—have a great evening!
---

### Step 1: `openalgo/.env` File Update Karein

WSL terminal mein `~/trading-projects/openalgo` folder ke andar `.env` file open karein:

```bash
cd ~/trading-projects/openalgo
nano .env

```

Isme Fyers ke credentials aur required keys is format mein update karein (purane Sahi/dummy keys ko replace karke):

```env
# Broker Configuration
VALID_BROKERS = 'fyers'

# Fyers Credentials (MyAPI portal se mila hua App ID aur Secret)
BROKER_API_KEY = 'AAPKA_FYERS_APP_ID'
BROKER_API_SECRET = 'AAPKA_FYERS_SECRET_KEY'
REDIRECT_URL = 'http://127.0.0.1:5000/fyers/callback'

# OpenAlgo Security & Host Settings (Agar pehle se hain toh rehne dein)
APP_KEY = 'koi_bhi_random_32_char_secret_string_yahan_rakhein'
API_KEY_PEPPER = 'koi_bhi_random_pepper_string_yahan_rakhein'
HOST_SERVER = 'http://127.0.0.1:5000'
NGROK_ALLOW = 'FALSE'

```

*(Note: Fyers mein `App ID` generally `XXXXXXXXXX-100` format mein hota hai—poora string including `-100` dalega).*
Save karne ke liye `Ctrl + O`, `Enter`, aur exit karne ke liye `Ctrl + X` press karein.

---

### Step 2: OpenAlgo Server Start Karein Aur Fyers Login Karein

1. Terminal mein OpenAlgo virtual environment activate karke server start karein:
```bash
cd ~/trading-projects/openalgo
source venv/bin/activate
python app.py

```


2. Apne Windows browser mein open karein:
**`[http://127.0.0.1:5000](http://127.0.0.1:5000)`**
3. Pehli baar open karne par OpenAlgo ka local admin user/password create karein aur login karein.
4. Dashboard par **"Login with Fyers"** button par click karein $\rightarrow$ apna Fyers Client ID, TOTP (Google Authenticator), aur 4-digit PIN daal kar OAuth login complete karein.
*(Jab pehli baar Fyers login hota hai, OpenAlgo automatically NSE/BSE/NFO/BFO master contracts download karta hai—isliye 15–30 seconds wait karein jab tak dashboard par "Connected" status na dikh jaye).*

---

### Step 3: Analyze Mode ON Karein Aur Local API Key Copy Karein

1. OpenAlgo dashboard ke top bar mein **Mode** toggle ko **Analyze Mode** (Paper Trading) par switch karein (confirm karein ki live mode OFF hai).
2. Left menu mein **API Key** section par jayein, apni **OpenAlgo Local API Key** generate/copy karein.

---

### Step 4: Apne Algo (`config.py`) Ko Fyers + OpenAlgo Se Connect Karein

Ek **doosra WSL terminal** kholein aur `~/trading-projects/option-trailing-algo/config.py` update karein:

```bash
cd ~/trading-projects/option-trailing-algo
nano config.py

```

Sirf yeh 2 lines update karein:

```python
MODE = "OPENALGO"
OPENALGO_API_KEY = "YAHAN_OPENALGO_DASHBOARD_KI_API_KEY_PASTE_KAREIN"

```

---

### Step 5: Live Quote & Symbol Format Check (Even After Market Hours!)

Abhi **7:06 PM IST** ho rahe hain (equity/index market closed hai), lekin aap abhi bhi **API connection aur last traded price (`LTP`) fetch** test kar sakte hain! Terminal mein yeh quick one-liner run karke dekhein:

```bash
cd ~/trading-projects/option-trailing-algo
source venv/bin/activate
python -c "import config; from openalgo import api; c = api(api_key=config.OPENALGO_API_KEY, host=config.OPENALGO_HOST); print(c.quotes(symbol='NIFTY26NOV2624500CE', exchange='NFO'))"

```

*(Ya OpenAlgo dashboard ke **Symbol Search / Playground** tab mein jaakar exact active SENSEX / NIFTY weekly/monthly strike symbol verify kar lein).*

Steps 1 se 4 complete karke batayein—jaise hi OpenAlgo dashboard par Fyers login aur Analyze Mode green ho jaye, hum agla step check karenge!
