Good catch closing the position manually—that was an invaluable live forward-test discovery.

### Root Cause Audit: Why Fyers Hit `429 Too Many Requests` After Running Smoothly for 2 Hours

Look closely at the exact Fyers URL in that traceback:
`[https://api-t1.fyers.in/data/depth?symbol=BSE%3ASENSEX26NOV82000PE&ohlcv_flag=1](https://api-t1.fyers.in/data/depth?symbol=BSE%3ASENSEX26NOV82000PE&ohlcv_flag=1)`

1. **Daily Rate-Limit Bucket on Fyers `/data/depth`:** Fyers enforces both a **per-second (`1 req/sec`)** limit and a **per-minute (`100–200 req/min`) / daily REST quota** on `/data/depth`. Even at `1.5s` intervals (`40 calls/min`), running continuously for ~3 hours (`~7,200` depth calls) + `2x` retry bursts exhausted Fyers' daily/rolling `/data/depth` quota!
2. **Stale Cache Blindness (`V3.1` Bug):** When `get_quote()` returned `None` after failing retries, `v3_momentum_engine.py` kept running on the frozen `last_quote` (`275.20`) instead of triggering a **Data-Feed Circuit Breaker** or falling back to a lighter quote endpoint.

---

### Complete V3.2 Upgrade Package (Fixes All 3 Items at Once)

Let's inspect what methods your `openalgo` SDK exposes (`dir(c)`) so we can check if OpenAlgo has a lighter LTP/quote endpoint or WebSocket feed alongside `/quotes`—run this 1-line check first:

```bash
cd ~/trading-projects/option-trailing-algo && source venv/bin/activate
python3 -c 'import config; from openalgo import api; c = api(api_key=config.OPENALGO_API_KEY, host=config.OPENALGO_HOST); print([m for m in dir(c) if not m.startswith("_")])'

```

As soon as you paste that method list, I will give you the clean, one-shot **V3.2 upgrade** that permanently locks in:

1. **Zero-429 Quote Architecture:** Uses OpenAlgo's lightest quote/WebSocket path + **`3.0s` polling interval** (`20 calls/min` = `3,600` calls across an entire 3-hour session, well under Fyers' daily quota) with zero retry storms (`1` attempt per cycle, immediate `429` backoff).
2. **Stale-Quote Safety Circuit Breaker (`MAX_CONSECUTIVE_QUOTE_FAILURES = 8`):** If live quotes fail for `~25–30 seconds`, the engine loudly alerts—and if `in_trade`, **auto-exits at Market** so a dead/rate-limited feed can never leave an open position unmanaged.
3. **Your `₹4,000 – ₹5,000` Capital-Normalized Band (`4000 / qty` to `5000 / qty`):** Automatically sets `₹200.00–₹250.00` for SENSEX (`20 qty`) and `₹61.55–₹76.90` for NIFTY (`65 qty`) across `config.py`, `strike_screener.py`, and `v3_momentum_engine.py`.
4. **Live `BASE OBSERVE` Heartbeat:** Prints a clean live tick indicator between 3-minute candle closes so you always see active feed confirmation.
