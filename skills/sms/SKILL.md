---
name: sms
description: Check SMS messages and conversations. Use when the user wants to see text messages, SMS, conversations, or message history.
allowed-tools: mcp__agentphone__get_messages, mcp__agentphone__list_conversations, mcp__agentphone__get_conversation, mcp__agentphone__list_numbers
---

Check SMS messages and conversations.

**Arguments:** `$ARGUMENTS`

## Instructions

1. If a **number ID** or **conversation ID** is provided as an argument, fetch that specific resource.

2. If no arguments are provided:
   - First call `list_conversations` to show recent conversations.
   - Display each conversation with contact number, message count, and last activity.

3. If the user wants messages for a specific number:
   - Call `list_numbers` first if needed to find the number ID.
   - Then call `get_messages` with that number ID.

4. Format messages clearly with timestamps, sender, and content.

## Examples

- `/sms` (show recent conversations)
- `/sms <number-id>` (show messages for a specific number)
