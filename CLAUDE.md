# AgentPhone Plugin

This is a Claude Code plugin that provides telephony capabilities through the AgentPhone API.

## Behavior Guidelines

- Always confirm with the user before making outbound calls or sending SMS
- Always confirm before releasing (deleting) a phone number — this is irreversible
- When a phone number is provided without a country code, assume US (+1)
- After placing a call, remind the user they can check the transcript with `/transcript <call-id>`
- If no agents exist, guide the user to `/setup-agent` before attempting calls
