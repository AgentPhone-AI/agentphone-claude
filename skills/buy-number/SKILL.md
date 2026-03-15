---
name: buy-number
description: Buy a new phone number. Use when the user wants to get, purchase, or provision a new phone number.
argument-hint: [country-code]
allowed-tools: mcp__agentphone__buy_number, mcp__agentphone__list_agents
---

Buy a new phone number using AgentPhone.

**Arguments:** `$ARGUMENTS`

## Instructions

1. Parse arguments for an optional country code (2-letter ISO, e.g. US, CA, GB). Default to US if not specified.

2. Optionally check if there's an agent to attach it to by calling `list_agents`. If exactly one agent exists, offer to attach the number to it automatically.

3. Call `buy_number` with the country code (and agent_id if applicable).

4. Report the purchased number to the user.

## Examples

- `/buy-number` (buys a US number)
- `/buy-number CA` (buys a Canadian number)
- `/buy-number GB` (buys a UK number)
