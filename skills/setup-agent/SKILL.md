---
name: setup-agent
description: Create a new AI agent and optionally buy and attach a phone number. Use when the user wants to set up a new agent, get started with AgentPhone, or create an agent with a number.
argument-hint: [agent-name]
allowed-tools: mcp__agentphone__create_agent, mcp__agentphone__buy_number, mcp__agentphone__attach_number, mcp__agentphone__list_agents
---

Set up a new AI agent with AgentPhone, optionally buying and attaching a phone number.

**Arguments:** `$ARGUMENTS`

## Instructions

1. Parse arguments for an agent name. If not provided, ask the user what to name the agent.

2. Call `create_agent` with the name and a sensible description.

3. Ask the user if they'd like to buy a phone number for this agent. If yes:
   - Call `buy_number` with the agent's ID so it gets attached automatically.

4. Summarize what was created:
   - Agent name and ID
   - Phone number (if purchased)
   - Next steps (e.g. "You can now use `/call +1234567890 <topic>` to make calls")

## Examples

- `/setup-agent Customer Support`
- `/setup-agent` (will prompt for name)
