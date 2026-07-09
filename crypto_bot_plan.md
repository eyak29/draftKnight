# Plan: Coinbase Multi-Crypto Trading Bot

## Context
Building a Python-based automated trading bot that scans all available Coinbase USD pairs (~200+) and trades using professional breakout + momentum signals. Bot trading is explicitly permitted by Coinbase's ToS via their Advanced Trade API. The only prohibited uses are market manipulation, wash trading, and screen-scraping.

---

## Rules & Constraints Confirmed
- **Allowed**: Programmatic order placement via Coinbase Advanced Trade API
- **Prohibited**: Wash trading, spoofing, market manipulation
- **Rate limits**: ~30 req/sec authenticated; mitigated by batching and caching
- **Fees**: 0.40% maker / 0.60% taker (drops with volume)
- **Legal**: Automated crypto trading is legal in the US; same rules as human traders apply

---

## Strategy: Confluence of Signals (Pro Framework), 4-Hour Candles

Professional traders don't rely on one indicator — they look for multiple signals confirming the same direction before pulling the trigger. The bot requires confluence across all four categories below.

### Data Sources (all free, minimal auth)
| Data | Source | Auth needed? |
|---|---|---|
| OHLCV candles | Coinbase Advanced API | Yes (already have) |
| Funding rates | Binance Futures public API | No |
| Open Interest | Binance Futures public API | No |
| Stablecoin supply | DeFiLlama API | No |
| BTC price (for relative strength) | Coinbase API | Yes (already have) |

### Pre-Entry Filters (must pass BOTH — coin skipped if either fails)
| Filter | Condition | Why |
|---|---|---|
| Trend direction | Price above 50 EMA | Never fight the trend |
| Volume floor | Current volume > 20-day average | Avoid illiquid/dead coins |

---

### Signal Stack — Confluence of 4 Categories (100 pts total)

#### 1. Trend & Momentum (30 pts)
| Signal | Pts | Condition |
|---|---|---|
| EMA Golden Cross | 15 | 50 EMA above 200 EMA (or 50 EMA recently crossed above 200 EMA within 10 bars) |
| MACD Cross | 10 | MACD line crossed above signal line within last 3 bars |
| RSI Divergence | 5 | Bullish divergence: price made lower low but RSI made higher low; OR RSI trending up from 40–60 zone |

#### 2. Volatility & Breakout (25 pts)
| Signal | Pts | Condition |
|---|---|---|
| Bollinger Band Squeeze | 15 | Band width < 20th percentile of last 60 days AND price breaking above upper band |
| Fibonacci Zone | 10 | Price bouncing from 61.8% retracement of last major swing (golden zone) |

#### 3. Volume & Price Action (20 pts)
| Signal | Pts | Condition |
|---|---|---|
| Volume Breakout | 15 | Price at 20-day high with volume ≥ 2× 20-day average |
| BTC Relative Strength | 5 | Alt's 7-day return > BTC's 7-day return |

#### 4. On-Chain & Sentiment (25 pts)
| Signal | Pts | Condition |
|---|---|---|
| Funding Rate | 10 | Rate < 0.01% — room to run, no overleveraged longs |
| Open Interest | 10 | OI rising alongside price — confirms real momentum |
| Stablecoin Supply | 5 | 30-day stablecoin market cap trend is rising |

**Minimum score to enter: 65/100** — requires signals from at least 3 of the 4 categories

---

## Exit Strategy — Data-Driven, No Fixed Time Limit

**Philosophy:** Let the signals tell you when to exit. No clock pressure — hold as long as momentum supports it. Most trades close within 1–3 days; hard backstop at 10 days. The backtester runs all 25 configurations against 18 months of history and ranks by Sharpe ratio — the winner becomes the default.

### Category A — Fixed Profit Targets
| Config | Stop | Target | R/R | Notes |
|---|---|---|---|---|
| A1 | 2.5% | 5% | 2.0× | High frequency, small wins |
| A2 | 3% | 8% | 2.7× | Sweet spot candidate |
| A3 | 3% | 10% | 3.3× | Needs strong moves |
| A4 | 4% | 15% | 3.75× | Bull market only |
| A5 | 2% | 5% | 2.5× | Tight stop variant |

### Category B — Trailing Exits
| Config | Mechanism | Notes |
|---|---|---|
| B1 | ATR trailing stop (2× ATR) | Classic — how much does it give back? |
| B2 | ATR trailing stop (1.5× ATR) | Tighter trail, exits faster |
| B3 | Parabolic SAR | Self-adjusting, accelerates as price moves up |
| B4 | Chandelier Exit (3× ATR from highest close) | Wider trail, better for volatile coins |
| B5 | Price closes below 8 EMA | Very tight, catches reversals early |

### Category C — Tiered Sells
| Config | Structure | Notes |
|---|---|---|
| C1 | Sell 50% @ 5%, trail 50% | Locks half, lets half run |
| C2 | Sell 33% @ 5%, 33% @ 10%, trail 33% | Three-stage ladder |
| C3 | Sell 25% @ 3%, 25% @ 6%, 25% @ 10%, trail 25% | Four-stage — aggressive |
| C4 | Sell 50% @ 8%, trail 50% (1.5× ATR) | Bigger first exit, tight trail |
| C5 | Sell 50% @ 5%, 25% @ 10%, trail 25% | Asymmetric ladder |
| C6 | Sell 75% @ 5%, trail 25% | Conservative — take most profits early |

### Category D — Signal-Based Exits
| Config | Exit Trigger | Notes |
|---|---|---|
| D1 | MACD histogram peaks & starts declining | Earlier than zero-cross |
| D2 | RSI crosses back below 65 | Earlier exit, less giveback |
| D3 | Volume drops below 50% of entry-day volume | Crowd leaving = move done |
| D4 | Score degradation >25 pts (re-score each candle) | "Would I still enter this?" |
| D5 | Bollinger upper band contracting post-breakout | Volatility collapsing |
| D6 | Two-signal requirement before exiting | Filters noise |

### Category E — Breakeven Stop Variants
| Config | Mechanism | Notes |
|---|---|---|
| E1 | Move stop to breakeven once up 3% | Never lose after quick move |
| E2 | Move stop to entry +1% once up 5% | Guaranteed profit floor |
| E3 | Move stop to breakeven, then trail | Protection + upside |
| E4 | Move stop to +2% once up 8% | Stronger floor |

### Category F — Time + Profit Hybrids
| Config | Rule | Notes |
|---|---|---|
| F1 | Open 3 days and profit < 2% → exit | Free capital for better setups |
| F2 | Open 5 days and profit < 5% → tighten stop to 1% | Pressure slow trades |
| F3 | Day 1–2: wide stop (4%); Day 3+: tight stop (1.5%) | Time-decaying tolerance |
| F4 | After breakeven stop, exit if flat 2 more days | Don't hold dead trades |

### Live Exit Conditions (applied after backtest picks winning config)
| Trigger | Urgency |
|---|---|
| Price hits profit target | IMMEDIATE |
| Price hits stop loss | IMMEDIATE |
| RSI > 78 (overbought) | IMMEDIATE |
| MACD histogram flips negative | NEXT CANDLE |
| Funding rate spikes > 0.05% | NEXT CANDLE |
| Hold time > 10 days | NEXT CANDLE |

---

## Architecture

### Tech Stack
- Python 3.12+, `asyncio`
- `coinbase-advanced-py` (official SDK, wrapped in `run_in_executor`)
- `pandas-ta` for all technical indicators
- `pydantic-settings` for typed config (`.env` + `config.yaml`)
- SQLite for trade/position/scan logging
- `loguru` for structured logging

### Scan Timing
- 4-hour candles (aligned to candle close)
- Scan loop: every 4 hours
- Monitor loop: every 5 minutes (200 pairs in 2 API calls via batching)
- Expected hold time: 1–3 days; hard backstop at 10 days

### Folder Structure
```
crypto_bot/
├── config/
│   ├── settings.py        # Pydantic BaseSettings: .env + config.yaml
│   └── config.yaml        # All trading parameters (no secrets)
├── core/
│   ├── bot.py             # TradingBot: orchestrates all loops
│   └── scheduler.py       # Drift-corrected interval runner
├── market/
│   ├── client.py          # SDK wrapper with retry logic
│   ├── pair_discovery.py  # All USD spot pairs, filter stablecoins
│   └── data_fetcher.py    # Async rate-limited candle cache
├── signals/
│   ├── indicators.py      # pandas-ta: EMA, RSI, MACD, ATR, Bollinger
│   ├── filters.py         # Pre-entry: price > 50 EMA, volume > avg
│   ├── trend.py           # EMA cross + MACD cross + RSI divergence (30 pts)
│   ├── volatility.py      # Bollinger squeeze + Fibonacci zone (25 pts)
│   ├── price_action.py    # Volume breakout + BTC relative strength (20 pts)
│   ├── onchain.py         # Funding/OI (Binance) + stablecoins (DeFiLlama) (25 pts)
│   ├── regime.py          # Market regime: BULL / NEUTRAL / BEAR
│   └── scanner.py         # Filters → score all 4 categories → SignalResult
├── trading/
│   ├── trader.py          # Order execution (real + paper mode)
│   ├── position.py        # Position dataclass + trailing stop logic
│   └── exit_manager.py    # Polls positions, fires all 25 exit configs
├── risk/
│   ├── risk_manager.py    # Pre-trade gate (fail-fast)
│   ├── position_sizer.py  # ATR-aware: risk_amount ÷ stop_distance
│   └── kill_switch.py     # Daily loss circuit breaker
├── backtest/
│   ├── engine.py          # Core loop: no lookahead bias
│   ├── data_loader.py     # Historical OHLCV from Coinbase (up to 2 years)
│   ├── simulator.py       # Fills with slippage + 0.60% fee deducted
│   ├── portfolio.py       # Tracks cash, positions, PnL through history
│   └── reporter.py        # Ranks all 25 exit configs by Sharpe
├── data/
│   ├── database.py        # SQLite schema + connection (WAL mode)
│   ├── models.py          # Trade, Position, ScanResult, DailyStat
│   └── repository.py      # All SQL here + export_tax_report()
├── utils/
│   ├── rate_limiter.py    # TokenBucket + asyncio.Semaphore
│   ├── notifier.py        # Telegram alerts (fire-and-forget async)
│   └── logger.py          # loguru setup
├── tests/
├── .env.example
├── main.py
└── requirements.txt
```

### Risk Management
- 2% portfolio at risk per trade (position size = risk_amount ÷ stop_distance)
- Max 10 simultaneous positions, max 30% total portfolio exposure
- 5% daily drawdown kill switch
- `PAPER_TRADING=true` by default

---

## Additional Considerations

### 1. Market Regime Detection
`signals/regime.py` — checks BTC 20 EMA vs 50 EMA each cycle. Bearish cross → no new entries. BTC down >15% over 30 days → entries paused. Resume when BTC reclaims 20 EMA.

### 2. Telegram Notifications
`utils/notifier.py` — Telegram Bot API, no extra library needed. Setup: @BotFather (5 min). Alerts: trade entered/exited, kill switch, regime change, midnight daily summary.

### 4. Fees in Backtesting
Every simulated fill deducts 0.60% taker fee (~1.2% round trip). Reporter shows gross and net returns side by side.

### 5. API Key Security
Enable View + Trade only. Disable Transfer. Whitelist IP. Store in `.env` only — never commit.

### 6. Tax Tracking
`python main.py --export-taxes 2026` → `taxes_2026.csv` in IRS Form 8949 format.

### 7. New Coin Listing Detection
New coins (<30 days) require score ≥75 (vs 65). Flagged as `is_new_listing` in scan results.

---

## Backtesting

### Free to backtest
EMA cross, MACD cross, RSI divergence, Bollinger squeeze, volume breakouts, Fibonacci zones, BTC relative strength.

### Requires approximation
Funding rates, Open Interest, stablecoin supply (no free historical on-chain data — use regime proxies).

### How to Run
```bash
python -m backtest.run --start 2024-01-01 --end 2025-06-01
python -m backtest.run --coins BTC-USD,ETH-USD,SOL-USD --start 2024-01-01
python -m backtest.run --min-score 70 --exit-config C2
```

### Output per config
Total return (gross + net), win rate, profit factor (target >1.5), avg hold time, max drawdown, Sharpe ratio, per-coin breakdown, equity curve PNG. Top 3 by Sharpe highlighted.

---

## Sequenced Build Order

| Stage | What | Est. Time |
|---|---|---|
| 1 | Config, logging, DB schema, models, repository | 2–3 hrs |
| 2 | Coinbase client, pair discovery, rate limiter, candle cache | 3–4 hrs |
| 3 | Signal Phase 1: EMA, MACD, RSI, Bollinger, Volume | 3–4 hrs |
| 4 | Signal Phase 2: Funding/OI (Binance), stablecoins (DeFiLlama), BTC RS | 2–3 hrs |
| 5 | Backtesting engine | 3–4 hrs |
| 6 | Run backtests — tune thresholds, pick winning exit config | 1–2 days |
| 7 | Signal Phase 3: RSI divergence, Fibonacci swing detection | 3–4 hrs |
| 8 | Risk manager, position sizer, kill switch | 2–3 hrs |
| 9 | Trader, exit manager, scheduler, bot.py, main.py, Telegram | 3–4 hrs |
| 10 | Paper trading 1–2 weeks | Ongoing |
| 11 | Live trading with small capital | After 10 |

**Total: ~25–30 hours. Critical path: Stage 5–6 before any live capital.**

---

## Verification Plan
1. Stage 2: `warm_up_cache` → 150+ pairs load with valid OHLCV
2. Stage 3: `scan_all_pairs` → top 10 results look like real setups
3. Stage 6 gate: at least one config shows Sharpe >0 and profit factor >1.2
4. Stage 9: paper mode 24–48 hrs — all DB tables populate, Telegram alerts fire
5. Pre-live: paper PnL positive ≥2 weeks; kill switch works; restart loads open positions; tax export valid

---

*Project location: `crypto_bot/` — separate repo from draftKnight*
