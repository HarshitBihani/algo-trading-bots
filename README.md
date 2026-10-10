<div align="center">

![Algo Trading Bots](docs/hero.svg)

![Python](https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white)
![Dhan API](https://img.shields.io/badge/Broker_API-Dhan_v2-6d28d9)
![WebSockets](https://img.shields.io/badge/Live_Data-WebSockets-047857)
![SQLite](https://img.shields.io/badge/Storage-SQLite-003B57?logo=sqlite&logoColor=white)
![Status](https://img.shields.io/badge/Project-Personal_build-success)
![Source](https://img.shields.io/badge/Source_code-Private-lightgrey)

**I design, build and operate automated trading systems from market data to risk controls.**
This page shows how they are engineered. Strategy rules, parameters, credentials and trade data are not published.

</div>

---

## At a glance

| | |
|---|---|
| **Systems built** | 3 (a live capable options bot, a multi strategy paper platform, a research pipeline) |
| **Strategies running side by side** | 25 in a single process |
| **Market data connections** | 1 shared websocket for all of them |
| **Modes** | Paper trading and live trading, separated by a double safety lock |
| **Safety nets** | Risk caps, loss limit, watchdog, restart reconciliation |
| **Operated** | Through real market sessions |

## System architecture

![Architecture](docs/architecture.svg)

## Order lifecycle and safety

Every order passes through the same gates, from signal to close.

![Order lifecycle](docs/order-lifecycle.svg)

---

## Explore each system

<details>
<summary><b>1. Intraday options bot</b> (click to expand)</summary>

<br>

- Paper and live modes, with a double safety lock before any real order can be placed
- Resting exit orders placed on the exchange instead of relying on polling
- Live websocket price feed with automatic recovery
- Daily risk caps and loss limits
- Watchdog thread that raises an alarm if the main loop stalls
- Safe to restart: reconciles open positions with the broker after a crash
- Local web dashboard for positions, P&L and logs

</details>

<details>
<summary><b>2. Multi strategy paper trading platform</b> (click to expand)</summary>

<br>

- 25 independent strategies run in **one process** on a single shared websocket connection
- Each strategy has its own capital, trade log and rules
- Paper only by design: no live order path exists in the code
- Daily start, monitor and square off routine with a full operations handbook

</details>

<details>
<summary><b>3. Quantitative research pipeline</b> (click to expand)</summary>

<br>

- Systematic search for robust intraday edges on second level data
- Strict workflow against overfitting: train, validation and test splits, walk forward testing, pessimistic fill assumptions
- Results are judged on unseen data only

</details>

---

## Engineering war stories

<details>
<summary><b>The silent order rejection</b></summary>

<br>

NSE option prices must be exact multiples of the tick size. A rounding bug produced invalid prices, and the exchange rejected the orders silently while the bot logged success. Found through a real rejection, fixed with rounding that respects the tick size.

</details>

<details>
<summary><b>The healthy looking dashboard that hid a frozen bot</b></summary>

<br>

The dashboard runs on its own thread, so it kept responding while the trading loop was frozen. This led to a watchdog that raises an alarm within about two minutes if the loop stops beating.

</details>

<details>
<summary><b>The IPv4 and IPv6 mix up</b></summary>

<br>

Adding a websocket made the machine connect over IPv6, which broke the broker's IP whitelist for order placement. Fixed by forcing a single IP stack at the socket level, covering every library at once.

</details>

<details>
<summary><b>The orphaned position</b></summary>

<br>

A failed exit used to clear the bot's own tracking, leaving a real position that nothing watched. Exits now only clear tracking after confirmed success, and the end of day square off retries and shouts loudly if anything stays open.

</details>

<details>
<summary><b>The ledger that drifted from reality</b></summary>

<br>

A purely simulated capital ledger can disagree with the broker's real balance. Lesson learned: reconcile against the broker, and credit the ledger whenever a trade is closed by hand.

</details>

---

## Tech stack

`Python` · `asyncio` · `websockets` · `pandas` · `SQLite` · `Flask` · `Dhan API v2`

> **Disclaimer:** Educational and personal project. Not investment advice. Trading involves risk.
