# NeuroTrade

NeuroTrade is a full-stack stock market analysis and simulated trading platform. It brings market quotes, historical charts, watchlists, portfolio tracking, trade history, price alerts, news sentiment, and experimental price predictions into one interface.

> **Trading note:** Orders are recorded in NeuroTrade as simulated trades. The application does not connect to a brokerage or place live market orders. Machine-learning predictions and sentiment scores are for exploration and are not guaranteed forecasts or investment advice.

## Features

- **Market data:** Stock quotes, company information, historical OHLCV data, market indices, and market movers via `yahoo-finance2`.
- **Charts and trading terminal:** React interface for viewing stock information and simulated trading workflows.
- **Portfolio and trade history:** Track holdings, average purchase price, current estimated value, and profit/loss; record simulated buys and sells.
- **Watchlists:** Add and remove symbols, with quote data used to enrich watchlist entries.
- **Authentication:** Supabase Auth email/password registration and login, access-token-protected API routes, and refresh-session endpoint.
- **Price alerts:** Create price alerts; a scheduled task checks alerts and can send SMS notifications through Twilio when configured.
- **News and sentiment:** Fetch market news from RSS feeds, score news text, and cache results in Supabase.
- **Experimental price predictions:** TensorFlow.js LSTM model trained on historical OHLCV data, with prediction results cached in Supabase. A linear-trend fallback is used when TensorFlow.js is unavailable.
- **Background services:** Scheduled news refreshes, price-alert checks, daily portfolio summaries, and Supabase Realtime initialization.

## Architecture

```text
React + Vite frontend
        │ HTTP requests (/api)
        ▼
Node.js + Express REST API
   ├── Supabase Auth and PostgreSQL (user data, holdings, trades, watchlists)
   ├── Yahoo Finance data service (quotes and historical prices)
   ├── TensorFlow.js prediction service (optional model dependency)
   ├── RSS news and sentiment service
   └── node-cron scheduler and optional Twilio notifications
```

The frontend is in `frontend/`. The backend is in `ml-service/`. The backend validates Supabase access tokens on protected routes and uses Supabase row-level security for user-owned records. Market and news data are fetched by backend services and returned through REST endpoints.

### Technology

| Area | Technology |
| --- | --- |
| Frontend | React, Vite, JavaScript |
| Backend | Node.js, Express |
| Database and authentication | Supabase, PostgreSQL, Supabase Auth, Row Level Security |
| Market data | `yahoo-finance2` |
| Price prediction | TensorFlow.js LSTM (optional dependency) |
| News | RSS Parser and a Transformers.js text-classification pipeline |
| Validation and scheduled jobs | Zod, `node-cron` |
| Notifications | Twilio (optional) |

### Sentiment model detail

The news service currently loads `Xenova/distilbert-base-uncased-finetuned-sst-2-english`, a general-purpose sentiment model. Although some project descriptions refer to FinBERT, FinBERT is **not** the model currently configured in the code. The sentiment output should therefore be understood as an experimental general sentiment signal, not a financial-domain model result.

## Requirements

- Node.js 18 or later and npm
- A Supabase project
- Optional: TensorFlow.js dependencies for LSTM inference/training
- Optional: Twilio credentials for SMS notifications

## Getting started

### 1. Install dependencies

From the project root:

```bash
npm install
npm install --prefix ml-service
npm install --prefix frontend
```

Or run the root helper:

```bash
npm run install:all
```

### 2. Configure environment variables

Copy `ml-service/.env.example` to `ml-service/.env` and set the values for your environment. At minimum, configure:

```dotenv
SUPABASE_URL=your_supabase_project_url
SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_server_only_service_role_key
PORT=8000
FRONTEND_URL=http://localhost:5173
```

Some backend tasks require additional settings, such as an admin key for model training, an SMS provider account for Twilio notifications, and a frontend URL for password reset links. Refer to `ml-service/.env.example` for the complete variable list. Keep secrets in environment files and never commit them. The service-role key must only be used on the server.

### 3. Set up Supabase

In the Supabase SQL Editor, run the migrations in order:

1. `ml-service/supabase/migrations/001_initial_schema.sql`
2. `ml-service/supabase/migrations/002_phone_and_intraday.sql`

The first migration creates profiles, portfolios, trades, watchlists, predictions, alerts, and news-cache tables with row-level security policies. Configure Supabase Storage for model artifacts if you plan to persist trained models.

### 4. Start the frontend and backend

From the project root:

```bash
npm run dev
```

The frontend is served at `http://localhost:5173` and the API defaults to `http://localhost:8000`. The Vite development server proxies `/api` requests to the backend. To run one service independently:

```bash
npm run backend
npm run frontend
```

Check the backend health endpoint at `http://localhost:8000/api/health`.

## API overview

Authenticated endpoints require `Authorization: Bearer <supabase_access_token>`.

| Area | Endpoint examples | Purpose |
| --- | --- | --- |
| Authentication | `POST /api/auth/register`, `POST /api/auth/login`, `POST /api/auth/refresh` | Register, sign in, and refresh sessions |
| Stocks | `GET /api/stocks/quote/:symbol`, `GET /api/stocks/history/:symbol`, `GET /api/stocks/movers` | Quotes, historical data, and market movers |
| Portfolio | `GET /api/portfolio`, `POST /api/portfolio/trade`, `GET /api/portfolio/trades` | Holdings and simulated order history |
| Watchlist | `GET /api/watchlist`, `POST /api/watchlist` | Read and update a user's watchlist |
| Predictions | `GET /api/predictions/:symbol`, `POST /api/predictions/train/:symbol` | Request predictions or start model training |
| News | `GET /api/news`, `GET /api/news/:symbol`, `GET /api/news/sentiment/:symbol` | General and symbol-specific news and sentiment |
| User and alerts | `GET /api/users/me`, `GET /api/users/alerts`, `POST /api/users/alerts` | Profile and price-alert management |

See `ml-service/README.md` and the route files in `ml-service/src/routes/` for more details.

## Machine-learning notes

The LSTM uses a 60-day input sequence of open, high, low, close, and volume values. Training uses five years of daily history when available, min-max normalization, 50 epochs, and a validation split. When TensorFlow.js is not installed, prediction uses a simple linear trend based on recent closing prices.

The current implementation does not provide a completed out-of-sample historical backtest report. Treat predictions and the displayed confidence value as experimental; do not interpret them as measured investment performance.

## Security and data handling

- Protected API routes validate the Supabase access token before handling the request.
- User-owned portfolio, trade, watchlist, profile, and alert records are associated with a Supabase user ID and protected by row-level security policies.
- The Supabase service-role key bypasses row-level security, so it must remain server-side and be used only for trusted administrative tasks.
- Do not put API keys or secrets in frontend code or commit `.env` files.

## License

No license is currently specified. Add a `LICENSE` file if you intend to distribute or reuse this project under particular terms.
