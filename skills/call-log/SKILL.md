---
name: call-log
description: View recent phone call history. Use when the user wants to see call logs, recent calls, call history, or past calls.
allowed-tools: mcp__agentphone__list_calls, mcp__agentphone__get_call
---

View recent phone call history.

**Arguments:** `$ARGUMENTS`

## Instructions

1. Call `list_calls` to retrieve recent calls.

2. Display them in a clean format showing:
   - Direction (inbound/outbound)
   - From number
   - To number
   - Status
   - Start time
   - Duration (if ended)

3. If a **call ID** is passed as an argument, call `get_call` to get detailed info including the transcript.

4. If no calls exist, let the user know and suggest `/call` to make one.

## Examples

- `/call-log` (show recent calls)
- `/call-log <call-id>` (show details for a specific call)
