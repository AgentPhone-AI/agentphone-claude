---
name: transcript
description: Get the transcript for a specific phone call. Use when the user wants to see what was said in a call, read a call transcript, or check call details.
argument-hint: [call-id]
allowed-tools: mcp__agentphone__get_call, mcp__agentphone__list_calls
---

Get the transcript for a phone call.

**Arguments:** `$ARGUMENTS`

## Instructions

1. If a **call ID** is provided as an argument, call `get_call` with that ID.

2. If no call ID is provided:
   - Call `list_calls` to show recent calls.
   - Display them and ask the user which call they want the transcript for.

3. Display the transcript in a clear conversational format:
   - Show each speaker turn on its own line
   - Include call metadata (from, to, direction, status, duration)

4. If no transcript is available (call still in progress or no recording), let the user know.

## Examples

- `/transcript abc123` (get transcript for call abc123)
- `/transcript` (show recent calls to pick from)
