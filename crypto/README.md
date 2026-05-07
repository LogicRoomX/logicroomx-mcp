# LogicRoomX Crypto MCP

Real-time cryptocurrency data for AI assistants. Provides what LLMs physically cannot know on their own.

**MCP Endpoint:** `https://api.logicroomx.com/mcp`

---

## Why This Exists

Large language models have a knowledge cutoff. They cannot tell you:
- The current price spread between Binance and OKX
- Whether a Cash & Carry trade is profitable right now
- What the funding rate is at this moment
- Whether an exchange is under maintenance

This plugin bridges that gap with live exchange data.

---

## Tools

### `get_spread`
Real-time price spread between Binance and OKX.

**Input:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| symbol | string | ✅ | Crypto symbol: BTC, ETH, SOL, etc. |
| taker_fee | number | ❌ | Taker fee per side as decimal. Default: 0.001 (0.1%) |

**Output:** Bid/ask on both exchanges, gross spread %, net spread after fees, best route, profitability verdict.

**Example:**
```json
{
  "symbol": "BTC/USDT",
  "binance": { "bid": 80813.28, "ask": 80813.29 },
  "okx": { "bid": 80827.20, "ask": 80827.30 },
  "spread": {
    "gross_pct": "0.0172%",
    "net_pct": "-0.1828%",
    "profitable": false
  },
  "verdict": "Not profitable after 0.200% fees"
}
```

---

### `get_funding_rate`
Current perpetual contract funding rates on Binance and OKX.

**Input:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| symbol | string | ✅ | Crypto symbol: BTC, ETH, SOL, etc. |

**Output:** 8h funding rate, annualized yield, next settlement time, which exchange pays more.

---

### `get_cash_carry`
Cash & Carry arbitrage yield calculator (delta-neutral basis trade).

**Strategy:** Buy spot + short perpetual = collect funding rate with near-zero market risk.

**Input:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| symbol | string | ✅ | Crypto symbol: BTC, ETH, SOL, etc. |
| capital_usdt | number | ❌ | Capital in USDT to estimate profit amounts |

**Output:** Gross annualized yield, net yield after fees, daily/annual profit estimate.

---

### `get_exchange_status`
Live operational status of Binance and OKX.

**Input:** None

**Output:** API reachability, system status, active maintenance incidents.

---

## Integration

### Claude Desktop
Add as a custom connector:
1. Open Claude Desktop → Customize → Connectors → + → Add custom connector
2. Name: `LogicRoomX Crypto`
3. URL: `https://api.logicroomx.com/mcp`

### Any MCP-compatible client
```
MCP Endpoint: https://api.logicroomx.com/mcp
Protocol: MCP Streamable HTTP (2025-03-26)
Legacy SSE: https://api.logicroomx.com/sse
```

### REST API (any AI with function calling)
```
GET https://api.logicroomx.com/api/spread?symbol=BTC
GET https://api.logicroomx.com/api/funding-rate?symbol=BTC
GET https://api.logicroomx.com/api/cash-carry?symbol=BTC&capital_usdt=10000
GET https://api.logicroomx.com/api/exchange-status
```

---

## Changelog

### v1.0.2 — 2026-05-06
- Migrated to MCP Streamable HTTP transport (2025-03-26 spec)
- Kept legacy SSE endpoint for backward compatibility
- Fixed: multiple concurrent connections now handled correctly

### v1.0.1 — 2026-05-06
- Fixed: server crash on multiple SSE connections
- Each connection now uses its own MCP server instance

### v1.0.0 — 2026-05-05
- Initial release
- Tools: get_spread, get_funding_rate, get_cash_carry, get_exchange_status

---

## Feedback

Found a bug or want a feature? Open an issue on this repo or call the feedback endpoint:

```bash
curl -X POST https://api.logicroomx.com/feedback \
  -H "Content-Type: application/json" \
  -d '{"type": "bug", "message": "describe the issue here"}'
```

---

Built by [LogicRoomX](https://logicroomx.com)
