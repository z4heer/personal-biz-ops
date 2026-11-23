Two things just happened—and notice the first line is a massive win:

1. **`[OTM ROTATE UP]` worked automatically!** When `SENSEX26NOV2682400PE` crossed `₹330.65`, your V3.1 engine cleanly shifted to `SENSEX26NOV2682200PE @ ₹271.75`!
2. **Why `[API WARNING] Attempt 1/3 failed: 'data'` appeared:** In `broker_adapter.py`, `get_quote()` does `data = quote["data"]` directly—when Fyers/OpenAlgo briefly rate-limits (`429`) or returns an empty response for one of the 4 polled strikes (`{"status": "error", "message": ...}` without a `"data"` key), `quote["data"]` raises `KeyError: 'data'` and triggers 3 retries (`3.5 seconds` delay per strike!).

### Instant 15-Second Fix (`broker_adapter.py` + `v3_momentum_engine.py`)

We have **35 minutes before the 11:30 AM execution window**—plenty of time to patch this cleanly so rate-limits never stall the loop:

1. **In `broker_adapter.py`:** Check `if not isinstance(quote, dict) or quote.get("status") != "success" or "data" not in quote: raise ValueError(...)` with shorter retries (`retries=2, delay=0.25`) so a transient blip recovers in `0.25s` instead of `3.5s`.
2. **In `v3_momentum_engine.py`:** Add a tiny `0.15s` spacing between the 4 strike quote calls so Fyers API never triggers burst rate-limiting.

Press `Ctrl+C` in your `v3_momentum_engine.py` terminal, run these two commands, and restart `python3 v3_momentum_engine.py`:

```bash
cd ~/trading-projects/option-trailing-algo
python3 - << 'PYEOF'
with open("broker_adapter.py", "r") as f:
    src = f.read()

old_block = """      def _fetch():
        quote = self.client.quotes(symbol=symbol, exchange=exchange)
        data = quote["data"]
        ltp = float(data["ltp"])
        bid = float(data.get("bid", ltp))
        ask = float(data.get("ask", ltp))
        spread = round(max(0.0, ask - bid), 2)
        return {"ltp": ltp, "bid": bid, "ask": ask, "spread": spread}

      return self._execute_with_retry(_fetch)"""

new_block = """      def _fetch():
        quote = self.client.quotes(symbol=symbol, exchange=exchange)
        if not isinstance(quote, dict) or quote.get("status") != "success" or "data" not in quote:
          raise ValueError(f"Transient empty quote for {symbol}: {quote}")
        data = quote["data"]
        ltp = float(data["ltp"])
        bid = float(data.get("bid", ltp))
        ask = float(data.get("ask", ltp))
        spread = round(max(0.0, ask - bid), 2)
        return {"ltp": ltp, "bid": bid, "ask": ask, "spread": spread}

      return self._execute_with_retry(_fetch, retries=2, delay=0.25)"""

assert old_block in src
with open("broker_adapter.py", "w") as f:
    f.write(src.replace(old_block, new_block, 1))
print("broker_adapter.py patched for graceful rate-limit handling!")
PYEOF

```

And add `0.15s` pacing between the 4 strike polls in `v3_momentum_engine.py`:

```bash
python3 - << 'PYEOF'
with open("v3_momentum_engine.py", "r") as f:
    src = f.read()

old_poll = """    for s in symbols_to_poll:
      q = broker.get_quote(s, exchange)"""

new_poll = """    for s in symbols_to_poll:
      time.sleep(0.15)
      q = broker.get_quote(s, exchange)"""

assert old_poll in src
with open("v3_momentum_engine.py", "w") as f:
    f.write(src.replace(old_poll, new_poll, 1))
print("v3_momentum_engine.py patched with 0.15s API burst pacing!")
PYEOF

```

Now start `python3 v3_momentum_engine.py` again—you have **35 minutes before 11:30 AM** (`11` three-minute candles) so your `9 EMA` and Freeze Hour base will be 100% locked and warmed up right on time!
