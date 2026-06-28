# 🚀 Deployment Guide — GitHub + Render

**Bullish Pattern Scanner Backend** ko Render pe deploy karne ka complete step-by-step guide.

**Designed & developed by Shrey Halba**

---

## 📋 Prerequisites (Pehle Ye Chahiye)

1. **GitHub account** — https://github.com (free)
2. **Render account** — https://render.com (free tier works, signup with GitHub)
3. **Backend files** — `backend/` folder (main.py, nse500.py, requirements.txt)
4. **Git installed** on your PC — https://git-scm.com

---

## STEP 1: GitHub Repository Banao

### Option A — Browser se (easiest)

1. GitHub pe login karo → https://github.com/new
2. **Repository name**: `bullish-scanner` (ya jo bhi chaho)
3. **Description**: `Bullish Pattern Scanner — FastAPI + yfinance + pandas-ta`
4. **Public** select karo (taaki Render free tier pe deploy kar sake)
5. ✅ **Add a README file** — tick karo
6. **Create repository** button dabao

### Option B — Git CLI se

```bash
# PC pe naya folder banao
mkdir bullish-scanner
cd bullish-scanner
git init

# Backend files copy karo (zip se extract karke)
# (main.py, nse500.py, requirements.txt, render.yaml ko `backend/` folder me rakho)

git add .
git commit -m "Initial commit — Bullish Pattern Scanner backend"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/bullish-scanner.git
git push -u origin main
```

---

## STEP 2: Repository Structure (Aisa Hona Chahiye)

GitHub pe tumhare repo ka structure aisa hona chahiye:

```
bullish-scanner/
├── backend/
│   ├── main.py              # FastAPI app
│   ├── nse500.py            # NSE 500 stocks list
│   └── requirements.txt     # Python dependencies
├── render.yaml              # Render Blueprint (auto-config)
└── README.md
```

> **Important:** `main.py`, `nse500.py`, `requirements.txt` — teeno files `backend/` folder ke ANDAR hone chahiye. `render.yaml` root me.

---

## STEP 3: Render Pe Deploy Karo

### Method 1 — Blueprint se (Easiest, auto-config)

1. https://dashboard.render.com pe jao
2. **New +** → **Blueprint**
3. Apna GitHub repo select karo (`bullish-scanner`)
4. Render automatically `render.yaml` detect kar lega
5. **Apply** button dabao
6. Render 2-3 min me deploy kar dega
7. URL milega: `https://bullish-scanner-api.onrender.com`

### Method 2 — Manual Web Service

1. https://dashboard.render.com pe jao
2. **New +** → **Web Service**
3. **Build and deploy from a Git repository** → **Next**
4. Apna GitHub repo connect karo (`bullish-scanner`)

5. **Settings fill karo:**

   | Field | Value |
   |-------|-------|
   | **Name** | `bullish-scanner-api` (ya jo bhi chaho) |
   | **Runtime** | `Python 3` |
   | **Root Directory** | `backend` ⚠️ (IMPORTANT — `backend` likho) |
   | **Build Command** | `pip install -r requirements.txt` |
   | **Start Command** | `uvicorn main:app --host 0.0.0.0 --port $PORT` |
   | **Instance Type** | `Free` |

6. **Advanced Settings (optional):**
   - **Environment Variables**: `PYTHON_VERSION` = `3.12.0`

7. **Create Web Service** button dabao

8. **Wait 3-5 minutes** — Render install karega dependencies + start karega server

---

## STEP 4: Deploy Verify Karo

### 4.1 — Logs check karo

Render dashboard me **Events** tab me dekho — ye messages aane chahiye:

```
==> Detected Python app
==> Running build command 'pip install -r requirements.txt'...
==> Successfully installed fastapi-0.115.6 uvicorn-0.34.0 ...
==> Running start command 'uvicorn main:app --host 0.0.0.0 --port $PORT'...
==> Your service is live 🎉
```

### 4.2 — Health check

Browser me apna URL kholo + `/` add karo:

```
https://bullish-scanner-api.onrender.com/
```

Ye JSON response aana chahiye:

```json
{
  "status": "ok",
  "service": "Bullish Pattern Scanner",
  "endpoints": ["/scan", "/patterns", "/chart/{symbol}"],
  "patterns": ["Cup and Handle", "Inverse Head & Shoulders", ...]
}
```

### 4.3 — Scan test karo

Browser me:

```
https://bullish-scanner-api.onrender.com/scan?timeframe=1D&min_confidence=70
```

30-60 second wait karo (first request pe cold start) — phir JSON me bullish stocks aayenge.

### 4.4 — NSE 500 test

```
https://bullish-scanner-api.onrender.com/scan?timeframe=1D&min_confidence=70&universe=nse500
```

60-90 second wait karo — 435 stocks scan honge.

---

## STEP 5: Flutter App Me URL Update Karo

Ab tumhara backend live hai! Flutter app me ye URL daalo:

### 5.1 — `main.dart` me URL badlo

File: `frontend/lib/main.dart`, **line 16**:

```dart
// Pehle:
const String _baseUrl = 'https://your-app-name.onrender.com';

// Ab (tumhara URL):
const String _baseUrl = 'https://bullish-scanner-api.onrender.com';
```

### 5.2 — APK build karo

```bash
cd frontend
flutter create bullish_scanner_app
cd bullish_scanner_app
cp ../lib/main.dart lib/main.dart
cp ../lib/patterns.dart lib/patterns.dart
cp ../pubspec.yaml pubspec.yaml
cp -r ../assets assets

flutter pub get
flutter build apk --release
```

APK milega: `build/app/outputs/flutter-apk/app-release.apk`

### 5.3 — Phone pe install

```bash
adb install build/app/outputs/flutter-apk/app-release.apk
```

Ya APK file ko phone me copy karke tap karo → Install.

---

## 🔄 Auto-Deploy Setup

Render **auto-deploy** support karta hai — jab bhi tum GitHub pe code push karoge, Render automatically rebuild karega.

1. Render dashboard → tumhari service → **Settings**
2. **Auto-Deploy** section me **Yes** selected hona chahiye
3. Ab `git push` karte hi naya version live ho jayega

**Code update example:**
```bash
# main.py me kuch change karo
git add .
git commit -m "Added new pattern filter"
git push
# Render automatically rebuild karega (2-3 min)
```

---

## 🐛 Common Problems + Solutions

### Problem 1: "Application failed to bind to $PORT"

**Solution:** Start command me `--port $PORT` hona chahiye:
```
uvicorn main:app --host 0.0.0.0 --port $PORT
```
(NOT `--port 10000` — Render dynamically port assign karta hai)

---

### Problem 2: "ModuleNotFoundError: No module named 'nse500'"

**Solution:** `Root Directory` = `backend` set karo (Settings me). Agar `nse500.py` `backend/` ke andar hai to import work karega.

---

### Problem 3: "yfinance returns 404/empty on Render"

**Cause:** Yahoo Finance kabhi kabhi Render ke cloud IPs block karta hai.

**Solution:** Backend me already **mock fallback** hai — agar Yahoo fail ho to deterministic mock data return hota hai. Response me `"source": "mock"` dikhega. Tum actual trading ke liye Alpha Vantage ya Finnhub jaisa paid API use kar sakte ho (requirements.txt me `alpha_vantage` add karke).

---

### Problem 4: "Free tier sleeps after 15 min"

**Cause:** Render free tier idle pe sleep mode me chala jata hai. First request pe 30 second cold start.

**Solution 1:** Ugrade to **Starter plan** ($7/month) — no sleep.
**Solution 2:** Free tier me rehne ke liye, external service (cron-job.org, uptime-robot) se har 10 min me ping karo:
```
GET https://bullish-scanner-api.onrender.com/
```

---

### Problem 5: "Build timeout / out of memory"

**Solution:** `pandas-ta` build heavy hai. Agar build fail ho to:
1. Render dashboard → **Settings** → **Instance Type** → **Starter** ($7/mo, 512MB RAM)
2. Ya `requirements.txt` me `pandas-ta==0.3.14b1` ko `pandas-ta==0.3.14b0` se try karo

---

### Problem 6: "CORS error in Flutter"

**Solution:** Backend me already CORS enabled hai (`allow_origins=["*"]`). Agar bhi error aaye to `main.py` me check karo:
```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["GET", "POST", "OPTIONS"],
    allow_headers=["*"],
)
```

---

## 📊 API Endpoints Summary

Deploy hone ke baad ye endpoints available honge:

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/` | Health check |
| GET | `/patterns` | List of 8 patterns + timeframes |
| GET | `/symbols?universe=nse500` | NSE 500 tickers list |
| GET | `/scan?timeframe=1D&min_confidence=70&universe=nifty30` | Scan Nifty 30 |
| GET | `/scan?timeframe=1D&universe=nse500` | Scan NSE 500 (full) |
| GET | `/scan?symbols=RELIANCE,TCS&timeframe=1D` | Scan custom symbols |
| POST | `/scan` | Scan with JSON body (CSV upload) |
| GET | `/scan/csv?timeframe=1D` | Download results as CSV |
| GET | `/chart/RELIANCE?timeframe=1D` | OHLCV candles + indicators |

---

## 💰 Render Free Tier Limits

- **750 hours/month** (enough for 1 always-on service)
- **512 MB RAM, 0.1 CPU**
- **Sleeps after 15 min idle** (30s cold start on wake)
- **Custom domain** not available (use `.onrender.com` subdomain)

Agar serious use karna ho to **Starter plan ($7/month)** me:
- No sleep
- 512 MB RAM, 0.5 CPU
- Custom domain support
- Faster builds

---

## ✅ Quick Deploy Checklist

- [ ] GitHub repo bana (`bullish-scanner`)
- [ ] `backend/` folder me `main.py`, `nse500.py`, `requirements.txt` push kiya
- [ ] `render.yaml` root me push kiya
- [ ] Render pe **New Web Service** ya **Blueprint** create kiya
- [ ] `Root Directory` = `backend` set kiya
- [ ] Build command = `pip install -r requirements.txt`
- [ ] Start command = `uvicorn main:app --host 0.0.0.0 --port $PORT`
- [ ] Deploy successful (logs me "live 🎉" dikh raha)
- [ ] `https://your-app.onrender.com/` pe health check JSON aaya
- [ ] `/scan?timeframe=1D` pe results aaye
- [ ] Flutter app me `_baseUrl` update kiya
- [ ] APK build karke phone pe install kiya

**Done! 🎉 Ab tumhara app live hai!**
