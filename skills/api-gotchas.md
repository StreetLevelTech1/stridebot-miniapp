# API Gotchas — StrideBot

Real failures we hit in production. Check here before debugging.

---

## Binance

**Problem:** Returns `{"code":0,"msg":"Service unavailable from restricted location"}`

**Cause:** Render/Railway IPs are geo-blocked by Binance

**Fix:** This is expected. CoinGecko or Kraken serves as fallback. Log as `WARNING` only, never `ERROR`.

**Code:** `if price: return price` — chain falls through automatically

---

## CoinGecko

**Problem:** `{"status":{"error_code":429}}` — rate limit hit constantly

**Cause:** Free tier is 30 calls/minute. Scheduler + user commands exhaust this fast.

**Fix:** 60-second TTL cache in `crypto.py`. Returns stale price on 429 rather than `None`.

**Key:** `_price_cache` dict — `{ coin_id: (price, timestamp) }`. Check `PRICE_CACHE_TTL = 60`.

**DO NOT** add more CoinGecko calls without checking cache is in place.

---

## Polymarket (Gamma API)

**Problem:** Market data returns `"outcomePrices":"[0.68, 0.32]"` — JSON string not list

**Cause:** Polymarket wraps array in a string

**Fix:** `json.loads(market["outcomePrices"])` — always parse, never assume list

**Also:** CLOB API endpoint is `/markets` not `/clob/markets`

---

## pytrends

**Problem:** `Retry.__init__() got an unexpected keyword argument 'method_whitelist'`

**Cause:** urllib3 2.0 renamed `method_whitelist` → `allowed_methods`

**Fix:** Pin `urllib3==1.26.18` in `requirements.txt`

---

**Problem:** `trending_searches()` returns casino spam and SEO garbage

**Fix:** Use `related_queries()` only. Seed terms: bitcoin, crypto, ethereum, solana. Never use `trending_searches()` in production.

---

**Problem:** Google blocks Railway/Render datacenter IPs

**Symptom:** Empty results, no error thrown

**Fix:** Wait — usually temporary. If persistent, switch to news-based trend detection.

---

## Render

**Problem:** Health check returns 501 Not Implemented

**Cause:** UptimeRobot sends HEAD requests, HealthHandler only had `do_GET`

**Fix:** Add `do_HEAD` and `do_POST` to `HealthHandler` in `bot.py`

---

**Problem:** PORT env var — Address already in use

**Cause:** Render injects `PORT` env var (usually 10000), not 8080

**Fix:** `port = int(os.getenv("PORT", 8080))` — always read from env

---

**Problem:** Two instances conflict on redeploy

**Symptom:** `Conflict: terminated by other getUpdates request`

**Fix:** `t.sleep(20)` in `main()` before `run_polling()` — gives old instance time to die

---

## Groq

**Problem:** Context degradation after long sessions

**Cause:** Token budget exceeded — Groq silently truncates input

**Fix:** Keep input ≤7000 tokens. Scheduler calls use `max_tokens=800` separately. Static content goes first in system prompt. Never put timestamps in system prompt.

---

## PostgreSQL (Render/Railway)

**Problem:** `psycopg2.OperationalError: SSL connection has been closed unexpectedly`

**Cause:** Render PostgreSQL idles and drops connections after ~5 minutes

**Fix:** Retry logic in `get_connection()` — one automatic retry on `OperationalError`. Check `database.py` `get_connection()` function.

---

## Cardtonic

**Status:** API key on waitlist as of April 2026. Beta test form submitted.

**Endpoint (when available):** `https://api.cardtonic.com/v1/trade/validate`

**Auth:** Bearer token in `Authorization` header

**Manual fallback:** Admin receives card details via DM, redeems on cardtonic.com manually

**IMPORTANT:** Cardtonic rate is fixed regardless of whether receipt is provided. Do not show receipt bonus to users — it does not exist on Cardtonic.

---

## Breet

**Status:** API access pending — contact dev@breet.io

**Limit:** ~₦500k cumulative before KYC triggered

**Manual fallback:** Admin receives payout request, processes manually via Breet app

**IMPORTANT:** Warn users at ₦400k cumulative to avoid hitting KYC gate unexpectedly.

---

## Alpha Vantage

**Problem:** `{"Note":"Thank you for using Alpha Vantage..."}` — rate limit hit

**Cause:** 25 request/day free tier limit hit

**Fix:** Cache stock data aggressively. Replace with Finnhub when integrated. Finnhub free tier: 60 req/minute. Register at finnhub.io.

---

## ExchangeRate-API

**Primary:** `api.exchangerate-api.com/v4/latest/USD` — requires key

**Fallback:** `open.er-api.com/v6/latest/USD` — completely free, no key

Always use fallback chain. Never assume primary works.
