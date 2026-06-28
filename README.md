# Bullish Pattern Scanner — Flutter + Python (FastAPI)

Scans **NSE 500 stocks** for **8 bullish chart patterns** (Cup & Handle, Inverse Head & Shoulders, Double Bottom, Bull Flag, Ascending Triangle, Rounding Bottom, Symmetrical Triangle, Rectangle) using real Yahoo Finance data and returns only the **bullish** ones as JSON.

This is a port of your existing Next.js scanner logic to:
- **Backend** → Python FastAPI + `yfinance` + `pandas-ta`, deployed on **Render** (free tier works)
- **Frontend** → Flutter app (Android APK ready), consumes the Render API via HTTP GET/POST

**Designed & developed by Shrey Halba**

---

## 📁 Project Structure

```
flutter-python-scanner/
├── backend/
│   ├── main.py             # FastAPI app — all 8 pattern detectors + yfinance fetch
│   ├── nse500.py           # NSE 500 stocks list (435 tickers)
│   ├── requirements.txt    # Python deps (fastapi, yfinance, pandas-ta, ...)
│   └── README.md           # Render deployment guide
└── frontend/
    ├── lib/
    │   └── main.dart       # Complete Flutter app — NSE500, CSV upload/download, footer
    ├── assets/
    │   └── app_icon.png    # App icon (1024x1024, generated)
    ├── pubspec.yaml        # Flutter deps (http, file_picker, share_plus, ...)
    └── README.md           # APK build guide
```

> **Note:** `frontend/` only contains the *source* files. To get a buildable Flutter project, run `flutter create .` first, then drop these files in. See the **Build APK** section below.

---

## 🚀 Quick Start (TL;DR)

1. **Deploy backend on Render** (5 min)
   - New Web Service → connect repo → root = `backend/`
   - Build: `pip install -r requirements.txt`
   - Start: `uvicorn main:app --host 0.0.0.0 --port $PORT`
   - Note the URL: `https://your-app.onrender.com`

2. **Update Flutter app with the Render URL**
   - Edit `frontend/lib/main.dart`, line 16:
     ```dart
     const String _baseUrl = 'https://your-app-name.onrender.com';
     ```

3. **Build APK**
   ```bash
   # One-time: scaffold a full Flutter project (creates android/, ios/, test/, ...)
   flutter create bullish_scanner
   cd bullish_scanner

   # Replace the default source with our files
   cp ../frontend/lib/main.dart lib/main.dart
   cp ../frontend/pubspec.yaml pubspec.yaml

   # Install deps + build release APK
   flutter pub get
   flutter build apk --release
   ```
   APK at: `build/app/outputs/flutter-apk/app-release.apk`

4. **Install on phone**
   ```bash
   adb install build/app/outputs/flutter-apk/app-release.apk
   ```
   Or copy the APK to your phone and tap to install.

---

## 🔌 API Contract

### `GET /scan`

Query params (all optional):
| Param           | Default          | Description                          |
|-----------------|------------------|--------------------------------------|
| `symbols`       | *(none)*         | Comma-separated tickers (overrides universe) |
| `universe`      | `nifty30`        | `nifty30` or `nse500` (full NSE 500) |
| `timeframe`     | `1D`             | `5m\|15m\|1h\|4h\|1D\|1W\|1M`        |
| `patterns`      | all 8            | Comma-separated pattern names        |
| `min_confidence`| `50`             | 0–100                                |

**Examples:**
```
GET /scan?timeframe=1D&min_confidence=70&universe=nse500
GET /scan?timeframe=1D&symbols=RELIANCE,TCS,INFY
```

### `POST /scan`
For large symbol lists (e.g. CSV upload). Body:
```json
{
  "symbols": ["RELIANCE", "TCS", "INFY"],
  "timeframe": "1D",
  "min_confidence": 60
}
```

### `GET /scan/csv`
Same as GET /scan but returns a downloadable CSV file.
```
GET /scan/csv?timeframe=1D&min_confidence=70&universe=nse500
```

### `POST /scan/csv`
CSV upload + CSV download combo. POST body like `/scan`, returns CSV.

### `GET /symbols`
Returns the ticker list for a universe.
```
GET /symbols?universe=nse500   →  { "count": 435, "symbols": [...] }
GET /symbols?universe=nifty30  →  { "count": 30, "symbols": [...] }
```

### `GET /`
Health check. | `GET /patterns` — list supported patterns + timeframes. | `GET /chart/{symbol}` — raw OHLCV candles.`

**Response:**
```json
{
  "scannedAt": 1700000000000,
  "durationMs": 8420,
  "scannedSymbols": 30,
  "scannedTimeframes": 1,
  "totalCombinations": 240,
  "source": "yfinance",
  "realDataCount": 12,
  "mockDataCount": 0,
  "results": [
    {
      "id": "RELIANCE-1D-Cup_and_Handle",
      "symbol": "RELIANCE",
      "name": "Reliance Industries",
      "exchange": "NSE",
      "pattern": "Cup and Handle",
      "timeframe": "1D",
      "rsi": 62.5,
      "rsiStatus": "Bullish",
      "volumeSpike": 1.8,
      "volumeStatus": "Above Avg",
      "status": "BREAKOUT",
      "confidence": 85,
      "lastPrice": 2920.5,
      "breakoutLevel": 2945.5,
      "supportLevel": 2850.0,
      "targetPrice": 3181.14,
      "stopLoss": 2793.0,
      "description": "U-shaped cup with handle compression...",
      "source": "yfinance",
      "detectedAt": 1700000000000
    }
  ]
}
```

Other endpoints:
- `GET /` — health check
- `GET /patterns` — list supported patterns + timeframes
- `GET /chart/{symbol}?timeframe=1D` — raw OHLCV candles + indicators (for charting)

---

## 🐍 Backend — Deploy on Render

### Step 1 — Push to GitHub
Push the `backend/` folder to a GitHub repo (Render reads from GitHub).

### Step 2 — Create Web Service on Render
1. Go to <https://dashboard.render.com> → **New +** → **Web Service**
2. Connect your GitHub repo
3. Settings:
   - **Name**: `bullish-scanner` (or anything)
   - **Runtime**: Python 3
   - **Root Directory**: `backend` *(important — if your repo root has both folders)*
   - **Build Command**: `pip install -r requirements.txt`
   - **Start Command**: `uvicorn main:app --host 0.0.0.0 --port $PORT`
   - **Instance Type**: Free (sleeps after 15 min idle, ~30s cold start)

4. Click **Create Web Service**. Wait ~3 min for first deploy.

5. Test it: visit `https://your-app.onrender.com/` — should return JSON health check.

### Step 3 — Test the scan endpoint
```
https://your-app.onrender.com/scan?timeframe=1D&min_confidence=60
```

> **Free tier tip:** First request after idle takes ~30s (cold start). The Flutter app has a 60s timeout to handle this gracefully.

### Local testing (optional)
```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --reload --port 10000
# Visit http://localhost:10000/scan?timeframe=1D
```

---

## 📱 Frontend — Build the APK

### Prerequisites
- Flutter SDK 3.10+ → <https://docs.flutter.dev/get-started/install>
- Android Studio (for SDK + build tools) OR just command-line tools
- A phone with "Install from unknown sources" enabled

### Step 1 — Update backend URL
Edit `frontend/lib/main.dart`, **line 16**:
```dart
const String _baseUrl = 'https://your-app-name.onrender.com';
```

### Step 2 — Install deps
```bash
cd frontend
flutter pub get
```

### Step 3 — Build release APK
```bash
flutter build apk --release
```

Output:
```
build/app/outputs/flutter-apk/app-release.apk
```

### Step 4 — Install on phone
**Option A — ADB (USB debug):**
```bash
adb install build/app/outputs/flutter-apk/app-release.apk
```

**Option B — Manual:**
1. Copy the APK to your phone (Drive, email, USB).
2. Open it in Files app → tap Install.
3. If prompted, allow "Install from unknown sources".

### Build for App Bundle (Play Store)
```bash
flutter build appbundle --release
# Output: build/app/outputs/bundle/release/app-release.aab
```

---

## 🎨 Flutter App Features

- ✅ **Loading state** with animated "Scanning..." indicator
- ✅ **State management** via `ChangeNotifier` + `ListenableBuilder` (no extra deps)
- ✅ **Clean ListView** of bullish stocks, sorted by confidence
- ✅ **Pull-to-refresh** + dedicated Scan FAB
- ✅ **Filters**: timeframe, min confidence, status (CONFIRMED/BREAKOUT/FORMING/WATCH)
- ✅ **Detail bottom sheet** with full analysis per stock
- ✅ **Risk:Reward** calculation per pattern
- ✅ **Material 3** with seed color, light + dark theme
- ✅ **Error handling** with retry (Render cold start aware)
- ✅ **Source badge** (yfinance / mock / mixed) so you know if data is real

---

## 🧪 Local Dev Workflow

Run backend locally + Flutter against it:
```bash
# Terminal 1 — backend
cd backend
uvicorn main:app --reload --port 10000

# Terminal 2 — Flutter (Android emulator)
cd frontend
# Set _baseUrl = 'http://10.0.2.2:10000' in main.dart for emulator
# (10.0.2.2 maps to host machine's localhost from the Android emulator)
flutter run
```

For a physical phone on the same Wi-Fi as your laptop:
```bash
# Find your laptop's LAN IP (e.g. 192.168.1.10)
# Set _baseUrl = 'http://192.168.1.10:10000'
# Run uvicorn on all interfaces:
uvicorn main:app --host 0.0.0.0 --port 10000
flutter run
```

---

## 🐛 Troubleshooting

| Problem | Fix |
|---|---|
| Flutter shows "Request timed out" | Render free tier cold-start (~30s). Tap Retry. Or upgrade to paid tier. |
| 0 results returned | Lower min_confidence to 50 in filters, or try `1D` timeframe. |
| `yfinance` returns empty on Render | Yahoo sometimes blocks cloud IPs. The backend auto-falls back to mock data so you'll still get results, but the `source` field will say `mock`. |
| `pandas_ta` install fails | Pin `numpy<2` if needed: `numpy==1.26.4` |
| APK install blocked on phone | Enable "Install unknown apps" for your file manager in Android Settings → Apps. |
| `flutter build apk` fails | Run `flutter doctor` first. Make sure Android SDK + build tools are installed. |
| Hot reload not picking backend changes | Backend is on Render — push to GitHub, Render auto-deploys. |

---

## 📦 Dependencies Summary

### Python (`requirements.txt`)
- `fastapi` — web framework
- `uvicorn[standard]` — ASGI server
- `yfinance` — Yahoo Finance data fetcher (replaces your TS fetch logic)
- `pandas` — DataFrame for OHLCV manipulation
- `pandas-ta` — RSI, volume indicators (replaces your `computeRSI` / `computeVolumeSpike`)
- `pydantic` — response models
- `requests` — used internally by yfinance

### Flutter (`pubspec.yaml`)
- `http` — HTTP GET requests to Render backend
- `cupertino_icons` — iOS-style icons (Flutter default)
- That's it — state management uses Flutter's built-in `ChangeNotifier` (no extra dep needed)

---

## 📄 License

Your existing project license applies. Pattern detection logic is a faithful Python port of your original TypeScript `detector.ts`.
