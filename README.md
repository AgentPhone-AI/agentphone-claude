# AgentPhone for Claude Code

Give your AI agents phone numbers, SMS, and voice calls — right from Claude Code.

## Setup

1. Install the skill:
   ```bash
   npx skills.sh install agentphone
   ```

2. Authenticate. Two options:
   - **MCP (recommended):** the skill connects to the hosted server at `https://mcp.agentphone.ai/mcp` and you sign in through your browser (OAuth) — no key to paste.
   - **API key:** `export AGENTPHONE_API_KEY=your_key_here` (get one at [agentphone.ai](https://agentphone.ai)). Used for the REST path and scripted MCP use.

3. Start using it:
   ```
   /agentphone create an agent called My Assistant and buy it a phone number
   ```

## What it can do

- Create and manage AI voice agents
- Buy and manage phone numbers
- Make AI-powered outbound calls
- Read SMS conversations and messages
- Set up webhooks for inbound events
- Check account usage and limits
- List available voices for agents

## How it works

This skill connects Claude Code to the [AgentPhone API](https://agentphone.ai) via MCP. Your agents can make and receive phone calls, send and read SMS messages, and handle inbound communication through webhooks.

## Structure

```
SKILL.md              # Entry point: onboarding, quick start, capabilities
references/
  rest-api.md         # Complete REST API reference (every HTTP endpoint)
  mcp-tools.md        # The 28 MCP tools
.mcp.json             # Hosted MCP server config
```

## License

MIT
