# Render Runbook — StrideBot

Log error -> known fix. Check here before debugging from scratch.

## Conflict: terminated by other getUpdates request

**Cause:** Two bot instances running simultaneously during redeploy
**Fix:** Wait 60 seconds. If persists, manually restart in Render dashboard.
**Prevention:** t.sleep(20) in bot.py before run_polling()

## 502 Bad Gateway (UptimeRobot)

**Cause A:** Service not yet deployed - **Fix A:** Wait 2-3 minutes after deploy
**Cause B:** Health check returning 501 Not Implemented - **Fix B:** Ensure do_HEAD and do_POST are in HealthHandler (bot.py)
**Cause C:** X-Render-Routing: no-deploy in response - **Fix C:** Check Render dashboard for failed build

## CoinGecko 429 flooding logs

**Cause:** Price cache not working or TTL too short
**Fix:** Check _price_cache in crypto.py. TTL is 60 seconds. Ensure get_price() reads cache before hitting API.
**Verify:** Look for `returning stale cache` in logs.

## Binance price empty (every request)

**Cause:** Expected - Render IP is geo-blocked by Binance
**Action:** None required. Log as WARNING only. CoinGecko and Kraken handle prices fine.

## psycopg2.OperationalError: SSL connection closed

**Cause:** PostgreSQL idle timeout on Render free tier
**Fix:** database.py has retry logic - one automatic retry. Check get_connection() in database.py.

## pytrends rising queries error: method_whitelist

**Fix:** urllib3==1.26.18 in requirements.txt. Redeploy.

## No crypto-relevant trends detected

**Cause A:** Google blocking Render datacenter IP - **Action A:** Wait - usually resolves in a few hours
**Cause B:** No qualifying terms in rising queries - **Action B:** Normal - check again in 12 hours

## Bot commands not showing for users

**Cause:** Bot command list not updated after adding new commands
**Fix:** Deploy bot.py - register_commands() runs on startup and updates Telegram

## X posting 401 Unauthorized

**Cause:** X API credentials not set in Render environment variables
**Fix:** Add X_API_KEY, X_API_SECRET, X_ACCESS_TOKEN, X_ACCESS_TOKEN_SECRET in Render

## X posting 402 Payment Required

**Cause:** X free tier is read-only - posting requires Basic plan ($100/month)
**Action:** Manual copy-paste fallback active. No fix without paid X plan.

## Breaking news posting same story repeatedly

**Cause A:** posted_content table not being checked - **Fix A:** Ensure database.py has posted_content table and is_content_posted()
**Cause B:** Bot restarted - old in-memory cooldowns lost - **Fix B:** DB-backed cooldowns in scheduler.py. Check db.is_on_cooldown() is called.

## Exchange transactions not reaching admin

**Cause:** ADMIN_USER_ID not set in Render env vars
**Fix:** Set ADMIN_USER_ID in Render -> StrideBot -> Environment

## Render deployment keeps failing (Pillow build error)

**Cause:** Pillow pinned to version incompatible with Python 3.11+
**Fix:** Pillow>=10.4.0 in requirements.txt (not ==10.2.0)
