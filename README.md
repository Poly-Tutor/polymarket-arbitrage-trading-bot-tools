# Polymarket Crypto Trading Bot Suite

**Educational and research tooling** for [Polymarket](https://polymarket.com) short-horizon crypto **Up / Down** markets. This repository bundles **three production-style Python bots** with different signal philosophies: microstructure + VWAP (BTC), multi-asset late consensus (BTC/ETH/SOL/XRP), and oracle-vs-strike (PTB) rules with optional intramarket exits.

| Resource | Link |
|----------|------|
| **Repository** | [github.com/AlterEgoEth/polymarket-crypto-trading-bot](https://github.com/AlterEgoEth/polymarket-crypto-trading-bot.git) |
| **Contact (Telegram)** | [@AlterEgo_Eth](https://t.me/AlterEgo_Eth) |

---

## Who this is for

- **Developers** who want readable Python, WebSocket market data, and CLOB execution patterns to study or extend
- **Traders** who understand that **no bot guarantees profit** and who will paper-test, size small, and own their risk.
- **Clients / integrators** evaluating automation on Polymarket-compatible flows (wallet, API keys, redeem paths)

Everything here is provided **for education and experimentation**. Trading prediction markets can result in **total loss** of capital deployed. Past backtests or anecdotal results **do not** predict live performance.

If you want **additional strategies**, **custom deployment**, or **professional risk and sizing frameworks** beyond this open suite, reach out on **[Telegram @AlterEgo_Eth](https://t.me/AlterEgo_Eth)**. If you are looking for **bots oriented toward live profitability**—more advanced signals, sizing, and execution than this public suite—**please contact the same Telegram**; what is available, how it is shared, and any terms are discussed **individually**

---

## Bots in this repository

| Directory | Focus | Markets | Core idea |
|-----------|--------|---------|-----------|
| [`btc-binary-VWAP-Momentum-bot/`](btc-binary-VWAP-Momentum-bot/) | VWAP, deviation, momentum, z-score | BTC **5m** or **15m** | Enter the **favorite** only when price has **pulled above VWAP** with **positive momentum** inside a **late, narrow time window**—filtering for “consensus + short-term continuation.” |
| [`up-down-spread-bot/`](up-down-spread-bot/) (**Meridian**) | Late Entry V3 (`late_v3`) | BTC, ETH, SOL, XRP — **5m** or **15m** | In the **last minutes**, buy the side the book **already favors**, but only if **spread** and **confidence** (ask skew) pass sanity checks; **stop-loss** and **flip-stop** cut bad paths before expiry. |
| [`5min-15min-PTB-bot/`](5min-15min-PTB-bot/) | PTB diff + probability triggers | BTC **5m** or **15m** | Compare **live BTC** to Polymarket’s **price-to-beat (PTB)** for the window; fire when **time**, **dollar diff**, and **implied probability** align; manage risk with **take-profit / stop-loss** on token prices. |

Each folder has its own **README** (and Meridian has a deep dive under `docs/`). Start there for install paths, env vars, and config.

---

## Why these approaches can make money (and when they do not)

None of the following is investment advice; it is **mechanics**.

### 1. VWAP + momentum (BTC binary bot)

- **Economic story:** In the last slice of a binary window, the “favorite” often trades at a **high implied probability** (e.g. $0.75–$0.88). If the crowd is **directionally right often enough**, buying the favorite can have **positive expectancy** even though each win is small in dollar terms per share.
- **Why filters matter:** Requiring **deviation from VWAP** and **positive momentum** tries to avoid **chasing stale prices** and favors entries where **recent flow** supports the favorite.
- **Failure modes:** Sharp reversals into the close, thin books, or regimes where **implied odds are miscalibrated** can make “favorite following” lose fast. Break-even win rate ≈ **entry price** (e.g. $0.82 entry ⇒ need ~82% wins before fees).

### 2. Late consensus + skew (Meridian / `late_v3`)

- **Economic story:** Nearer expiry, **information about the fixing** is more concentrated in prices; the order book’s **ask skew** (`|up_ask − down_ask|`) is used as a **confidence** proxy. You trade **less time exposed** but pay **higher prices** when consensus is strong.
- **Risk layer:** **Per-market investment cap**, **max entry price**, **stop-loss** (fixed or percent), and **flip-stop** (exit if your side loses “favorite” status) are explicit **risk overrides**—they can cap damage but also **stop out** before a recovery.
- **Failure modes:** **Last-minute noise**, oracle quirks, or **one-sided liquidity** can flip perceived favorites. **Spread > ~$1.05** is treated as unreliable in the default logic.

### 3. PTB distance + probability bands (5m/15m PTB bot)

- **Economic story:** If **spot** is consistently **on one side of PTB** by **$X** late in the window, the **UP** or **DOWN** token may still be **underpriced** vs that physical gap—rules try to catch **alignment** between **oracle BTC**, **strike**, and **token price**.
- **Risk layer:** After a fill, **take-profit** and **stop-loss** are defined in **probability space** (token price), aiming to **lock in gains** or **cut losses** before resolution.
- **Failure modes:** **Lag** between feeds, **PTB definition** vs your intuition, and **simulation vs live** fill behavior. Always validate with **`SIMULATION_MODE=true`** first.

---

## Risk management (summary)

| Bot | Primary levers |
|-----|----------------|
| **VWAP / momentum** | Price band (`min_price` / `max_price`), **narrow entry window**, **bet size**, optional **hedge** (opposite-side GTD), **FAK** execution with retries, **max entry price** cap. |
| **Meridian** | **Dry run**, **max order / total investment**, **entry window**, **confidence** and **spread** gates, **stop-loss**, **flip-stop**, **entry frequency**, **FAK / FOK** execution behavior. |
| **PTB bot** | **Simulation mode**, **per-trade USDC**, **TP/SL** on probability, **trigger** windows, **market lag** limits, **loop** cadence. |

**Operational hygiene:** dedicated wallet, **never commit keys**, private **RPC**, monitor **logs**, start with **minimum size**.

---

## When to use which bot

| Situation | Sensible starting point |
|-----------|-------------------------|
| You want **one asset (BTC)** and **indicator-style** rules with a **terminal dashboard** | `btc-binary-VWAP-Momentum-bot` |
| You want **several coins** from **one wallet** and **late-window consensus** with **structured exits** | `up-down-spread-bot` (Meridian) |
| You care about **PTB vs Chainlink BTC** and **rule-based** triggers with a **web dashboard** | `5min-15min-PTB-bot` |

You can run more than one bot **only if** you understand **collateral**, **nonce / rate limits**, and **position overlap**—typically use **separate wallets** or **non-overlapping** markets.

---

## Extended strategies & quant stack (separate offerings)

Beyond the three open-source-style bots in this tree, **AlterEgo Eth** maintains **additional strategies** used in professional workflows, including:

- **Position sizing & sequences:** martingale, anti-martingale, Fibonacci scaling (each with different **blow-up** and **recovery** profiles—**not** risk-free).
- **Technical analysis:** RSI, MACD, Bollinger Bands (and combinations with **regime filters**).
- **Probabilistic & execution-aware models:** Bayesian updating of beliefs, **edge** estimation vs market price, **spread** and liquidity-aware quoting, **Avellaneda–Stoikov–style** inventory / skew control, **Kelly**-style sizing (often **fractional Kelly** in practice), **Monte Carlo** scenario analysis for drawdown and tail risk.

These are **not** all shipped as drop-in folders in this public repository. For **access, customization, licensing, or collaboration**, contact **[Telegram: @AlterEgo_Eth](https://t.me/AlterEgo_Eth)**.

---

## Quick start (from clone)

```bash
git clone https://github.com/AlterEgoEth/polymarket-crypto-trading-bot.git
cd polymarket-crypto-trading-bot
```

Then open the **README** inside the bot you want:

- `btc-binary-VWAP-Momentum-bot/README.md`
- `up-down-spread-bot/README.md` (overview) and `up-down-spread-bot/docs/README.md` (full guide)
- `5min-15min-PTB-bot/README.md`

---

## License & disclaimer

Individual bots may ship their own **LICENSE**; where none is specified, treat usage as **at your own risk**. Authors and contributors are **not** responsible for trading losses, bugs, exchange rule changes, or regulatory issues in your jurisdiction.

**Not financial advice.** **No warranty.** Use **simulation / dry-run** until you trust the full stack.
