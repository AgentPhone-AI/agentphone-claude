---
name: my-agents
description: List all AI agents in the AgentPhone account. Use when the user asks about their agents, what agents they have, or wants to see agent details.
allowed-tools: mcp__agentphone__list_agents, mcp__agentphone__get_agent
---

List all AI agents in the AgentPhone account.

**Arguments:** `$ARGUMENTS`

## Instructions

1. Call `list_agents` to retrieve all agents.

2. Display them showing:
   - Agent name
   - Agent ID
   - Description
   - Attached phone numbers (if any)

3. If an agent ID is provided as an argument, call `get_agent` for detailed info on that specific agent.

4. If no agents exist, suggest the user run `/setup-agent` to create one.
