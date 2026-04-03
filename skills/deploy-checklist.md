# Deploy Checklist — StrideBot

Run every check before presenting any file for deployment. Production crashes during live usage are unacceptable.

## Pre-Deploy (MANDATORY)

### 1. Syntax check every modified file

```python
import ast
with open('filename.py') as f:
    ast.parse(f.read())
```

If it throws SyntaxError — do not deploy. Fix first.

### 2. Concurrency check

- Every async handler that makes multiple API calls must use asyncio.gather()
- Every blocking function with multiple API calls must use ThreadPoolExecutor
- Never write sequential awaits — parallelise them
- Check: grep for await inside loops — almost always wrong

### 3. File deployment list

State explicitly which files deploy together. If a function in handlers.py calls a function in exchange.py that was changed, both must deploy.

### 4. Bot startup trace

Read bot.py top to bottom. Verify:
- logger is defined before any function that uses it
- ADMIN_USER_ID is read from env and bot raises ValueError if missing
- clear_webhook() is called before ApplicationBuilder
- t.sleep(20) exists before run_polling()
- Health check server starts before bot polling

### 5. Callback patterns

Every InlineKeyboardButton callback_data must have a matching CallbackQueryHandler in bot.py. Check pattern strings match exactly.

## Changes That Require Bot Restart

- Any change to bot.py handler registration
- Any new command added
- Any new callback pattern added
- DATABASE schema changes (init_db() runs on startup)

## Database Schema Changes

1. Add CREATE TABLE IF NOT EXISTS or ADD COLUMN IF NOT EXISTS in init_db()
2. Deploy database.py WITH bot.py together
3. Verify in Render logs: "Database initialized successfully."

## Environment Variables

Required env vars in Render dashboard:

- BOT_TOKEN, ADMIN_USER_ID, DATABASE_URL
- GROQ_API_KEY, TAVILY_API_KEY, CHANNEL_ID
- NEWSAPI_KEY, GNEWS_API_KEY, GUARDIAN_API_KEY
- ALPHA_VANTAGE_API_KEY, FINNHUB_API_KEY
- EXCHANGERATE_API_KEY, FRED_API_KEY
- LUNARCRUSH_API_KEY, CMC_API_KEY, MARKETAUX_API_KEY
- X_API_KEY, X_API_SECRET, X_ACCESS_TOKEN, X_ACCESS_TOKEN_SECRET
- CARDTONIC_API_KEY (when obtained)
- BREET_API_KEY (when obtained)
- PAYSTACK_SECRET_KEY (when obtained)
- DEPOSIT_USDT_ADDRESS, DEPOSIT_BTC_ADDRESS, DEPOSIT_TON_ADDRESS, DEPOSIT_SOL_ADDRESS

## Render-Specific Notes

- Build command: requirements install from requirements.txt
- Start command: python bot/bot.py
- Python version: 3.11.9 (set in .python-version at repo root)
- UptimeRobot pings https://stridebot.onrender.com every 5 minutes
- Free tier spins down without UptimeRobot — keep it active

## One Feature Per Deployment Rule

Never combine multiple unrelated features in one deploy. Deploy Feature A, verify it works, then deploy Feature B.
