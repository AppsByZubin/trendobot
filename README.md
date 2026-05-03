# Trendobot

Trendobot is a Python trading bot focused on NIFTY 50 option strategies using
Upstox market data and order APIs. The current live engine routes `nifty50`
through the `vwma_ema_st` strategy and supports mock, sandbox, and production
execution modes.

This project is automation tooling for trading research and execution. Review
the strategy, parameters, and order-manager behavior carefully before using
real capital.

## What It Does

- Connects to Upstox market data streams.
- Builds NIFTY 50 index and futures candles from live ticks.
- Selects relevant option contracts around ATM/ITM strikes.
- Runs the VWMA/EMA/Supertrend strategy.
- Places and manages orders through mock, sandbox, or production managers.
- Writes order logs, event logs, and daily PnL outputs under `files/execution_results/`.
- Generates end-of-day/monthly order summaries.

## Project Layout

```text
trendobot/
├── main.py                         # CLI entrypoint
├── index/orchestrator.py           # Top-level instrument/strategy router
├── index/nifty50/nifty50_engine.py # NIFTY 50 live engine and websocket loop
├── index/nifty50/strategy/         # Strategy implementations
├── order_manager/                  # Mock and Upstox order managers
├── broker/                         # Upstox helper/wrapper
├── technicals/                     # Indicator helpers
├── utils/                          # Shared utilities
├── files/param.yaml                # Runtime strategy parameters
└── files/execution_results/        # Generated order/PnL logs
```

## Requirements

- Python 3.10+
- Upstox API credentials
- TA-Lib native library installed on the system
- Python packages from `requirements.txt`

Install Python dependencies:

```bash
python3 -m pip install -r requirements.txt
```

If `TA-Lib` fails to install, install the native TA-Lib package for your OS
first, then rerun the Python dependency install.

## Configuration

Strategy parameters live in:

```text
files/param.yaml
```

Common settings include:

- `trade-per-day`
- `max-open-trades`
- `take-profit`
- `stop-loss`
- `sl-limit-gap`
- `trade_expiry`
- `itm_strike_range`
- `lot-size`
- `trade-window`
- `historical-trends`

Upstox credentials are read from environment variables:

```bash
export upstox_api_access_token="..."
export upstox_sandbox_api_access_token="..."
```

The production token is required for market data. Sandbox token is additionally
required when running with `-l sandbox`.

## Run

Mock mode:

```bash
python3 main.py -i nifty50 -s vwma_ema_st -l mock
```

Sandbox mode:

```bash
python3 main.py -i nifty50 -s vwma_ema_st -l sandbox
```

Production mode:

```bash
python3 main.py -i nifty50 -s vwma_ema_st -l production
```

Valid execution modes are:

- `mock`
- `sandbox`
- `production`

Valid strategy currently wired through the NIFTY 50 engine:

- `vwma_ema_st`

## Output Files

Execution artifacts are written under mode-specific folders:

```text
files/execution_results/mock/
files/execution_results/sandbox/
files/execution_results/prod/
```

Typical outputs:

- `order_log.csv`
- `order_event_log.json`
- `order_status_log.csv`
- `daily_pnl.csv`

## VWMA/EMA/Supertrend Strategy Notes

The `vwma_ema_st` strategy:

- Uses VWMA, EMA, RSI moving average, ATR, and Supertrend filters.
- Enters CALL/PUT option trades based on directional setup and configured bias.
- Normalizes websocket feed messages before candle building and trade processing.
- Manages open trades through the configured order manager.

## Development Checks

Compile a changed file:

```bash
python3 -X pycache_prefix=/tmp/codex-pyc -m py_compile index/nifty50/strategy/vwma_ema_st.py
```

Run a quick import/compile sweep as needed before live use.

## Safety Notes

- Keep access tokens out of source control.
- Use `mock` mode first when changing strategy or order-manager code.
- Confirm `files/param.yaml` expiry dates before each trading session.
- Review generated order logs after every run.
- Production mode can place real orders.
