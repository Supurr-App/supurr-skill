# Supurr Skills

> Agent skills for Hyperliquid trading bot automation

## Installation

```bash
curl -fsSL https://cli.supurr.app/install | bash
```

Or via [skills.sh](https://skills.sh):

```bash
npx skills add Supurr-App/supurr_skill
```

Supports Claude Code, Cursor, and OpenCode.

## Available Skills

| Skill                | Description                                                       |
| -------------------- | ----------------------------------------------------------------- |
| [supurr](./SKILL.md) | Complete CLI for backtesting, deploying, and monitoring grid bots |

## Quick Start

```bash
# 1. Install CLI
curl -fsSL https://cli.supurr.app/install | bash

# 2. Setup credentials
supurr init --address 0x... --api-wallet 0x...

# 3. Create config
supurr new grid --asset BTC --levels 4 --start-price 88000 --end-price 92000

# 4. Backtest
supurr backtest -c config.json -s 2026-01-28 -e 2026-02-01

# 5. Deploy
supurr deploy -c config.json

# 6. Monitor
supurr monitor --watch
```

## Repository Structure

```
supurr_skill/
├── .claude-plugin/
│   └── plugin.json       # Claude plugin manifest
├── scripts/
│   └── install.sh        # Universal installer
├── SKILL.md              # Main skill documentation
└── README.md             # This file
```

## License

MIT
