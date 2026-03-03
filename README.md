# Polymarket Sports Arbitrage Trading Bot

TypeScript-native directional arbitrage pipeline and paper auto-trading engine for Polymarket sports markets.

This edition is intentionally built for Node.js operations, npm automation, and JS/TS ecosystem integration.

---

## Demo Video

https://github.com/user-attachments/assets/2077bde6-023d-4533-97af-c0401aed9cc0


## Overview

This project runs a full research-to-execution flow:

1. fetch sports markets from Polymarket Gamma
2. filter market rows to arbitrage-relevant candidates
3. fetch and merge sportsbook odds data
4. detect directional value opportunities
5. run paper auto-trading with risk gates
6. write persistent journal files for analysis

It is directional trading, not guaranteed risk-free arbitrage.

---

## Why This TS Build

- npm-first runtime and tooling
- TypeScript contract safety across modules
- easy script orchestration with `tsx`
- better fit for teams already running JS services
- straightforward integration into dashboards and bots

---

## Features

- Gamma market fetch + flatten
- match-winner and draw-focused filtering
- sport auto-detection from ticker/teams/text
- event matching with confidence score
- odds normalization and bookmaker consolidation
- directional opportunity detection and sell points
- paper-mode auto-trader with risk denials
- live-mode credential guardrails
- JSON, CSV, JSONL output surfaces

---

## Architecture

```text
src/
  cli/        # runnable commands
  data/       # fetch and merge adapters
  processing/ # matching and strategy math
  trading/    # config, engine, journals
  utils/      # logger and file helpers
```

Main loop:

`Gamma -> Filter -> Odds Merge -> Detection -> Risk -> Paper Fill -> Journals`

---

## Requirements

- Node.js 20+
- npm 9+
- valid `ODDS_API_KEY`

---

## Installation

```bash
npm install
cp .env.example .env
```

Optional build check:

```bash
npm run build
```

---

## Environment

Minimum required:

```env
ODDS_API_KEY=your_odds_api_key
TRADING_MODE=paper
ENABLE_LIVE_TRADING=false
TRADING_DRY_RUN=true
```

### Pipeline variables

| Variable | Default | Purpose |
|---|---|---|
| `GAMMA_API_URL` | `https://gamma-api.polymarket.com` | Market source |
| `OUTPUT_DIR` | `data` | Output directory |
| `ODDS_API_REGIONS` | `us,us_ex` | Odds regions |
| `ODDS_API_MARKETS` | `h2h` | Odds market type |
| `ODDS_API_MIN_CONFIDENCE` | `0.5` | Match confidence gate |
| `USE_STORED_EVENTS` | `true` | Use cached event files |
| `EVENTS_DIR` | `data/sportsbook_data/events` | Event cache location |

### Trading variables

| Variable | Default | Purpose |
|---|---|---|
| `TRADING_MODE` | `paper` | `paper` or `live` |
| `ENABLE_LIVE_TRADING` | `false` | Live hard gate |
| `TRADING_DRY_RUN` | `true` | Decision-only mode |
| `TRADING_STAKE_USD` | `25` | Per-entry size |
| `TRADING_MAX_POSITIONS` | `5` | Open position cap |
| `TRADING_MAX_DAILY_LOSS_USD` | `100` | Daily loss cap |
| `TRADING_MIN_PROFIT` | `0.02` | Min strategy edge |
| `TRADING_MIN_CONFIDENCE` | `0.75` | Min confidence gate |
| `TRADING_MIN_LIQUIDITY_USD` | `2000` | Liquidity floor |
| `TRADING_MAX_SPREAD` | `0.03` | Spread ceiling |
| `TRADING_CYCLE_SECONDS` | `60` | Loop interval |
| `TRADING_COMPARISON_PATH` | `data/arbitrage_comparison.json` | Trader input |
| `TRADING_JOURNAL_DIR` | `data/trading` | Journal path |

---

## Commands

- `npm run fetch:markets`
- `npm run filter:markets`
- `npm run fetch:odds-comparison`
- `npm run detect:arb`
- `npm run run:auto-trader -- --cycles 1 --dry-run`
- `npm run summarize:session`
- `npm run run:full`

---

## Runbook

Standard flow:

```bash
npm run fetch:markets
npm run filter:markets
npm run fetch:odds-comparison
npm run detect:arb
npm run run:auto-trader -- --cycles 1 --dry-run
npm run summarize:session
```

Paper fills (still simulated):

```bash
TRADING_DRY_RUN=false npm run run:auto-trader -- --cycles 3
```

One command:

```bash
npm run run:full
```

## Output Files

`data/` artifacts:

- `sports_markets.json/csv`
- `arbitrage_data.json/csv`
- `arbitrage_data_filtered.json/csv`
- `arbitrage_comparison.json/csv`
- `directional_arbitrage.json`

`data/trading/` journals:

- `signals.jsonl`
- `risk_events.jsonl`
- `orders.jsonl`
- `fills.jsonl`
- `positions.jsonl`
- `state.json`

---

## Live Mode Guardrails

Live startup is blocked unless:

- `TRADING_MODE=live`
- `ENABLE_LIVE_TRADING=true`
- private key exists (`PRIVATE_KEY` or `PK`)
- proxy wallet exists (`POLYMARKET_PROXY_ADDRESS` or `BROWSER_ADDRESS`)

And in this TS build, live execution still exits intentionally because adapter implementation is deferred.

---

## Troubleshooting

### `401 Unauthorized` on odds fetch

Usually invalid/missing `ODDS_API_KEY`.

```bash
echo $ODDS_API_KEY
```

Update `.env`, then rerun `npm run fetch:odds-comparison`.

### `ENOENT ... arbitrage_comparison.json`

The trader was started before comparison generation.

Run:

```bash
npm run fetch:markets
npm run filter:markets
npm run fetch:odds-comparison
```

### Many denials, few opens

Inspect `data/trading/risk_events.jsonl`.
Tune gradually:

- `TRADING_MAX_POSITIONS`
- `TRADING_MIN_CONFIDENCE`
- `TRADING_MAX_SPREAD`
- `TRADING_MIN_LIQUIDITY_USD`

---

## Tuning Guidance

Recommended progression:

1. run with `TRADING_DRY_RUN=true`
2. observe deny reasons and signal quality
3. adjust one threshold at a time
4. switch to `TRADING_DRY_RUN=false` in paper mode
5. compare session summaries across multiple runs

---

## Security Notes

- Never commit private keys.
- Keep `.env` local.
- Rotate exposed credentials immediately.
- Treat live-mode rollout as a production security event.

---

## Roadmap

- real live execution adapter with explicit order routing
- better soccer alias normalization for matching quality
- richer risk/exit parity with advanced engines
- broader test coverage for pipeline + CLI

---

## Disclaimer

For research and automation development only.
Not financial advice. Use paper mode first.
