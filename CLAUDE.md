# AgentPhone Skill

This is a Claude Code skill that provides telephony capabilities through the AgentPhone MCP server.

## Behavior Guidelines

- Always confirm with the user before making outbound calls or sending SMS
- Always confirm before releasing (deleting) a phone number — this is irreversible
- Always confirm before deleting an agent — this cannot be undone
- When a phone number is provided without a country code, assume US (+1)
- After placing a call, remind the user they can check the transcript later
- If no agents exist, guide the user to create one before attempting calls
- Use `account_overview` first when the user wants to see their current state
- Use `list_voices` to show available voices before creating/updating agents with voice settings
