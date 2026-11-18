# TruthLens AI — Universal Fake News Detection & Fact-Checking Platform

**TruthLens AI** is a full-stack, evidence-based AI fact-checking platform that verifies news headlines, full articles, social media posts, website URLs, screenshots/images, and PDF documents across **18 automatically detected news categories**.

Unlike superficial headline classifiers, TruthLens AI breaks submissions into individual verifiable claims, investigates each claim using live web search grounding, weighs supporting versus contradicting evidence across authoritative sources, and generates a transparent investigative dossier with a **7-tier verdict** and **0–100 Evidence Credibility Score**.

---

## Key Capabilities

- **4 Multi-Modal Input Formats:** Paste Text, Enter URL (SSRF-protected article scraper with graceful paywall/403 disclosure), Upload Image/Screenshot (multimodal OCR + optional Tesseract OCR + EXIF metadata inspection + SHA-256 hash + transparent reverse-image search disclosure), and Upload PDF (multi-page text & metadata extraction + scanned-PDF multimodal fallback).
- **18 Automatically Detected News Categories:** Politics & Elections, Geopolitics & International Relations, Wars/Conflicts/Diplomacy, Sports & Tournaments, Business & Finance, Economics & Government Policies, Science & Research, Technology & AI, Health & Medicine, Entertainment & Celebrities, Education, Crime & Law, Environment & Climate, History, Social Issues, Government Announcements, International News, and Other/Miscellaneous—each paired with a tailored category verification protocol.
- **7-Tier Evidence Verdict System:** `LIKELY TRUE`, `LIKELY FALSE`, `MISLEADING`, `UNVERIFIED`, `OUTDATED`, `PARTIALLY TRUE`, `SATIRE / PARODY`.
- **14 Mandatory AI Accuracy Safeguards:** Prompt-injection isolation (`<untrusted_submission>`), strict anti-hallucination URL validation against real search grounding citations, clear distinction between Credibility Score and statistical probability of truth, separation of publisher reputation from claim accuracy, and zero fabricated sample results when an API key is missing.
- **Full Persistence & Exports:** Relational SQLite (or PostgreSQL via `DATABASE_URL`) storage for users, analyses, claim-level records, and source citations; searchable & filterable History page; Markdown (`.md`), JSON (`.json`), and Print-to-PDF dossier exports; optional user accounts with JWT authentication + seamless guest session support.

---

## Architecture

```text
├── backend/                     # Python FastAPI REST API
│   ├── app/
│   │   ├── main.py              # FastAPI app, CORS, lifespan DB init
│   │   ├── config.py            # Environment-driven configuration (Pydantic Settings)
│   │   ├── constants.py         # 18 categories, 7 verdicts, category protocols, 14 accuracy rules
│   │   ├── database.py          # SQLAlchemy engine & session management
│   │   ├── models.py            # User, Analysis, ClaimRecord, SourceReference ORM tables
│   │   ├── schemas.py           # API & structured LLM output Pydantic schemas
│   │   ├── auth.py              # PBKDF2-SHA256 password hashing & JWT tokens
│   │   ├── services/
│   │   │   ├── ai_engine.py     # 2-stage Gemini Interactions API + Google Search grounding (+ OpenAI fallback)
│   │   │   ├── url_extractor.py # SSRF-guarded HTML/JSON-LD article extractor
│   │   │   ├── pdf_extractor.py # pypdf text & metadata extractor
│   │   │   ├── image_inspector.py # Pillow EXIF metadata, SHA-256 & optional Tesseract OCR
│   │   │   ├── web_search.py    # Optional Tavily & Serper live search enrichment
│   │   │   └── report_exporter.py # Markdown & JSON dossier generator
│   │   └── routers/
│   │       ├── health.py        # GET /api/health, GET /api/status
│   │       ├── auth.py          # POST /api/auth/register, /login, GET /me
│   │       ├── analyze.py       # POST /api/analyze/{text,url,image,pdf}
│   │       └── history.py       # GET/DELETE /api/history, GET /export
│   ├── tests/                   # Pytest unit & integration tests
│   ├── requirements.txt
│   └── .env.example
└── frontend/                    # React + TypeScript + Tailwind CSS v4 + Vite
    ├── src/
    │   ├── pages/               # HomePage, AnalysisDashboardPage, HistoryPage, AboutPage
    │   ├── components/          # SubmissionStudio, CredibilityGauge, ClaimCard, SourceCard, AnalyticsCharts, MediaForensicsPanel, etc.
    │   ├── context/             # AuthContext & system readiness state
    │   ├── lib/                 # Typed REST API client & verdict metadata
    │   └── types/               # TypeScript models
    └── package.json
```

---

## Prerequisites (100% Free-Tier Services)

- **Python 3.11+**
- **Node.js 20+** and **npm 10+**
- Two free-tier API keys (stored only in `backend/.env`, never exposed to frontend or committed to Git):
  - **Google Gemini API (`GEMINI_API_KEY`):** Free key from [Google AI Studio](https://aistudio.google.com/apikey) — uses free-tier Gemini Flash (`gemini-3.8-flash` with automatic free fallback to `gemini-3.5-flash-lite` / `gemini-3.1-flash-lite`) via `google-genai`. No OpenAI or paid billing required.
  - **Tavily Search API (`TAVILY_API_KEY`):** Free key (1,000 searches/month) from [Tavily](https://tavily.com) — uses free `basic` search depth with built-in TTL caching and supporting/contradicting query balancing.

---

## Local Setup & Running

### 1. Configure Environment Variables

Copy the example backend environment file and add your free API keys:

```bash
cp backend/.env.example backend/.env
```

Edit `backend/.env`:

```env
GEMINI_API_KEY=your-free-gemini-api-key
TAVILY_API_KEY=your-free-tavily-api-key
AI_PROVIDER=gemini
SEARCH_PROVIDER=tavily
GEMINI_MODEL=gemini-3.8-flash
CACHE_TTL_SECONDS=3600
MAX_TAVILY_QUERIES_PER_ANALYSIS=3
```

> **Security & Cost Note:** `backend/.env` is excluded via `.gitignore`. TruthLens AI uses only free-tier endpoints, caches recent searches and reports to conserve free quota, displays a clear message if free limits are exhausted, and never generates fake results on error.

### 2. Start the Backend Server (FastAPI)

On Windows PowerShell:

```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

On macOS / Linux:

```bash
cd backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

API docs are automatically available at `http://127.0.0.1:8000/docs`.

### 3. Start the Frontend Development Server (React + Vite)

In a second terminal:

```bash
cd frontend
npm install
npm run dev
```

Open `http://localhost:5173` in your browser. Vite automatically proxies `/api/*` requests to `http://127.0.0.1:8000`.

---

## Running Tests

### Backend Tests (Pytest)

```powershell
backend\.venv\Scripts\python.exe -m pytest -c backend\pytest.ini backend\tests -v
```

### Frontend Typecheck & Production Build

```bash
cd frontend
npm run build
```

---

## Deployment Guide

### Backend (Render, Railway, Fly.io, or Docker)
1. Provision a PostgreSQL database and set `DATABASE_URL=postgresql+psycopg://...`.
2. Set `GEMINI_API_KEY`, `JWT_SECRET` (strong random secret), and `CORS_ORIGINS=https://your-frontend-domain.com`.
3. Start with production ASGI server:
   ```bash
   uvicorn app.main:app --host 0.0.0.0 --port $PORT --workers 2
   ```

### Frontend (Vercel, Netlify, or Cloudflare Pages)
1. Set build command to `npm run build` and output directory to `dist` inside `frontend/`.
2. Set `VITE_API_BASE_URL=https://your-backend-api-domain.com` as a build-time environment variable.
3. Configure SPA fallback rewrite (`/* -> /index.html`) for client-side React Router routes (`/analysis/:id`, `/history`, `/about`).
