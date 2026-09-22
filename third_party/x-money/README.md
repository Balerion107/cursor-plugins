# X Money

Grok Bot plugin that connects agents to [X Money](https://x.com/i/money) through X Money's hosted [Model Context Protocol](https://modelcontextprotocol.io/) server at `https://mcp.money.x.com/mcp`.

Check balances, review transactions, send and request money on X, and pay online with a single-use virtual card.

## Who can use it

- Grok Bot **0.52** or newer.
- An active X Money account. X Money is available to eligible customers in the United States.
- Not available in Cursor. Cursor must not list or install this plugin.

## Install

1. Open **Grok Bot → Plugins**.
2. Search for **X Money**.
3. Click **Install**, then complete the X Money connection when prompted.

Or ask the agent to connect your X Money account.

## MCP

```json
{
  "mcpServers": {
    "x-money": {
      "type": "http",
      "url": "https://mcp.money.x.com/mcp",
      "placement": "server"
    }
  }
}
```

## Connecting

Auth is OAuth 2.1 with PKCE. There is no client ID, API key, or token to paste. The Cursor backend is a registered confidential client of X Money and completes the sign-in on your behalf.

1. Grok Bot opens the X Money consent page in a browser.
2. The page shows a connection code and a QR code.
3. Open the code in the X app and approve it with your passkey.
4. The page redirects back to Grok Bot and the connection completes.

Access tokens expire after 15 minutes and refresh automatically. Tokens are stored encrypted on the Cursor backend and never reach the agent, the chat, or your device. You can revoke the connection at any time from **X → Money → Settings → Connections**.

## What agents can do

| Tool | What it does |
| --- | --- |
| `get_balance` | Available balance of your main, secondary, and joint accounts |
| `get_transactions` | Your transaction history, newest first, with optional filters |
| `send_money` | Send money from your X Money balance to another X user by @handle or user id |
| `request_money` | Ask another X user to pay you through X Money |
| `get_card_details` | Create a single-use virtual card for one online purchase at one merchant |

The hosted server is the source of truth for tool names and schemas.

## Notes

- Payments and cards created through this server are live and move real money.
- Every action passes the same X Money checks as the app. X Money also enforces agent spending limits on top of your account limits. A refused action returns a message for the agent to relay and is not retried.
- Higher-risk transfers, such as bank transfers and large payments, are not available to agents and stay behind verification in the X app.
- Each virtual card works for exactly one merchant and one purchase. Card details are returned to the agent to complete checkout and are not shown in chat.
- The agent never sees your X Money password, passkey, or bank credentials.

## Docs

- X Money: https://x.com/i/money
- Manage connected agents: https://x.com/i/money/settings/connections
- Server URL: https://mcp.money.x.com/mcp

Logo is X Money's official mark.

## License

MIT
