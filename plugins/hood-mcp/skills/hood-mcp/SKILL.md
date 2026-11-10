---
name: hood-mcp
description: Query Robinhood Chain (chain ID 4663) through the hood-mcp data server tools, and set up the separate opt-in trading server with spend caps and a confirm gate. Use for Stock Token quotes, portfolios, memecoin launches and chain stats.
---

# hood-mcp

The `hood-mcp` npm package ships two stdio MCP servers. This plugin's `.mcp.json` starts the read-only data server with `npx -y hood-mcp`. It needs no key and uses the public RPC.

## Data server tools (read-only)

| Tool | Use it for |
| --- | --- |
| `get_chain_stats` | Latest block, gas price, TVL, network totals |
| `list_stock_tokens` | The Stock Token registry: ticker, name, contract, feed |
| `get_stock_quote` | Chainlink price, DEX price, premium or discount for a ticker |
| `get_portfolio` | Multiplier-correct Stock Token positions and USDG balance for any address |
| `get_coin` | Price, volume, liquidity, holders for a token address |
| `list_trending_coins` | Trending pools right now |
| `get_recent_launches` | Recent NOXA and The Odyssey launches from on-chain logs |
| `watch_launches` | Watch for new launches for up to 120 seconds |
| `search_token` | Find a token by ticker, name or address |

Environment variables, all optional: `HOOD_MCP_NETWORK` (`mainnet` default, or `testnet` for chain 46630), `ALCHEMY_KEY` (private RPC), `HOOD_MCP_TRANSPORT=http` with `HOOD_MCP_PORT` (default 8730) for Streamable HTTP serving `POST /mcp` and `GET /health`.

Typical flow: `search_token` to resolve a ticker, `get_stock_quote` for the premium or discount, `get_portfolio` for an address.

## Trading server (opt in)

The trading server is a second bin of the same package, `hood-mcp-trading`. It is not a separate npm package, so launch it with `npx -y -p hood-mcp hood-mcp-trading`. It is stdio only and refuses to start unless both `HOOD_MCP_ENABLE_TRADING=1` and `ROBINHOOD_CHAIN_PRIVATE_KEY` are set. Add it yourself, never through this plugin:

```json
{
  "mcpServers": {
    "hood-mcp-trading": {
      "command": "npx",
      "args": ["-y", "-p", "hood-mcp", "hood-mcp-trading"],
      "env": {
        "HOOD_MCP_ENABLE_TRADING": "1",
        "ROBINHOOD_CHAIN_PRIVATE_KEY": "${ROBINHOOD_CHAIN_PRIVATE_KEY}"
      }
    }
  }
}
```

Tools: `get_my_portfolio` and `get_swap_quote` (read-only), `execute_swap` (`tokenIn`, `tokenOut`, `amountIn`, optional `slippageBps` 1-5000 and `confirm`), and `transfer_usdg` (`to`, `amount`, optional `confirm`).

Guards: spend per call is capped by `HOOD_MCP_MAX_SPEND_USDG` (default 25) and per session by `HOOD_MCP_MAX_SESSION_USDG` (default 100). Buying a Stock Token also needs `HOOD_MCP_ACKNOWLEDGE_ELIGIBILITY=1`. The first `execute_swap` or `transfer_usdg` call only returns a simulation; a second call with `confirm: true` and identical arguments broadcasts.

These calls move real funds. Run the preview, show the user the recipient or pair, amount, token and network, and set `confirm: true` only after an explicit yes. Never put a private key in a file or a prompt.
