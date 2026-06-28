# Backend — Bullish Pattern Scanner (FastAPI)

## Files
- `main.py` — FastAPI app with all 8 pattern detectors + yfinance fetcher
- `requirements.txt` — pinned Python deps

## Run locally
```bash
pip install -r requirements.txt
uvicorn main:app --reload --port 10000
```
Visit <http://localhost:10000/scan?timeframe=1D&min_confidence=60>

## Deploy on Render
1. Push `backend/` to GitHub
2. Render → New Web Service → Python 3
3. Root Directory: `backend`
4. Build: `pip install -r requirements.txt`
5. Start: `uvicorn main:app --host 0.0.0.0 --port $PORT`
6. Wait ~3 min → test `https://your-app.onrender.com/`

## Endpoints
- `GET /` — health
- `GET /patterns` — supported patterns + timeframes
- `GET /scan?symbols=RELIANCE,TCS&timeframe=1D&min_confidence=60&patterns=Cup%20and%20Handle`
- `GET /chart/RELIANCE?timeframe=1D` — raw OHLCV candles

## Notes
- Yahoo Finance sometimes blocks cloud IPs (Render). The backend auto-falls back to deterministic mock data, and the `source` field in the response tells you which (`yfinance`, `mock`, or `mixed`).
- Render free tier sleeps after 15 min idle → first request takes ~30s cold start. Flutter app has a 60s timeout.
- In-process cache (60s TTL) reduces Yahoo API calls.
