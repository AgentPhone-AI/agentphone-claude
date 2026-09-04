# AgentPhone REST API Reference

Complete reference for the AgentPhone HTTP API. Base URL: `https://api.agentphone.ai`.

For the agent-facing overview and onboarding flow, see [SKILL.md](../SKILL.md). To use the tools through MCP instead of raw HTTP, see [mcp-tools.md](./mcp-tools.md).

## Authentication

Every `/v1/*` endpoint accepts your API key as a Bearer token:

```
Authorization: Bearer sk_live_...
```

The `/v0/agent/sign-up` and `/v0/agent/verify` endpoints are the only unauthenticated ones. API keys and dashboard session tokens are both accepted. To scope an API key to a child account, add the `X-Sub-Account-Id` header. Missing or invalid auth returns `401`.

All phone numbers use **E.164** format (`+14155551234`).

---

## Signup — `/v0/agent`

Self-serve, two-step, unauthenticated. Step one emails a 6-digit code; step two verifies it and atomically provisions everything.

### POST /v0/agent/sign-up
Begin signup and email an OTP. Nothing is provisioned yet.

| field | type | required | notes |
|---|---|---|---|
| `human_email` | string | yes | 6–200 chars, must contain `@`; disposable domains rejected |
| `agent_name` | string | no | truncated to 100 chars; auto-generated from the email if omitted |

Returns `verification_id`, `human_email`, `expires_at` (10-minute expiry), `message`. An existing account for that email returns **409** — fall back to asking the human for a dashboard API key. Rate limit 10/min.

### POST /v0/agent/verify
Verify the OTP and provision User + Account + a starter hosted Agent + a US number + API key, atomically.

| field | type | required | notes |
|---|---|---|---|
| `verification_id` | string | yes | from sign-up |
| `otp_code` | string | yes | the 6-digit code |

Returns `account_id`, `agent_id`, `number_id`, `phone_number`, `api_key` (**plaintext, shown once — save it**). Expired/incorrect codes return `400` (a transient failure is retryable with the same code). No US number available → `503`. Rate limit 20/min.

---

## Numbers — `/v1/numbers`

US/CA only. You get one number on signup.

### POST /v1/numbers
Provision a new SMS-enabled number.

| field | type | required | default | notes |
|---|---|---|---|---|
| `country` | string | no | `"US"` | 2-letter ISO; **US/CA only** |
| `areaCode` | string | no | — | exactly 3 digits |
| `agentId` | string | no | — | attaches the number to this agent on creation |

Returns a number object: `id, phoneNumber, country, status, type` (default `"sms"`), `agentId`, `outboundSms` (`enabled` \| `activating` \| `registration_required`), `createdAt`. Number-limit reached → `409`; billing disabled → `402`. No inventory for a specific area code → `409` (the `detail` suggests alternative area codes); country-wide outage → `503`. Rate limit 20/min.

### GET /v1/numbers
List active numbers. Query: `limit` (default 20, max 100), `offset`. Returns `{data, hasMore, total}`.

### GET /v1/numbers/{number_id}
Get one number (returns it even if released).

### GET /v1/numbers/{number_id}/messages
Messages for a number. Query: `limit` (default 50, max 200), `before`/`after` datetime cursors. Scoped to the current owner, so a transferred number hides the prior owner's history.

### GET /v1/numbers/{number_id}/calls
Calls for a number. Query: `limit` (default 20, max 100), `offset`.

### GET /v1/numbers/lookup
Line-type intelligence (mobile/landline/voip, country, RCS capability). Query: `phoneNumbers` (comma-separated E.164, **max 25**). **Billed $0.009 per number.** Returns `{data: [{phoneNumber, lineType, country, rcsEnabled}]}`. Rate limit 30/min.

### DELETE /v1/numbers/{number_id}
Release a number. **Irreversible; no proration/refund.** History is retained. Already-released → `400`. Rate limit 60/min.

---

## Messages — `/v1/messages`

One endpoint for SMS, iMessage, and WhatsApp. The platform delivers over iMessage automatically when both sides support it and falls back to SMS/MMS otherwise; the response `channel` (`sms` \| `mms` \| `imessage` \| `whatsapp`) tells you how it went out. iMessage/WhatsApp unlock threaded replies, effects, reactions, and richer content; on SMS those extras are ignored.

### POST /v1/messages
Send a message (or start an iMessage group).

| field | type | required | notes |
|---|---|---|---|
| `agent_id` | string | conditional | sender selector (lowest priority) |
| `to_number` | string | conditional | recipient of an existing chat: phone / short code / email / `grp_...`. Mutually exclusive with `recipients` |
| `recipients` | string[] | conditional | start a NEW iMessage group (2–32 members); returns a new `grp_` id. iMessage only |
| `body` | string | yes* | may be empty when sending media/template only |
| `media_url` | string | no | single attachment (mutually exclusive with `media_urls`) |
| `media_urls` | string[] | no | 1–20 attachments, all `https` (iMessage image carousel) |
| `number_id` | string | no | explicit number to send from (overrides agent) |
| `from_number` | string | no | exact E.164 to send from (highest-priority selector) |
| `channel` | string | no | only valid value is `"whatsapp"` (force the WhatsApp binding) |
| `send_style` | string | no | **iMessage only** — send effect (list below) |
| `reply_to_message_id` | string | no | **iMessage + WhatsApp only** — thread under an earlier message in the same chat |
| `buttons` | string[] | no | **WhatsApp only** — 1–3 quick-reply buttons, ≤20 chars each |
| `list` | object | no | **WhatsApp only** — a list menu (≤10 options/rows) |
| `cta` | object | no | **WhatsApp only** — call-to-action URL button (`display_text` ≤20, `url` https) |
| `template` | object | no | **WhatsApp only** — a Meta-approved template (`name` required, `language` default `en_US`, `variables`) |

Returns `id, status` (`"sent"`), `channel, from_number, to_number` (resolved group id), `conversation_id, media_urls, reply_to_message_id, reply_parent_unresolved`.

Notes: restricted/emergency destinations → `403`; unregistered 10DLC numbers → `403` with a registration link; WhatsApp requires the account's WhatsApp binding to be enabled; the WhatsApp rich fields (`buttons`/`list`/`cta`/`template`) are mutually exclusive with each other — `cta`, `list`, and `template` carry no media, but `buttons` may include `media_urls` to render an image/video header. Rate/size limits are enforced server-side.

**`send_style` values (iMessage only):** `celebration`, `fireworks`, `lasers`, `love`, `confetti`, `balloons`, `spotlight`, `echo`, `invisible`, `gentle`, `loud`, `slam`.

**Threaded replies:** set `reply_to_message_id` to an earlier message's `id`. If it can't be threaded (e.g. it was SMS), the message still sends and the response carries `reply_parent_unresolved: true`.

**Group chats:** to post into an iMessage group, send to its `groupId` (`grp_...`), not an individual member (sending to a member starts a separate 1:1). Inbound group messages arrive on your webhook with `data.group` and `data.senderIdentifier`.

### POST /v1/messages/{message_id}/reactions
React to a message with a tapback or emoji.

| field | type | required | notes |
|---|---|---|---|
| `reaction` | string | yes | a classic tapback or a single emoji |

Classic tapbacks (work on **every** iMessage line): `love`, `like`, `dislike`, `laugh`, `emphasize`, `question`. A **single custom emoji** (e.g. `"🔥"`) is also accepted, but only delivers on newer lines — if the line can't send a custom emoji the API returns `400`, so fall back to a classic tapback. Reactions are **iMessage-only** (SMS → `400`); WhatsApp reactions are also supported on WhatsApp-bound messages. Returns `{id, reaction_type, message_id, channel}`.

---

## Conversations — `/v1/conversations`

A conversation is a thread with one contact (native iMessage/SMS live in one thread; WhatsApp is a separate thread on the same number). The 1:1 detail view returns a `capabilities` object telling you what the chat supports (which channels can send, whether reactions/emoji/media work, the WhatsApp 24h window expiry).

### GET /v1/conversations
List. Query: `limit` (default 20, max 100), `offset`. Sorted by most recent. Returns `{data, hasMore, total}` of conversation summaries (`id, agentId, phoneNumber, participant, channel, isGroup, groupId/Name, messageCount, lastMessageAt, lastMessagePreview, metadata, ...`).

### GET /v1/conversations/{conversation_id}
Get a conversation plus recent messages and a `capabilities` block. Query: `message_limit` (default 50, max 100).

### GET /v1/conversations/{conversation_id}/messages
Paginated messages. Query: `limit` (default 50, max 200), `before`/`after` cursors.

### PATCH /v1/conversations/{conversation_id}
Update `metadata` (full replace of your custom state) and/or `group_name` (iMessage groups only). Returns the conversation.

### POST /v1/conversations/{conversation_id}/typing
Show a typing bubble. **iMessage only**, best-effort, auto-expires (no stop call). Empty body.

### POST/DELETE /v1/conversations/{conversation_id}/background
Set (`POST` with `{image_url}`, https, ≤10 MB) or clear (`DELETE`, 204) a chat wallpaper. **iMessage + 1:1 only.**

---

## Calls — voice

Outbound PSTN and browser (WebRTC) calls, with live and post-call transcripts. Status renders as `in-progress`, then `completed` or `failed`.

### POST /v1/calls
Place an outbound call. With a `systemPrompt`, the AI runs the conversation autonomously (no webhook needed); without it, each turn is forwarded to your configured webhook.

| field | type | required | default | notes |
|---|---|---|---|---|
| `agentId` | string | yes | — | must have a number attached |
| `toNumber` | string | yes | — | E.164 |
| `initialGreeting` | string | no | — | first line; `""` = the agent stays silent until spoken to |
| `systemPrompt` | string | no | — | when set, runs the built-in LLM (autonomous) instead of a webhook |
| `modelTier` | string | no | — | `turbo` \| `balanced` \| `max`; only with `systemPrompt` |
| `variables` | object | no | — | `{{var}}` substitution (hosted/systemPrompt only) |
| `voice` | string | no | — | voice-id override for this call |
| `fromNumberId` | string | no | — | caller-ID number (must belong to the agent) |
| `callScreeningIdentity` | string | no | — | ≤100 chars |
| `callScreeningPurpose` | string | no | — | ≤300 chars |
| `disableRecording` | boolean | no | `false` | **when true, no audio recording is stored** (transcript is still captured and delivered). Use for two-party-consent situations. Independent of the account-level recording add-on. |

Returns immediately with `{id, agentId, phoneNumberId, fromNumber, toNumber, direction, status, startedAt}` — the transcript populates after the call ends. Outbound calls require a real payment method on file (anti-abuse) → `402` otherwise. Concurrency caps return `409`/`429` with `Retry-After`. Rate limit 120/min.

### POST /v1/calls/web
Create a browser/WebRTC call and get a Retell access token for the Web SDK (**token expires ~30s** after creation). Body: `agentId` (required), `metadata`, `variables` (hosted only). Rate limit 30/min.

### GET /v1/calls
List calls. Query: `limit` (default 20, max 100), `offset`, `status`, `direction`, `type` (`pstn` \| `web`), `search`.

### GET /v1/calls/{call_id}
Get a call plus transcript turns. If still `in-progress`, transcripts are partial — re-poll until `completed`/`failed`. `recordingUrl` is present only when the account has the recording add-on enabled and a recording exists.

### POST /v1/calls/{call_id}/end
Terminate an in-progress call. Only valid while `in_progress` (else `409`). Returns `{id, status: "ending"}`; the real status arrives asynchronously.

### GET /v1/calls/{call_id}/transcript
Flat, ordered transcript: `{callId, status, ..., transcript: [{role: "user"|"agent", content, createdAt}]}`.

### GET /v1/calls/{call_id}/transcript/stream
Server-sent-events live transcript. Replays existing turns, then streams `turn` events live, `ended` when the call finishes (15s heartbeats).

### GET /v1/calls/{call_id}/recording
Streams the call recording WAV (available only when the recording add-on is enabled).

---

## Agents — `/v1/agents`

Your agent is your phone persona: name, voice, prompt, model tier, and call-handling behavior. You get one starter agent (hosted mode) on signup. **List your agents before creating a new one** — you probably already have the starter.

### GET /v1/agents/voices
List available voices. Returns `{data: [{voice_id, voice_name, provider, gender, accent, preview_audio_url}]}` — `gender`/`accent`/`preview_audio_url` can be `null`. Use `voice_id` when creating/updating an agent. Default is Skylar — Friendly Guide.

### POST /v1/agents
Create an agent. `name` is required; `systemPrompt` is required when `voiceMode` is `hosted`.

| field | type | default | notes |
|---|---|---|---|
| `name` | string | — | required |
| `voiceMode` | string | `webhook` | `webhook` \| `hosted`. Pass `hosted` explicitly for autonomous AI-agent use. |
| `systemPrompt` | string | — | required for hosted; the agent's instructions on calls |
| `beginMessage` | string | — | greeting when a call connects |
| `voice` | string | platform default | a `voice_id` |
| `modelTier` | string | `balanced` | `turbo` \| `balanced` \| `max` |
| `enableMessaging` | boolean | `true` | hosted agent can send/read texts mid-call |
| `enableBackchannel` | boolean | `true` | filler words ("uh-huh") |
| `transferNumber` | string | — | E.164 (+1 US/CA only) to transfer to on request |
| `voicemailMessage` | string | — | ≤1000 chars |
| `callScreeningIdentity` | string | — | ≤100 chars |
| `callScreeningPurpose` | string | — | ≤300 chars |
| `sttMode` | string | `fast` | `fast` \| `accurate` |
| `ambientSound` | string | `none` | `none` \| `office` \| `coffee-shop` \| `outdoor` |
| `denoisingMode` | string | `noise-cancellation` | or `noise-and-background-speech-cancellation` |
| `maxSilenceMs` | int | 600000 | 10,000–3,600,000 |
| `voiceSpeed` | float | 1.0 | 0.5–2.0 |
| `interruptionSensitivity` | float | 0.8 | 0.0–1.0 |
| `language` | string | `en-US` | BCP-47 locale |
| `customTools` | array | — | hosted-only function tools (max 20; each has `name`, `description` ≤1024, `url` https ≤2048, `method` GET/POST, optional `headers`/`parameters`/`timeoutMs` 1000–120000/`speakDuringExecution`/`executionMessage` ≤500). Reserved names are rejected. |

Rate limit 30/min. Returns the full agent object plus `numbers[]`.

### GET /v1/agents  ·  GET /v1/agents/{agent_id}
List (query `limit` default 20 max 100, `offset`) / get one.

### PATCH /v1/agents/{agent_id}
Partial update — only the fields you send change; same types/ranges as create. Switching `voiceMode` to `hosted` requires a `systemPrompt`; switching to `webhook` clears the hosted prompt/greeting. Rate limit 20/min.

### DELETE /v1/agents/{agent_id}
Delete an agent. Attached numbers/conversations/calls are kept with `agentId` cleared. Rate limit 30/min.

### POST /v1/agents/{agent_id}/numbers  ·  DELETE /v1/agents/{agent_id}/numbers/{number_id}
Attach (`{numberId}`) / detach a number. Detaching keeps the number active and re-attachable.

### GET /v1/agents/{agent_id}/conversations  ·  GET /v1/agents/{agent_id}/calls
Agent-scoped lists (query `limit`, `offset`).

---

## Contacts — `/v1/contacts`

A simple address book.

### GET /v1/contacts
List. Query: `search` (name or number), `limit` (default 50, max 200), `offset`. Returns `{data, hasMore, total}`.

### POST /v1/contacts
Create (**201**). Body: `phoneNumber` (required, E.164), `name` (required), `email`, `notes`. Duplicate phone → `409`.

### GET /v1/contacts/capabilities
Live channel check for a number without sending. Query: `phone_number` (E.164). Returns `{phoneNumber, capabilities: {imessage, sms}, checkedAt}`. Rate limits 60/min and 1000/day.

### GET/PATCH/DELETE /v1/contacts/{contact_id}
Get / partial-update (duplicate phone → `409`) / delete (**204**).

---

## Usage — `/v1/usage`

Pay-as-you-go — no per-month message or minute caps. `numbers.limit` is the account's self-serve hold limit; read it from the response rather than assuming a fixed number (contact us to raise it).

### GET /v1/usage
Summary: `{numbers: {used, limit, remaining}, stats: {totalMessages, messagesLast24h/7d/30d, smsSegmentsLast30d, totalCalls, callsLast24h/7d/30d, totalWebhookDeliveries, successfulWebhookDeliveries, failedWebhookDeliveries}, periodStart, periodEnd}`. (There is no `plan` block.)

### GET /v1/usage/daily  ·  /v1/usage/monthly
Time series. Query `days` (default 30, max 365) / `months` (default 6, max 24). Returns `{data: [{date|month, messages, calls, webhooks}], ...}`.

### GET /v1/usage/by-number  ·  /v1/usage/by-agent
Per-number and per-agent breakdowns (`by-agent` takes `period` = `week`\|`month`\|`year` and optional `tz`).

> Account balance and detailed (sub-cent) billing live under the dashboard's billing surface, not `/v1/usage`. `/v1/usage` reports activity counts, not dollar costs.

---

## Verify — `/v1/verify`

Phone verification (send a code, check it) over carrier-registered sender IDs.

### POST /v1/verify/send
Body: `phone_number` (E.164). Sends a 6-digit code (10-minute expiry, max 3 check attempts — not configurable). Returns `{verification_id, status: "pending"}`. Rate limit 60/min.

### POST /v1/verify/check
Body: `verification_id`, `code` (6 digits). Returns `{result, attempts_remaining}` where `result` is `ok` \| `incorrect` \| `exhausted` \| `expired` \| `already_verified`. **Billed $0.05 only on a successful check.** Rate limit 120/min.

---

## Webhooks — `/v1/webhooks` and `/v1/agents/{id}/webhook`

Receive real-time events. Each account has a default webhook; per-agent webhooks override it for that agent.

### Account-level
- **POST /v1/webhooks** — create/update. Body: `url` (https, SSRF-checked), `contextLimit` (0–50, default 10; 0 disables history), `timeout` (5–120s). Returns `{id, url, secret, status, contextLimit, timeout, createdAt}`. Rate limit 10/min.
- **GET /v1/webhooks** — returns the webhook or `null`.
- **DELETE /v1/webhooks** — remove it.
- **GET /v1/webhooks/deliveries** — history. Query `limit` (default 50, max 100), `offset`, `hours` (1–168). Also `/deliveries/stats` and `/deliveries/all-time`.
- **POST /v1/webhooks/test** — send a synthetic inbound event. Rate limit 10/min.

### Per-agent (`/v1/agents/{agent_id}/webhook`)
Same shape: `POST` (create/update), `GET`, `DELETE`, `GET .../deliveries`, `POST .../test`.

### Events
| event | channels | description |
|---|---|---|
| `agent.message` | `sms`, `mms`, `imessage`, `whatsapp`, `voice` | inbound message or voice utterance |
| `agent.call_ended` | `voice` | call finished — includes the full transcript |
| `agent.reaction` | `imessage`, `whatsapp` | inbound tapback/reaction |
| `agent.contact_shared` | — | a contact card was shared |

### Signature verification
Each webhook has a secret (`whsec_...`). Every request is signed: `X-Webhook-Signature: sha256=<hex>` is `HMAC-SHA256(secret, "{timestamp}." + rawBody)`. Other headers: `X-Webhook-Timestamp` (reject if older than ~5 minutes to prevent replay), `X-Webhook-ID`, `X-Webhook-Event`. Delivery is retried on transient (5xx/timeout) failures with backoff up to ~21 hours; permanent 4xx failures aren't retried.

---

## Not covered here

SIP trunking (`/v1/sip-trunks`), sub-accounts (`/v1/sub-accounts`), and RCS (`/v1/rcs`, private/undocumented) exist but are outside this agent-facing reference. Admin, provisioning, and inbound provider webhooks are internal.

## Billable actions (at a glance)

- Number lookup — **$0.009 per number** (`GET /v1/numbers/lookup`)
- Verify — **$0.05 per successful check** (`POST /v1/verify/check`)
- SMS/MMS, voice minutes, and monthly number rental bill per usage (pay-as-you-go). Your $5 signup credit covers your starter number's first month.
