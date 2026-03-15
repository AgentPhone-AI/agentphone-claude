---
name: webhooks
description: Manage AgentPhone webhooks. Use when the user wants to set, view, or delete webhook endpoints for inbound call and SMS events.
argument-hint: [url]
allowed-tools: mcp__agentphone__get_webhook, mcp__agentphone__set_webhook, mcp__agentphone__delete_webhook
---

Manage AgentPhone webhook configuration.

**Arguments:** `$ARGUMENTS`

## Instructions

1. If no arguments, call `get_webhook` to show the current webhook configuration.

2. If a **URL** is provided as an argument, call `set_webhook` with that URL.

3. If the user says "delete" or "remove", call `delete_webhook`.

4. When setting a webhook, remind the user:
   - The URL must be publicly accessible
   - A webhook secret will be returned for verifying payloads
   - Optionally they can specify a context_limit (0-50) for how many recent messages to include

## Examples

- `/webhooks` (show current webhook)
- `/webhooks https://example.com/hook` (set webhook URL)
- `/webhooks delete` (remove webhook)
