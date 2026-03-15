---
name: my-numbers
description: List all phone numbers in the AgentPhone account. Use when the user asks about their numbers, what numbers they have, or wants to see their phone numbers.
allowed-tools: mcp__agentphone__list_numbers, mcp__agentphone__release_number
---

List or release phone numbers in the AgentPhone account.

**Arguments:** `$ARGUMENTS`

## Instructions

1. If no arguments, call `list_numbers` to retrieve all phone numbers.

2. Display them in a clean table or list format showing:
   - Phone number
   - Number ID
   - Country
   - Status
   - Attached agent (if any)
   - Created date

3. If the user asks to release/delete a number, call `release_number` with the number ID. Warn that this is irreversible before proceeding.

4. If no numbers exist, suggest the user run `/buy-number` to get one.
