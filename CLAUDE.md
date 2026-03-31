# AgentPhone Skill

This skill provides telephony capabilities through the AgentPhone MCP server.

## Usage Notes

- Outbound calls and SMS are billable actions — confirm before executing
- Releasing a phone number is irreversible (the number returns to the carrier pool)
- Deleting an agent cannot be undone
- Phone numbers without a country code default to US (+1)
- Call transcripts are available after a call completes
- The `account_overview` tool provides a quick snapshot of the current account state
- The `list_voices` tool shows available voice options for agent configuration
