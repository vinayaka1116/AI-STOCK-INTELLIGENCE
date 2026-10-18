# StockAI Intelligence

StockAI is a React/Vite frontend backed by a FastAPI service. It provides market analysis, model estimates, account access, manually tracked portfolios, and revocable read-only portfolio sharing for Indian and U.S. tickers.

## Local development

### Backend

From `backend`:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
Copy-Item .env.example .env
uvicorn app.main:app --reload
```

The API is available at `http://127.0.0.1:8000`; interactive API docs are at `/docs`. Google sign-in requires the same Google OAuth **Web application** client ID in both `backend/.env` (`GOOGLE_CLIENT_ID`) and `frontend/.env.local` (`VITE_GOOGLE_CLIENT_ID`). Replace the example placeholder in each file. In Google Cloud Console, add the exact frontend origin you use under **Authorized JavaScript origins** (for example, `http://127.0.0.1:5173`; `localhost` and `127.0.0.1` are distinct origins). Restart the backend and Vite dev server after changing either file.

### Backend tests

From `backend`, with the backend environment active:

```powershell
python -m unittest discover -s tests -v
```

The account-isolation tests use temporary SQLite databases and do not modify the development database or place broker orders.

### Frontend

From `frontend`:

```powershell
npm install
Copy-Item .env.example .env.local
npm run dev
```

Set `VITE_API_BASE_URL` to the backend URL. Vite-exposed variables are public client configuration; never put server credentials in a `VITE_*` variable. Copy `backend/.env.example` to `.env` for development. Before deployment, register the production HTTPS frontend origin with Google and set the same OAuth client ID in the frontend and backend environments. Also set `APP_ENV=production`, `SESSION_COOKIE_SECURE=true`, exact HTTPS `APP_CORS_ORIGINS`, and the correct cookie policy for your hosting domains.

## Security behavior

- Registration requires a 12-character password. Successful password or Google authentication issues a random 12-hour session; only its SHA-256 hash is stored in SQLite, and logout revokes it server-side.
- Browser sessions are issued as `HttpOnly`, `SameSite=Lax` cookies; the raw session token is not returned in JSON or stored in JavaScript-accessible storage. State-changing `/api/` requests require an exact configured `Origin`. Use HTTPS and set `SESSION_COOKIE_SECURE=true` before deployment; production mode refuses insecure cookies and non-HTTPS origins. Cookie-based requests require credentialed CORS with exact origins.
- Portfolio read, add, and delete routes require a valid session. The server derives the account from that session; an email supplied by a browser is not used to select portfolio data.
- Portfolio sharing is opt-in. A high-entropy, read-only link exposes the tracked holdings and entry prices, not the account email. Creating another link invalidates the previous one; the owner can revoke all links. Treat the URL as a secret and share it only with people who should see those holdings.
- The former built-in demo credential no longer signs in, and the application no longer creates that demo account on startup.
- Password recovery is deliberately unavailable until an email delivery and verification provider is implemented. The API does not return reset tokens to callers.
- `APP_CORS_ORIGINS` must be set to the exact deployed frontend origins (comma-separated). The development CORS defaults are only for local use.

## Before real-world deployment

This change secures the current session and portfolio boundaries; it does **not** make the application ready to execute trades or handle regulated financial activity. Before opening it to real users:

1. Migrate SQLite user and portfolio data to a managed database with encrypted backups; use a shared session store before running multiple backend instances.
2. Add email-based recovery, login and API rate limits, MFA, account deletion/export, audit logging, monitoring, automated tests, and a documented incident/restore process.
3. Serve frontend and API over HTTPS; configure host-level security headers (including CSP and `Referrer-Policy`), trusted proxy settings, secrets management, and least-privilege database access.
4. Complete a security and privacy review, including applicable financial-services and consumer-protection obligations in each target market.
5. For a public per-user broker connection, obtain the broker's formal platform/partner approval and use its end-customer account/OAuth flow. Alpaca's Broker API is intended for apps serving end customers and requires platform onboarding; an ordinary user's trading API key is not an appropriate shared server credential. Do not ask users to paste broker API keys into StockAI or expose broker secrets in frontend configuration.
6. Select and separately onboard an approved Indian broker for NSE/BSE. Alpaca's U.S. integration does not provide the Indian broker connection. Start with each provider's sandbox, then implement per-customer account authorization, order confirmation, idempotency, reconciliation, and audit trails before any live-order capability.

StockAI does **not** place real orders. Predictions are estimates, not guaranteed returns or personalized financial advice.
# AI-STOCK-INTELLIGENCE
