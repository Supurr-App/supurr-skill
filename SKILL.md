---
name: supurr
description: Backtest, deploy, and monitor grid trading bots on Hyperliquid. Supports Native Perps, Spot markets (USDC/USDH), and HIP-3 sub-DEXes.
---

# Supurr CLI — Complete Command Reference

> **For LLMs**: This is the authoritative reference. Use exact syntax. Config files are in `~/.supurr/configs/`.

---

## Quick Reference

| Command                | Purpose                      |
| ---------------------- | ---------------------------- |
| `supurr init`          | Setup wallet credentials     |
| `supurr whoami`        | Show current wallet          |
| `supurr new grid`      | Generate strategy config     |
| `supurr configs`       | List saved configs           |
| `supurr config <name>` | View config details          |
| `supurr backtest`      | Run historical simulation    |
| `supurr deploy`        | Deploy bot to production     |
| `supurr monitor`       | View active bots             |
| `supurr history`       | View historical bot sessions |
| `supurr stop`          | Stop a running bot (signed)  |
| `supurr prices`        | Debug price data             |
| `supurr update`        | Update CLI to latest         |

---

## Global Options

```bash
supurr --help              # Show all commands
supurr --version, -V       # Show CLI version
supurr -d, --debug         # Enable debug logging (any command)
```

---

## 1. `supurr init` — Credential Setup

```bash
# Interactive
supurr init

# Non-interactive
supurr init --address 0x... --api-wallet 0x...

# Overwrite existing
supurr init --force
```

| Option                | Description                    |
| --------------------- | ------------------------------ |
| `-f, --force`         | Overwrite existing credentials |
| `--address <address>` | Wallet address (0x...)         |
| `--api-wallet <key>`  | API wallet private key         |

---

## 2. `supurr whoami` — Show Identity

```bash
supurr whoami    # Shows: Address + masked key
```

---

## 3. `supurr new <strategy>` — Config Generator

### Basic Syntax

```bash
supurr new grid [options]
```

### Market Types

| Type     | Quote    | Requires  | Example                                 |
| -------- | -------- | --------- | --------------------------------------- |
| `native` | USDC     | —         | `--asset BTC`                           |
| `spot`   | Variable | `--quote` | `--asset HYPE --type spot --quote USDC` |
| `hip3`   | Per-DEX  | `--dex`   | `--asset BTC --type hip3 --dex hyna`    |

### All Options

| Option                  | Default       | Description                                    |
| ----------------------- | ------------- | ---------------------------------------------- |
| `-a, --asset <symbol>`  | `BTC`         | Base asset (BTC, ETH, HYPE, etc.)              |
| `-o, --output <file>`   | `config.json` | Output filename                                |
| `--type <type>`         | `native`      | Market type: native, spot, hip3                |
| `--dex <dex>`           | —             | **Required for hip3**: hyna, xyz, km, vntl     |
| `--quote <quote>`       | —             | **Required for spot**: USDC, USDE, USDT0, USDH |
| `--mode <mode>`         | `long`        | Grid mode: long, short, neutral                |
| `--levels <n>`          | `20`          | Number of grid levels                          |
| `--start-price <price>` | —             | Grid start price                               |
| `--end-price <price>`   | —             | Grid end price                                 |
| `--investment <amount>` | `1000`        | Max investment in quote currency               |
| `--leverage <n>`        | `2`           | Leverage (1 for spot)                          |
| `--testnet`             | false         | Use Hyperliquid testnet                        |

### Examples — All 4 Market Types

```bash
# Native Perp (BTC-USDC)
supurr new grid --asset BTC --levels 4 --start-price 88000 --end-price 92000 --investment 100 --leverage 20

# USDC Spot (HYPE/USDC)
supurr new grid --asset HYPE --type spot --quote USDC --levels 3 --start-price 29 --end-price 32 --investment 100

# Non-USDC Spot (HYPE/USDH)
supurr new grid --asset HYPE --type spot --quote USDH --levels 3 --start-price 29 --end-price 32 --investment 100

# HIP-3 (hyna:BTC)
supurr new grid --asset BTC --type hip3 --dex hyna --levels 4 --start-price 88000 --end-price 92000 --investment 100 --leverage 20
```

### HIP-3 DEXes

| DEX    | Quote | Assets                              |
| ------ | ----- | ----------------------------------- |
| `hyna` | USDE  | Crypto perps (BTC, ETH, HYPE, etc.) |
| `xyz`  | USDE  | Stocks (AAPL, TSLA, etc.)           |
| `km`   | USDT  | Kinetiq Markets                     |
| `vntl` | USDE  | AI/tech tokens                      |

---

## 4. `supurr configs` — List Saved Configs

```bash
supurr configs    # Lists all configs in ~/.supurr/configs/
```

**Output:**

```
📁 Configs (/Users/you/.supurr/configs):
  btc-grid.json         grid     BTC-USDC
  hype-usdc-spot.json   grid     HYPE-USDC
  hyna-btc.json         grid     BTC-USDE
```

---

## 5. `supurr config <name>` — View Config

```bash
supurr config btc-grid        # View btc-grid.json
supurr config btc-grid.json   # Same
```

---

## 6. `supurr backtest` — Run Backtest

### Syntax

```bash
supurr backtest -c <config> [options]
```

### Options

| Option                | Description                              |
| --------------------- | ---------------------------------------- |
| `-c, --config <file>` | **Required.** Config file (name or path) |
| `-s, --start <date>`  | Start date (YYYY-MM-DD)                  |
| `-e, --end <date>`    | End date (YYYY-MM-DD)                    |
| `-p, --prices <file>` | Use local prices file                    |
| `-o, --output <file>` | Save results to JSON                     |
| `--no-cache`          | Disable price caching                    |

### Examples

```bash
# By config name (looks in ~/.supurr/configs/)
supurr backtest -c btc-grid.json -s 2026-01-28 -e 2026-02-01

# By full path
supurr backtest -c ~/.supurr/configs/btc-grid.json -s 2026-01-28 -e 2026-02-01

# Save results
supurr backtest -c btc-grid.json -s 2026-01-28 -e 2026-02-01 -o results.json
```

### Archive Data Availability

| Dex           | Asset Format           | Example            |
| ------------- | ---------------------- | ------------------ |
| `hyperliquid` | `BTC`, `HYPE`          | Native perp + Spot |
| `hyna`        | `hyna:BTC`, `hyna:ETH` | HIP-3 DEX          |

> **Note**: Archive data available from 2026-01-28 onwards.

---

## 7. `supurr deploy` — Deploy Bot

```bash
supurr deploy -c <config>
```

| Option                | Description                              |
| --------------------- | ---------------------------------------- |
| `-c, --config <file>` | **Required.** Config file (name or path) |

### Example

```bash
supurr deploy -c btc-grid.json
```

**Output:**

```
✔ Bot deployed successfully!
📦 Deployment Details
  Bot ID:       217
  Pod Name:     bot-217-xyz
  Bot Type:     grid
  Market:       BTC-USDC
```

---

## 8. `supurr monitor` — View Active Bots

### Syntax

```bash
supurr monitor [options]
```

### Options

| Option                   | Description                    |
| ------------------------ | ------------------------------ |
| `-w, --wallet <address>` | Filter by wallet address       |
| `--watch`                | Live mode (refreshes every 2s) |

### Examples

```bash
supurr monitor                 # List all active bots
supurr monitor --watch         # Live monitoring (Ctrl+C to exit)
supurr monitor -w 0x1234...    # Filter by wallet
```

**Output Columns:**

- **ID** — Bot identifier
- **Type** — Strategy (grid, dca, mm)
- **Market** — Trading pair (BTC-USDC, HYPE-USDH)
- **Position** — Size + direction (L=Long, S=Short)
- **PnL** — Total profit/loss

---

## 9. `supurr history` — View Bot History

```bash
supurr history             # Show last 20 bot sessions
supurr history -n 50       # Show last 50 bot sessions
```

| Option                | Default | Description            |
| --------------------- | ------- | ---------------------- |
| `-n, --limit <count>` | `20`    | Number of bots to show |

**Output Columns:**

- **ID** — Bot identifier
- **Market** — Trading pair (from `config.markets[0]`)
- **Type** — Strategy (grid, dca, mm)
- **PnL** — Total profit/loss (realized + unrealized)
- **Stop Reason** — Why the bot stopped (`shutdown:graceful` → "User stopped the bot Successfully")

---

## 10. `supurr stop` — Stop Bot (Signature Auth)

Signs `Stop <bot-id>` with your API wallet private key (EIP-191 personal_sign) and sends the signature to the bot API.

```bash
supurr stop              # Interactive - select from list
supurr stop --id 217     # Stop specific bot by ID
```

| Option          | Description                            |
| --------------- | -------------------------------------- |
| `--id <bot_id>` | Bot ID to stop (from `supurr monitor`) |

> **Crypto:** Uses `@noble/curves/secp256k1` + `@noble/hashes/sha3` (pure JS, no native deps). Signature format: `0x{r}{s}{v}` (65 bytes).

---

## 11. `supurr prices` — Debug Price Data

```bash
supurr prices -a BTC                     # Fetch BTC prices (7 days)
supurr prices -a hyna:BTC --dex hyna     # HIP-3 prices
supurr prices -a HYPE -s 2026-01-28      # From specific date
```

| Option                 | Description                     |
| ---------------------- | ------------------------------- |
| `-a, --asset <symbol>` | **Required.** Asset symbol      |
| `--dex <dex>`          | DEX name (default: hyperliquid) |
| `-s, --start <date>`   | Start date                      |
| `-e, --end <date>`     | End date                        |
| `--no-cache`           | Disable caching                 |

---

## 12. `supurr update` — Self-Update

```bash
supurr update    # Check and install latest version
```

---

## Complete Workflows

### Workflow 1: Backtest → Deploy → Monitor

```bash
# 1. Initialize (first time only)
supurr init --address 0x... --api-wallet 0x...

# 2. Create config
supurr new grid --asset BTC --levels 4 --start-price 88000 --end-price 92000 --investment 100 --leverage 20 --output btc-grid.json

# 3. Backtest
supurr backtest -c btc-grid.json -s 2026-01-28 -e 2026-02-01

# 4. Deploy
supurr deploy -c btc-grid.json

# 5. Monitor
supurr monitor --watch

# 6. Stop when done
supurr stop --id <bot_id>
```

### Workflow 2: Test All Market Types

```bash
# Native Perp
supurr new grid --asset BTC --output native-btc.json
supurr backtest -c native-btc.json -s 2026-01-28 -e 2026-02-01

# USDC Spot
supurr new grid --asset HYPE --type spot --quote USDC --output hype-usdc.json
supurr backtest -c hype-usdc.json -s 2026-01-30 -e 2026-01-31

# Non-USDC Spot
supurr new grid --asset HYPE --type spot --quote USDH --output hype-usdh.json
supurr backtest -c hype-usdh.json -s 2026-01-30 -e 2026-01-31

# HIP-3
supurr new grid --asset BTC --type hip3 --dex hyna --output hyna-btc.json
supurr backtest -c hyna-btc.json -s 2026-01-28 -e 2026-02-01
```

---

## Config Storage

```
~/.supurr/
├── credentials.json      # { address, private_key }
├── configs/              # Saved bot configs
│   ├── btc-grid.json
│   ├── hype-usdc.json
│   └── ...
└── cache/                # Price data cache
    └── hyperliquid/
        ├── BTC/
        └── HYPE/
```

---

## API Endpoints Used

| Purpose       | Endpoint                                      | Auth              |
| ------------- | --------------------------------------------- | ----------------- |
| Bot Deploy    | `POST /bots/create/<wallet>`                  | —                 |
| Active Bots   | `GET /dashboard/active_bots`                  | —                 |
| Bot History   | `GET /dashboard/user_bots/<address>` (Python) | —                 |
| Stop Bot      | `POST /bots/<bot_id>/stop` (Node)             | EIP-191 signature |
| Price Data    | `GET /prices?dex=X&asset=Y&start_time=Z`      | —                 |
| Price Archive | `GET /{dex}/{asset}/{date}.json`              | —                 |

---

## Troubleshooting

| Issue                  | Solution                                       |
| ---------------------- | ---------------------------------------------- |
| "Config not found"     | Use `supurr configs` to list available configs |
| "No credentials"       | Run `supurr init` first                        |
| "0 prices fetched"     | Check date range (data from 2026-01-28+)       |
| "API wallet not valid" | Register API wallet on Hyperliquid first       |
| HIP-3 backtest fails   | Use format `--dex hyna --asset BTC`            |

---

# Appendix: Hyperliquid Info API

> **Get address via**: `supurr whoami` — returns the configured wallet address.

All endpoints use `POST https://api.hyperliquid.xyz/info` with `Content-Type: application/json`.

## Market Data (No Address Required)

| Query            | Request Body                         |
| ---------------- | ------------------------------------ |
| All Mid Prices   | `{"type": "allMids"}`                |
| Sub-DEX Prices   | `{"type": "allMids", "dex": "hyna"}` |
| Perp Metadata    | `{"type": "metaAndAssetCtxs"}`       |
| Spot Metadata    | `{"type": "spotMeta"}`               |
| L2 Order Book    | `{"type": "l2Book", "coin": "BTC"}`  |
| List HIP-3 DEXes | `{"type": "perpDexs"}`               |

## User Data (Address Required)

| Query           | Request Body                                                                   |
| --------------- | ------------------------------------------------------------------------------ |
| Perp Positions  | `{"type": "clearinghouseState", "user": "0x..."}`                              |
| Spot Balances   | `{"type": "spotClearinghouseState", "user": "0x..."}`                          |
| Open Orders     | `{"type": "openOrders", "user": "0x..."}`                                      |
| Order History   | `{"type": "historicalOrders", "user": "0x..."}`                                |
| Trade Fills     | `{"type": "userFills", "user": "0x...", "aggregateByTime": true}`              |
| Funding History | `{"type": "userFunding", "user": "0x...", "startTime": <ts>, "endTime": <ts>}` |
| Sub-Accounts    | `{"type": "subAccounts", "user": "0x..."}`                                     |

## HIP-3 Sub-DEXes

| DEX    | Quote | Assets                        |
| ------ | ----- | ----------------------------- |
| `hyna` | USDE  | Crypto perps (BTC, ETH, HYPE) |
| `xyz`  | USDE  | Stocks (AAPL, TSLA, NVDA)     |
| `vntl` | USDE  | AI/tech tokens                |
| `km`   | USDT  | Kinetiq Markets               |
| `cash` | USDC  | Tech stocks                   |

## TypeScript Helper

```typescript
const HL = "https://api.hyperliquid.xyz/info";

async function query<T>(body: object): Promise<T> {
  const res = await fetch(HL, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(body),
  });
  return res.json();
}

// Examples
const mids = await query({ type: "allMids" });
const positions = await query({ type: "clearinghouseState", user: "0x..." });
const spotBal = await query({ type: "spotClearinghouseState", user: "0x..." });
const dexes = await query({ type: "perpDexs" });
```

## Common Hazards

| Issue                   | Solution                                          |
| ----------------------- | ------------------------------------------------- |
| `szDecimals` truncation | Truncate qty to `szDecimals` before submit        |
| HIP-3 price prefix      | Sub-DEX prices keyed as `hyna:BTC`, not `BTC`     |
| Sub-DEX asset index     | Use local index from DEX's `universe`, not global |
| Fill limits             | `userFills` max 2000 — paginate with time ranges  |
