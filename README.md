# baselion-hyperliquid-connector

> **Robust open-source client for the [Hyperliquid](https://hyperliquid.xyz) public API.**

A focused, read-only Python client for Hyperliquid's public endpoints — order book, trades, leaderboards, funding rates, open interest. Designed for researchers and quants who want production-quality networking (reconnection, backoff, jittered retries) without having to roll their own.

> ⚠️ This client is **read-only** by design. If you need to place orders, use the official Hyperliquid SDK. We intentionally keep this package narrow so that it can be a trustworthy research dependency.

## Why another Hyperliquid client?

* **Production networking out of the box** — exponential backoff with jitter, connection-pool reuse, automatic WebSocket resubscription after a drop.
* **Typed responses** — every endpoint returns a dataclass; no dict-digging, no silent schema drift.
* **Order-flow helpers** — aggregate trades into OFI (order-flow imbalance), volume buckets, and funding snapshots with one call.
* **Tested** — unit tests against recorded fixtures + integration tests hitting the real mainnet.
* **Benchmarked** — REST throughput and WebSocket latency measured on every release.

## Install

```bash
pip install baselion-hyperliquid-connector
```

## Examples

```python
from baselion_hyperliquid import HyperliquidClient

client = HyperliquidClient()  # no auth needed for read-only

# Current L2 book
book = client.l2_book("ETH")
print(book.bids[0], book.asks[0])

# Recent funding snapshot
funding = client.funding_rates(coins=["BTC", "ETH", "SOL"])
for f in funding:
    print(f.coin, f.funding_rate, f.premium)

# Leaderboard (top 100 by 30-day PnL)
lb = client.leaderboard(window="30d", limit=100)
for row in lb[:10]:
    print(row.rank, row.address[:10], row.pnl_usd)

# Order-flow imbalance over last 1000 trades
ofi = client.ofi("BTC", n=1000)
print(f"OFI={ofi.value:+.3f}  confidence={ofi.confidence:.2f}")
```

## WebSocket (streaming)

```python
from baselion_hyperliquid import HyperliquidWS

async def on_trade(t):
    print(t.coin, t.side, t.size, t.price)

ws = HyperliquidWS(on_trade=on_trade, auto_reconnect=True)
await ws.subscribe_trades(["BTC", "ETH"])
await ws.run_forever()
```

## Benchmarks (indicative, last release)

| Endpoint | median | p95 | p99 |
|----------|-------:|----:|----:|
| `/info` (universe) | 38 ms | 61 ms | 88 ms |
| `/info` (l2Book) | 42 ms | 69 ms | 104 ms |
| `/info` (candleSnapshot) | 55 ms | 92 ms | 140 ms |

Measured from a São Paulo VPS over 1000 sequential calls. Benchmarks re-run on every release — see `benchmarks/RESULTS.md`.

## Status

Alpha — API may shift. Pin the version. Contributions welcome, especially coverage of endpoints we haven't wrapped yet.

## License

MIT.

---

## About Baselion

Baselion is an institutional-grade quantitative trading intelligence platform, powered by the proprietary FOX OMEGA Engine. Learn more at [baselion.ai](https://baselion.ai).
