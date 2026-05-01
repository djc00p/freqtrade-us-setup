# 📈 Freqtrade US Deployment & Compliance Manual

[![ClawHub Skill](https://img.shields.io/badge/ClawHub-Skill-blue)](https://clawhub.ai/djc00p/freqtrade-us-setup) [![Agent Skill](https://img.shields.io/badge/Agent-Skill-blue)](#) [![Freqtrade Crypto](https://img.shields.io/badge/Freqtrade-Crypto-green)](https://github.com/freqtrade/freqtrade)

**Secure, compliant, and automated deployment of Freqtrade for US-based traders using Kraken and Docker.**

> [!CAUTION]
> **REGULATORY COMPLIANCE WARNING:** This manual is intended for users operating within legal jurisdictions. **Never use VPNs or proxies to bypass exchange geo-restrictions.** Doing so violates the Terms of Service (ToS) of most exchanges (including Kraken), creates significant risk of permanent account freezes, and may carry legal implications. Always use a US-compatible exchange.

---

## 🛡️ Architectural Principle: Least Privilege

A fundamental pillar of this deployment is the **Principle of Least Privilege (PoLP)**. When generating API keys, you must ensure the bot has only the permissions necessary to execute trades, and **never** the permission to move funds.

### API Permission Matrix (Kraken)

When creating your API key in the Kraken dashboard, configure permissions **exactly** as follows:

| Permission | Status | Reasoning |
| :--- | :---: | :--- |
| **Funds: Query** | ✅ | Allows the bot to check balances for margin/trading. |
| **Orders: Query** | ✅ | Allows the bot to track open/closed trades. |
| **Orders: Create/Modify** | ✅ | Essential for placing entry and exit orders. |
| **Orders: Cancel/Close** | ✅ | Allows the bot to trigger stop-losses. |
| **Withdraw** | ❌ | **CRITICAL:** Disabling this prevents theft if keys are leaked. |
| **Transfer** | ❌ | Disabling this prevents funds from being moved internally. |

---

## ⚙️ Implementation: Secure Environment Injection

To prevent credential leakage, this deployment **strictly prohibits** hardcoding API keys in `config.json`. Instead, we utilize the **Environment Variable Override** pattern.

Freqtrade uses a double-underscore (`__`) syntax to map shell environment variables to the JSON configuration hierarchy at runtime.

### 1. The `.env` Configuration

Create a `.env` file in your project root. This file contains your secrets and **must be added to your `.gitignore` immediately.**

```bash
# --- FREQTREDE API SECRETS ---
# Maps to freqtrade.exchange.key
FREQTRADE__EXCHANGE__KEY=your-kraken-api-key

# Maps to freqtrade.exchange.secret
FREQTRADE__EXCHANGE__SECRET=your-kraken-secret

# --- API SERVER AUTHENTICATION ---
# Secure your Web UI with a robust password
FREQTRADE__API_SERVER__USERNAME=admin_user
FREQTRADE__API_SERVER__PASSWORD=your_secure_password
FREQTRADE_SERVER__JWT__SECRET__KEY=a_long_random_string_for_jwt

# --- TELEGRAM NOTIFICATIONS (Optional) ---
# Use these to receive trade alerts on your mobile device
FREQTRADE__TELEGRAM__TOKEN=your_bot_token
FREQTRADE__TELEGRAM__CHAT_ID=your_chat_id
```

### 2. The `config.json` Sanitization

Your `user_data/config.json` should contain the structure but **empty strings** for all sensitive fields. This ensures that if the config is ever leaked, the attacker has no credentials.

```json
{
  "exchange": {
    "name": "kraken",
    "key": "",
    "secret": "",
    "ccxt_config": {},
    "ccxt_async_config": {}
  },
  "stake_currency": "USDT",
  "dry_run": true,
  "max_open_trades": 3
}
```

---

## 🚀 Deployment Pipeline

### Step 1: Container Initialization

Initialize the Freqtrade directory structure using the Docker-based configuration wizard.

```bash
docker-compose run --rm freqtrade new-config --config /freqtrade/user_data/config.json
```

### Step 2: Data Ingestion

Fetch the necessary historical candlestick data for your strategy.

```bash
docker-compose run --rm freqtrade download-data \
  --pairs BTC/USDT ETH/USDT \
  --timeframe 5m \
  --timerange 20240101-
```

### Step 3: The "Dry-Run" Validation Phase

**Never deploy to live trading without a minimum of 14 days of successful `dry_run` performance.** This allows you to validate that your logic, the environment variables, and the exchange connectivity are all functioning as expected without financial risk.

```bash
# Start the bot in paper-trading mode
docker-compose up -d
```

---

## ✅ Pre-Live Checklist

Before switching `dry_run: false`, ensure all the following are verified:

- [ ] **Zero-Withdraw Permission:** Verified that the Kraken API key cannot move funds.
- [ ] **Git Integrity:** Verified that `.env` is present in `.gitignore`.
- [ ] **Data Continuity:** Verified that the historical data download is complete and uncorrupted.
  [ ] **Strategy Stability:** Verified that the strategy has survived at least 2 weeks of paper trading.
- [ ] **Alerting:** Verified that Telegram/Discord notifications are correctly alerting you to trade executions.

---

**Disclaimer:** *Freqtrade is open-source software. Trading cryptocurrencies involves significant risk of loss. The author is not responsible for financial losses incurred through the use of this software. Use at your own risk.*

**Original implementation by [@djc00p](https://github.com/djc00p)**
