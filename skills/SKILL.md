# StrideBot SKILL.md — Master Index

This is the index. Never dump content here. Each line is a pointer. Read the linked file when needed.

---

## API Gotchas
→ `skills/api-gotchas.md`
Binance geo-block, CoinGecko 429, Polymarket JSON parsing, pytrends urllib3 conflict, Render PORT var, Cardtonic endpoints

## Deploy Checklist
→ `skills/deploy-checklist.md`
ast.parse, concurrency check, file list, startup delay rules

## Render Runbook
→ `skills/render-runbook.md`
Log error → known fix mapping, common crash patterns

---

## Architecture Decisions (do not revisit)

- HTML `parse_mode` throughout (Markdown breaks on `$` and `_`)
- `asyncio.gather()` for all parallel API calls
- `ThreadPoolExecutor` for blocking functions with multiple API calls
- Token budget: 7k input, 800 for scheduler
- Cooldowns stored in DB — survive restarts
- Content hashes (MD5) in `posted_content` — prevent duplicate posts
- Fee: 7% on all exchange transactions
- Breaking news detector = KAIROS pattern (tick-based, append-only log)

---

## File Map

| File | Purpose |
|---|---|
| `bot.py` | ApplicationBuilder, command registration, scheduler jobs |
| `handlers.py` | All command and message handlers, exchange flow |
| `ai.py` | Groq calls, system prompts, tier config |
| `crypto.py` | Price fetching (Binance→CoinGecko→Kraken), 60s cache |
| `scheduler.py` | 10 scheduled posts, breaking news detector, trend check |
| `database.py` | PostgreSQL schema, all DB functions, cooldowns |
| `exchange.py` | Gift card redemption, crypto conversion, Cardtonic/Breet |
| `news_sources.py` | 20+ source pipeline (RSS + APIs) |
| `polymarket.py` | Polymarket intelligence, EV calculator, risk engine |
| `trends.py` | pytrends rising queries, spam filter, quality allowlist |
| `xposter.py` | Tweepy OAuth, post_tweet(), post_thread() |
| `miniapp/` | Telegram Mini App (index.html) |
