---
name: call
description: Make an AI-powered phone call to someone. Use when the user wants to call a number, have a conversation call, or place an outbound call.
argument-hint: [phone-number] [topic]
allowed-tools: mcp__agentphone__make_conversation_call, mcp__agentphone__make_call, mcp__agentphone__list_agents, mcp__agentphone__list_numbers
---

Make a phone call using AgentPhone.

**Arguments:** `$ARGUMENTS`

## Instructions

1. Parse the arguments to extract the phone number and topic/instructions.
   - The phone number should be in E.164 format (e.g. +14155551234). If the user gives a number without the country code, assume US (+1).
   - Everything after the phone number is the conversation topic.

2. You need an agent_id to make calls. First call `list_agents` to find an existing agent. If no agents exist, tell the user to run `/setup-agent` first.

3. Choose the right call type:
   - If a **topic** is provided, use `make_conversation_call` — this lets the AI hold an autonomous conversation about the topic.
   - If **no topic** is provided, use `make_call` — this initiates a basic outbound call.

4. After placing the call, tell the user:
   - The call ID (so they can check the transcript later with `/transcript`)
   - That the call is in progress
   - Suggest they use `/transcript <call-id>` to check the transcript once the call ends.

## Examples

- `/call +14155551234 schedule a dentist appointment for next Tuesday`
- `/call +14155551234` (basic call, no AI conversation)
- `/call 4155551234 ask about their return policy` (auto-adds +1)
