# AgentPhone for Claude Code

Give your AI agents phone numbers, SMS, and voice calls — right from Claude Code.

## Setup

1. Install the plugin:
   ```bash
   claude plugin add agentphone
   ```

2. Set your API key:
   ```bash
   export AGENTPHONE_API_KEY=your_key_here
   ```
   Get your API key at [agentphone.to](https://agentphone.to).

3. Start using it:
   ```
   /setup-agent My Assistant
   ```

## Commands

| Command | Description |
|---------|-------------|
| `/setup-agent [name]` | Create a new AI agent and buy a phone number |
| `/buy-number [country]` | Buy a phone number (default: US) |
| `/call [number] [topic]` | Make an AI-powered phone call |
| `/call-log [call-id]` | View recent call history |
| `/transcript [call-id]` | Get a call transcript |
| `/sms [number-id]` | View SMS messages and conversations |
| `/my-agents [agent-id]` | List your agents |
| `/my-numbers` | List your phone numbers |
| `/webhooks [url]` | Manage webhook endpoints |

## How it works

This plugin connects Claude Code to the [AgentPhone API](https://agentphone.to) via MCP. Your agents can make and receive phone calls, send and read SMS messages, and handle inbound communication through webhooks.

## License

MIT
