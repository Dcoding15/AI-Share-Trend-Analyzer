Based on your codebase (FastAPI + Next.js + Ollama) and the detailed academic project report (quantitative methods, case studies), I've synthesized a comprehensive `README.md` that unifies both. The new README accurately reflects your full‑stack architecture, AI/ML features, and the rigorous financial methodology described in the PDF (Fama‑French, technical indicators, Monte Carlo + GARCH, VADER sentiment, composite scoring).

```markdown
# 📈 AI Share Trend Analyzer

A full‑stack, AI‑powered equity research platform that combines **institutional‑grade quantitative analysis** with **modern web technologies**. It delivers fundamental ratios, technical signals, Monte Carlo risk simulation (GBM + GARCH), news sentiment scoring, and LLM‑based price predictions – all through an interactive Next.js dashboard with real‑time WebSocket updates.

> **Target users**: High‑information retail investors, CFA/MBA students, financial educators, and anyone who needs deeper stock analysis without paying for a Bloomberg Terminal.

---

## ✨ Key Features

### 📊 Quantitative Analysis Modules (Academic Grounding)
- **Fundamental Valuation** – P/E, P/B, ROE, dividend yield, market cap, compared to sector medians (Fama‑French factor tradition).
- **Technical Indicators** – SMA crossover (Golden/Death Cross), RSI(14), MACD(12,26,9) with clear buy/sell signals.
- **Monte Carlo Simulation** – 10,000 geometric Brownian motion paths, volatility calibrated via **GARCH(1,1)** (captures volatility clustering). Outputs median forecast, 5th percentile VaR, and upside potential.
- **News Sentiment (NLP)** – VADER lexicon scoring of recent headlines → compound score mapped to 0–100.
- **Composite Score** – Weighted combination (35% fundamental, 30% technical, 25% risk, 10% sentiment) → **Buy (70+)** / **Hold (40–69)** / **Sell (<40)**.

### 🤖 AI & Machine Learning
- **Ollama LLM (llama3.2)** – Generates structured JSON predictions (7d, 14d, 1m, 3m price targets, trend, confidence, summary) based on technical indicators. Falls back gracefully to mock predictions if Ollama is offline.
- **Fallback logic** ensures the dashboard never breaks.

### ⚡ Real‑Time & Interactive
- **WebSocket live prices** – Stream real‑time quotes from Yahoo Finance (via `yfinance.AsyncWebSocket`).
- **Interactive charts** – Historical OHLCV, RSI, MACD, and Monte Carlo fan charts.
- **Stock search** – Autocomplete with symbol, name, exchange.
- **Watchlist & history** – User accounts (NextAuth + Prisma) to save favourite stocks and prediction history.

### 🧩 Full‑Stack Architecture
| Layer               | Technology |
|---------------------|-------------|
| Backend API         | FastAPI (Python), Uvicorn |
| Frontend            | Next.js 15 (App Router), React 19, TypeScript |
| UI Components       | shadcn/ui, Tailwind CSS, Lucide Icons |
| Auth                | NextAuth.js (Auth.js) with PostgreSQL (Prisma) |
| Real‑time           | WebSockets (FastAPI native) + Yahoo Finance WS |
| Data Fetching       | yfinance (historical & fundamentals), News API |
| ML / Quant          | Ollama, arch (GARCH), numpy, scipy, vaderSentiment |

---

## 🚀 Getting Started

### Prerequisites
- **Node.js** 20+ & npm/yarn/pnpm
- **Python** 3.10+
- **PostgreSQL** (or SQLite for local dev)
- **Ollama** (optional, for real LLM predictions) – [install ollama](https://ollama.ai) and pull `llama3.2`

### Backend Setup (FastAPI)

```bash
# Clone the repository
git clone https://github.com/Dcoding15/AI-Share-Trend-Analyzer.git
cd AI-Share-Trend-Analyzer/ai_service

# Create virtual environment
python -m venv venv
source venv/bin/activate      # or venv\Scripts\activate on Windows

# Install dependencies
pip install -r requirements.txt

# (Optional) Start Ollama
ollama pull llama3.2
ollama serve

# Run FastAPI server
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

Backend will be available at `http://localhost:8000`  
API docs: `http://localhost:8000/docs`

### Frontend Setup (Next.js)

```bash
cd ../full_stack

# Install dependencies
npm install

# Environment variables – create .env.local (see below)
# Run database migrations
npx prisma generate
npx prisma db push

# Start development server
npm run dev
```

Frontend runs at `http://localhost:3000`

#### Environment Variables (`.env.local`)

```env
DATABASE_URL="postgresql://user:password@localhost:5432/stockdb"
NEXTAUTH_SECRET="your-secret-key"
NEXTAUTH_URL="http://localhost:3000"
NEXT_PUBLIC_API_URL="http://localhost:8000/api"
NEXT_PUBLIC_WS_URL="ws://localhost:8000/ws/prices"
```

---

## 📡 API Endpoints (Backend)

All endpoints are prefixed with `/api`.

| Method | Endpoint                     | Description |
|--------|------------------------------|-------------|
| GET    | `/stock/{symbol}`            | Current info + 30d prices/volumes |
| GET    | `/stocks/{symbol}/history`   | Full OHLCV history |
| GET    | `/stocks/search?q=`          | Search stocks (autocomplete) |
| GET    | `/stocks/batch?symbols=`     | Batch info for multiple symbols |
| POST   | `/predict`                   | LLM prediction (Ollama) based on indicators |
| POST   | `/predict/stream`            | Streaming text prediction (SSE) |
| WS     | `/ws/prices`                 | WebSocket for real‑time prices |

### WebSocket Usage Example

```javascript
const ws = new WebSocket("ws://localhost:8000/ws/prices");
ws.onopen = () => ws.send(JSON.stringify({ action: "subscribe", symbols: ["RELIANCE.NS", "TCS.NS"] }));
ws.onmessage = (event) => console.log(JSON.parse(event.data));
```

---

## 📁 Project Structure

```text
AI-Share-Trend-Analyzer/
├── ai_service/                     # FastAPI backend
│   ├── main.py                     # App entry, CORS, routers
│   ├── requirements.txt
│   ├── routes/
│   │   ├── prediction.py           # /predict (Ollama + fallback)
│   │   ├── stock_routes.py         # /stock, /stocks/search, /stocks/history, batch
│   │   └── ws_routes.py            # WebSocket /ws/prices
│   └── services/
│       ├── live_price_service.py   # WebSocket manager, Yahoo Finance WS integration
│       ├── prediction_service.py   # Indicator calculations (RSI, MACD, EMA, Bollinger, Stochastic, ATR, volume ratio)
│       └── stock_services.py       # yfinance fetching (single/batch/search/history)
│
└── full_stack/                     # Next.js frontend
    ├── app/
    │   ├── (auth)/login & register
    │   ├── dashboard/              # Protected routes
    │   │   ├── aipredictions/      # AI prediction page
    │   │   ├── explore/            # Stock search & discovery
    │   │   ├── history/            # User prediction history
    │   │   ├── watchlist/          # User's saved stocks
    │   │   └── stock/[symbol]/     # Individual stock detail page
    │   ├── api/                    # Next.js API routes (auth, watchlist, history, predict proxy)
    │   └── components/             # Shared React components
    ├── components/ui/              # shadcn/ui components
    ├── hooks/                      # useLivePrices, useWatchlist
    ├── lib/                        # Prisma client, API client, utils
    ├── prisma/                     # Prisma schema (User, Watchlist, PredictionHistory)
    └── public/                     # Static assets
```

---

## 🔬 Methodology (Academic Foundations)

The platform implements four independent analytical engines, each rooted in peer‑reviewed research:

| Module | Method | Academic Source |
|--------|--------|----------------|
| **Fundamental** | P/E, P/B, ROE, dividend yield vs sector median | Fama & French (1992) – three‑factor model |
| **Technical** | SMA crossover (50/200), RSI(14), MACD(12,26,9) | Brock et al. (1992); Jegadeesh & Titman (1993) |
| **Risk Simulation** | Geometric Brownian Motion + GARCH(1,1) volatility | Bollerslev (1986) |
| **Sentiment** | VADER lexicon (compound score -1 to +1) | Hutto & Gilbert (2014) |

### Composite Scoring
```
Composite = 0.35×Fundamental + 0.30×Technical + 0.25×Risk + 0.10×Sentiment
→ 70+ = Buy, 40–69 = Hold, <40 = Sell
```

> **Case Study Validation**  
> - **Apple Inc. (AAPL)** – Composite 78 → **BUY** (Golden Cross, strong fundamentals, positive sentiment)  
> - **Reliance Industries (RELIANCE.NS)** – Composite 62 → **HOLD** (Death Cross, elevated volatility, bimodal sentiment)  
> *These results demonstrate that the platform produces context‑sensitive, non‑trivial recommendations.*

---

## ⚠️ Known Limitations & Model Risks

We believe in **transparent risk disclosure**. Current limitations include:

1. **Data dependency** – Yahoo Finance is free but has no SLA; may have delays or gaps.  
2. **GBM tail‑risk misspecification** – Assumes normally distributed log‑returns, underestimates extreme events (empirical kurtosis ~5.2 vs Gaussian 3.0).  
3. **Symmetric GARCH** – Standard GARCH(1,1) ignores the leverage effect (negative shocks increase volatility more than positive ones).  
4. **VADER lexicon** – Not financial‑domain specific; F1‑score ~0.64 (vs FinBERT’s 0.88).  
5. **Static composite weights** – Fixed (35/30/25/10) do not adapt to changing market regimes.

> See the full project report (PDF) for a detailed discussion and a strategic roadmap (walk‑forward backtesting, asymmetric GARCH, FinBERT, adaptive ML weighting).

---

## 🧪 Running Tests & Validation

No automated test suite is included yet. To manually verify:

- Backend: `curl http://localhost:8000/api/stock/RELIANCE.NS`
- Frontend: Open `http://localhost:3000`, search for a stock, and check the dashboard.

---

## 🔮 Future Roadmap

| Timeframe | Enhancements |
|-----------|---------------|
| **Near‑term (0–6 mo)** | Migrate to IEX Cloud/Polygon.io; implement GJR‑GARCH (asymmetric); add walk‑forward backtesting framework. |
| **Medium‑term (6–18 mo)** | Replace GBM with Heston stochastic volatility or Merton jump‑diffusion; integrate FinBERT sentiment; portfolio mean‑variance optimisation (Black‑Litterman). |
| **Long‑term (18+ mo)** | Adaptive ML composite scoring (XGBoost/LightGBM) that learns dynamic weights; alternative data (satellite, credit card, earnings call transcripts). |

---

## 📄 License

This project is open source under the **MIT License**.

## 🙏 Acknowledgements

- Yahoo Finance (`yfinance`) & News API for free data  
- Ollama for local LLM inference  
- shadcn/ui, Next.js, FastAPI communities  
- The academic authors of Fama‑French, Brock, Bollerslev, VADER, and others whose work underpins the quantitative modules  

---

**Built with ❤️ by Debajyoti Majumder, Shaptorishi Bhattacharya, Debrishti Biswas, Ayan Jana, Sandip Naskar**  
*Brainware University – Master of Computer Applications (Computational Sciences)*
```

This README now serves as a complete, accurate, and professional documentation for your project – merging the existing codebase with the rigorous financial methodology and case studies from your project report.
