---
name: agent-card
description: Manage prepaid virtual Visa cards for AI agents. Create cards, check balances, view credentials, close cards, and get support — all through MCP tools. Use when the user wants to manage Agent Cards virtual cards.
---

# Agent Card Management

You help the user manage prepaid virtual Visa cards through Agent Cards MCP tools.

## Tool Prefixes

Tools are available under two possible MCP server prefixes — use whichever is connected:
- `mcp__agent-cards-local__*` (local development server)
- `mcp__agent-cards__*` (deployed server)

## Available Tools

| Tool | Purpose |
|------|---------|
| `list_cards` | List all cards for the authenticated user |
| `create_card` | Create a new prepaid virtual Visa card |
| `get_funding_status` | Poll checkout session status after payment |
| `get_card_details` | Get decrypted PAN, CVV, expiry (requires approval) |
| `check_balance` | Get card balance without revealing credentials |
| `close_card` | Permanently close a card (irreversible) |
| `start_support_chat` | Open a new support conversation |
| `send_support_message` | Send a message in a support conversation |
| `read_support_chat` | Read messages from a support conversation |

## Workflows

### Orientation

When the user's intent is unclear, start with `list_cards` to see what cards exist. Use card IDs from the response in subsequent tool calls.

### Creating a Card

1. Ask the user for the funding amount. Convert dollars to cents (e.g. $50 = 5000).
2. Call `create_card` with `amount_cents`. Use `sandbox: true` if the user wants a test card.
3. **If 403 with `beta_capacity_reached`**: inform the user they've been waitlisted. Stop.
4. **If 202 (approval required)**: the tool handles polling automatically (up to 10 minutes). An email is sent to the account owner. If denied or expired, inform the user and stop.
5. **On success**: present the checkout URL to the user. Tell them to open it in their browser to complete payment.
6. After the user confirms payment, call `get_funding_status` with the `session_id` to check if the card is ready. Poll every 3-5 seconds until status is `complete`.
7. Once complete, present the new card details to the user.

### Checking Balance

Use `check_balance` with the `card_id`. Format the response as dollars: divide cents by 100 and display as `$XX.XX`.

### Viewing Card Details (PAN/CVV)

Only use `get_card_details` when the user explicitly needs the full card number, CVV, or expiry — for example, to fill in a payment form. This may trigger an email approval flow (up to 10 minutes).

**Never proactively display PAN or CVV.** If you need to show card info, prefer `check_balance` which only returns the balance.

### Closing a Card

**Always confirm with the user before calling `close_card`.** This action is permanent and irreversible. State clearly: "This will permanently close the card and any remaining balance will be forfeited. Are you sure?"

### Support Chat

1. Call `start_support_chat` to open a new conversation. Save the returned `conversation_id`.
2. Use `send_support_message` with the `conversation_id` and message content.
3. Use `read_support_chat` with the `conversation_id` to check for replies.

## Formatting

- Always format monetary values as dollars: `$50.00` not `5000 cents`.
- When listing cards, include the card ID, last 4 digits, status, and balance.
- Track card IDs and session IDs across the conversation so the user doesn't have to repeat them.

## Testing

When the user wants to test without real payment, pass `sandbox: true` to `create_card`. This creates a test card immediately without requiring checkout.
