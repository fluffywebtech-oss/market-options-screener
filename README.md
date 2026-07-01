# Real-Time Options Screener (Upstox)

Live NSE option chain / stock screener. Any NSE stock or index is searchable:
- **F&O symbols** (NIFTY, RELIANCE, ZOMATO, ...) show the real option chain with
  live Call/Put LTP, OI, IV on a uniform strike grid.
- **Cash-only stocks** (e.g. IRCTC) show the live spot price plus an
  intrinsic-value strike grid.

## Architecture

| Environment | Data path |
|-------------|-----------|
| **Deployed (Vercel)** | Browser → `/api/chain` (Python serverless) → Upstox REST API |
| **Local dev** | Browser → `ws://localhost:8765` (`upstox_bridge.py`) → Upstox |

The frontend (`index.html`) auto-detects the host: it polls `/api/chain` when
served from a real domain, and uses the local WebSocket bridge on `localhost`.

## Deploy to Vercel

1. Import this repo into Vercel.
2. Add an Environment Variable:
   - `UPSTOX_ACCESS_TOKEN` = your daily Upstox access token.
3. Deploy. Open the site — it starts polling live data automatically.

### Important Upstox requirements
- **Disable the static-IP allowlist** on your Upstox API app. Vercel functions
  run from dynamic IPs; if the allowlist is on you'll get `UDAPI1221` / 403.
- The **access token expires daily** — update the `UPSTOX_ACCESS_TOKEN` env var
  (and redeploy) each day, or wire up an OAuth refresh flow.
- Data only ticks during NSE market hours (~09:15–15:30 IST); otherwise it
  shows the last traded price.

## Local development

```bash
pip install websockets upstox-python-sdk
export UPSTOX_ACCESS_TOKEN="your_token"
python upstox_bridge.py
```

Then open `index.html` in a browser and click **Connect to Upstox Bridge**.

## Endpoint

```
GET /api/chain?symbol=NIFTY[&expiry=2026-07-07]
```

Returns JSON: `{ type, symbol, spot, expiry, expiries[], rows[] }`.
