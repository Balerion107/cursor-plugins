---
name: X Money guide
description: >-
  Read this before the first X Money action in a session and again on any
  X Money error, refusal, or missing capability. Covers how the connection
  works (connection code plus passkey in the X app), how to walk the user
  through connect, reconnect, and revoke at x.com/i/money/settings/connections,
  what each refusal means, and the approval rule for any action that moves
  money or creates a card.
---

# X Money guide

This plugin uses the **X Money MCP** at `https://mcp.money.x.com/mcp`. The user connects once. The Cursor backend holds the tokens and refreshes them. The agent never sees tokens, card numbers in chat, passwords, or passkeys.

Discover the available actions from the server's tool list. The server is the source of truth for what the user can do and for every schema. Do not assume an action exists because it is mentioned here.

## Approval rule

Sending money, requesting money, and creating a virtual card move real money. Before each such call, ask the user for approval with a `SendToUser` widget that states the exact action in one sentence: amount, recipient or merchant, and purpose. Call the tool only after the user picks Approve. Never batch approvals. Never infer approval from an earlier message.

A tool result with `outcome: "refused"` is final. Relay its `message` to the user in plain words. Do not retry a refused call and do not change the amount to get around it.

## How the connection works

1. The user taps **Connect** on the X Money plugin in Grok Bot.
2. A browser opens the X Money consent page. It shows a **connection code** and a QR code.
3. The user opens the code in the **X app** on the account that owns the X Money account, then approves with their **passkey**.
4. The consent page detects the approval and returns to Grok Bot. The connection is complete.

The connection code expires after **5 minutes**. Access tokens live 15 minutes and refresh automatically. The connection stays valid until the user revokes it or it goes unused for 45 days.

## Connections screen

`https://x.com/i/money/settings/connections` — in the X app: **Money → Settings → Connections**.

Send the user here to:

- see which agents are connected to X Money,
- revoke Grok Bot's access,
- confirm that a connection they just approved is listed.

Revoking on this screen ends the connection immediately. The next Grok Bot call fails with an auth error and the user must connect again.

## Troubleshooting

Match the situation, say the quoted line in your own voice, then give the one next step. Do not explain OAuth, tokens, or backend internals.

### Not connected, or auth error on a call

Signals: no X Money tools in the tool list, `401`, `invalid_token`, `grant has been revoked`, `refresh token has expired`, `refresh token does not exist`.

> Your X Money connection is not active. Open the X Money plugin in Grok Bot and tap Connect, then approve the connection code in the X app with your passkey.

If the user says they did not revoke anything, still reconnect. A revoked or expired connection cannot be restored any other way.

### Connection code expired

Signal: the consent page says the code expired, or the user waited more than 5 minutes.

> The connection code expired. Tap Connect again in Grok Bot to get a new code, then approve it in the X app within 5 minutes.

### Code rejected in the X app

Signal: the X app says the code is invalid.

Ask which X account they are signed into. The code must be approved from the X account that holds the X Money account. If it is the right account, tell them to restart from Connect. Codes are single-use and tied to one connection attempt.

### Passkey step fails

Signal: the X app cannot complete the passkey challenge.

> Approving an agent connection needs a passkey on your X account. Set one up in X → Settings → Security, then approve the connection again.

### New connections not accepted

Signal: `new customer connections are not currently accepted`.

> X Money is not accepting new agent connections for your account right now. This is a staged rollout, not a problem with your account. Try again later.

Do not suggest workarounds.

### Action missing or "not ready for you yet"

Signals: an action the user asks for has no matching tool while other X Money tools work, or a call returns `This X Money MCP tool isn't ready for you yet.`

> That X Money action is not enabled for your account yet.

Offer the actions that are present. Do not tell the user to reinstall or reconnect; the tool set is decided per account by X Money.

### User has no X Money account

Signal: the consent page or the X app says the user cannot use X Money, or the user says they never set it up.

> X Money is available to eligible customers in the United States. Set up X Money in the X app first, then connect it here.

## Refusals when sending or requesting money

Relay the `message` from the result. Common ones and what to add:

| Message contains | Add |
| --- | --- |
| `verify this payment in the X Money app` | The user can send it themselves in the X app. Agents cannot complete verification steps. |
| `sending limits` or `Too many transfers` | Agent payments have their own daily caps on top of the account limits. Try again later or pay in the X app. |
| `balance is too low` | Check the balance and suggest a smaller amount, or adding funds in the X app. |
| `No X user named` | Confirm the @handle. Handles change; a numeric user id is more stable. |
| `only accepts money from people they follow` | Nothing to fix from here. |
| `could not confirm` | Do not send again. Tell the user to check the transaction in the X app. |
| `Agents can't move money for this account right now` | Agent payments are switched off for this account. Pay in the X app. |

## Virtual cards

- One card is valid for one merchant and one purchase. Never reuse a card or split a purchase across cards.
- A result that says the daily card count or spend is used up includes `limits_reset_at`. Tell the user when limits reset. Do not create another card.
- Card details go straight into the merchant checkout. Never print the card number, CVC, or expiry in chat.

## What not to do

- Never ask the user for an X Money password, passkey, card number, or bank login.
- Never ask the user to paste a token or code into chat.
- Never retry a `401`, a refusal, or a limits result unchanged.
- Never move money without a fresh approval for that exact action.
