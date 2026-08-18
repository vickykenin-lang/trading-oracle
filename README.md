# Trading Oracle

Autonomous NSE/BSE intraday research, signal, and **paper-trading** system.
Built against [`TRADING_ORACLE_v2_MASTER_SPEC.md`](./TRADING_ORACLE_v2_MASTER_SPEC.md) --
that document is the build contract; this README is a map of where the
project currently stands against it.

## What this is (and isn't)

This system generates research signals and simulates trades on paper. It
does not place real orders. Live execution is explicitly locked (spec
section 21) and will not be enabled implicitly, automatically, or on
request from within a chat session -- it requires a long list of
prerequisites the spec spells out, plus the operator's own explicit,
written go-ahead.

This is not investment advice. Intraday trading carries substantial risk of
loss, and most individual intraday traders in Indian equity markets lose
money over time. See section 22 of the master spec for the full disclaimer.

## Non-negotiables (why the code looks the way it does)

- **Python computes, the LLM narrates.** No indicator value, price, or
  statistic is ever produced by a language model (spec section 2).
- **No candle, no signal.** Stale, incomplete, or unadjusted data blocks
  the signal rather than degrading it (spec section 3).
- **NO TRADE is a valid, successful output.** The system never manufactures
  a setup because someone asked for one (spec section 14).
- **Every number in a rendered report must exist in the source JSON object**
  it was rendered from, or the output is rejected (spec section 13.1, 15.3).

## Phase status

Tracked against the roadmap in spec section 20. A phase is marked done only
when its exit criterion is demonstrably met -- not when it feels finished.

| Phase | Deliverable | Exit criterion | Status |
|---|---|---|---|
| 1 | Repo, config, logging | Config hashing works, logs reproducible | **Done** -- see `tests/unit/test_config.py`, `tests/unit/test_logging.py` |
| 2 | Data layer + integrity contract | All section 3.5 checks passing on live feed | Not started -- needs Zerodha Kite Connect API credentials (see `.env.example`) |
| 3 | Indicators | Unit tests match reference values exactly | Not started |
| 4 | Gates | Every gate demonstrably blocks its condition | Not started |
| 5 | Scanner | Runs a full session without integrity failure | Not started |
| 6 | Strategies + regime gating | Each strategy fires only in permitted regimes | Not started |
| 7 | Risk + sizing engine | Limits provably unbreakable in tests | Not started |
| 8 | Execution realism | Slippage and gap-through-stop simulated | Not started |
| 9 | Paper trading | 60 sessions logged with full audit trail | Not started |
| 10 | Backtest harness | Walk-forward validation clean, no leakage | Not started |
| 11 | Analytics + review | Statistics carry samples and intervals | Not started |
| 12 | Dashboard | All health indicators live | Not started |

## Repository layout

Only directories a completed phase actually needs exist yet. Later phases
add `src/data/`, `src/gates/`, `src/strategies/`, etc. as they start --
see spec section 19 for the full target layout.

```
trading-oracle/
├── config/               versioned, hashed behavioural config
├── src/
│   └── utils/            config loader + hasher, structured logging
├── tests/unit/           unit tests proving each phase's exit criterion
├── logs/                 runtime logs (git-ignored, folder tracked)
├── database/             reserved for Phase 9 (paper trading records)
├── .github/workflows/    CI: install deps, run tests, on every push
├── requirements.txt
├── .env.example          copy to .env, fill in real values, never commit it
└── pytest.ini
```

## Running locally

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env   # fill in real values when Phase 2 needs them
pytest -v
```

## Config and hashing

`config/default.yaml` holds all behavioural settings (risk limits, session
times, cost model, freshness thresholds). `src/utils/config.py` loads it and
computes a deterministic SHA-256 hash (`config_hash`) that will be stamped
on every signal and log record once signal generation exists (Phase 5+),
so any decision can be traced back to the exact config that produced it.

Secrets never enter the hashed config -- they live only in `.env` /
GitHub Secrets, referenced via `src/utils/config.get_env()`.
