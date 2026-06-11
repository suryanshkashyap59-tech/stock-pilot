# StockPilot — AI Stock Research Dashboard

Professional Groww/Zerodha-style stock research platform with real-time Yahoo Finance data, TradingView Lightweight Charts, AI technical analysis, and Indian + US market support.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — API server (port 8080, path `/api`)
- `pnpm --filter @workspace/stockpilot run dev` — Vite frontend (path `/`)
- `pnpm run typecheck` — full typecheck across packages

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5 (api-server)
- Frontend: Vanilla JS + Vite (stockpilot)
- Charts: TradingView Lightweight Charts v4.2.0 (CDN)
- Data: Yahoo Finance v8/v11 endpoints (no API key required)
- AI: Algorithmic (RSI, MACD, SMA, BB, volatility) — if `OPENAI_API_KEY` set, uses GPT-4o Mini

## Where things live

- `artifacts/api-server/src/routes/` — all API routes (stocks, history, profile, news, search, market, ai)
- `artifacts/api-server/src/lib/` — cache.ts (TTL cache), yahoo.ts (YF helpers + symbol map)
- `artifacts/stockpilot/index.html` — full app layout
- `artifacts/stockpilot/public/style.css` — Groww-style dark theme
- `artifacts/stockpilot/public/script.js` — all frontend logic

## API Routes

| Route | Description |
|---|---|
| `GET /api/stocks/search?q=` | Search autocomplete (Yahoo Finance v1) |
| `GET /api/stocks/:ticker` | Real-time quote (price, OHLCV, market cap) |
| `GET /api/stocks/:ticker/history?tf=` | OHLCV candles (1D/1W/1M/3M/6M/1Y/5Y/MAX) |
| `GET /api/stocks/:ticker/profile` | Company profile + financials (quoteSummary) |
| `GET /api/stocks/:ticker/news` | Latest news headlines |
| `GET /api/market/overview` | Indices + popular US/IN stocks + gainers/losers |
| `POST /api/ai/analyze` | Technical analysis (algorithmic or GPT) |

## Indian Stock Support

Pass `RELIANCE`, `TCS`, `INFY`, `HDFCBANK`, `SBIN`, `WIPRO` etc. without `.NS` suffix — the backend maps these automatically via `SYMBOL_MAP` in `lib/yahoo.ts`.

## Architecture decisions

- All Yahoo Finance calls proxy through the Express backend (avoids CORS, enables caching)
- TTL cache in `lib/cache.ts` — 15s for quotes, 5min for news/history, 1hr for profiles
- TradingView charts use Unix timestamp (seconds) throughout, consistent with Yahoo Finance responses
- AI analysis is algorithmic by default; set `OPENAI_API_KEY` env var to upgrade to GPT-4o Mini (uses raw `fetch`, no extra npm package needed)
- Indian stocks mapped by removing `.NS` from user input and re-adding for Yahoo Finance

## Product

Landing page shows live market indices, US + Indian stock grids, top gainers/losers, scrolling ticker tape. Stock dashboard shows: large TradingView chart (area/candlestick/line, 8 timeframes, SMA20/SMA50/EMA20/BB/RSI/MACD overlays), company profile + financials, AI technical analysis card (signal, confidence, factors), and live news feed. Watchlist persisted in localStorage.

## User preferences

_Populate as you build._

## Gotchas

- Yahoo Finance quoteSummary (profile) may return 404 for some tickers — handled gracefully with empty response
- Market overview fetches 14 stocks in parallel — may be slow on cold start (~3-5s)
- `pnpm run build` requires `PORT` env var (set by workflow); use `typecheck` for CI validation instead
