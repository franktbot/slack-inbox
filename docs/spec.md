# Agent Project Instructions

## Project: Read-Only Slack Inbox Aggregator (Single Workspace, Slack Connect + DMs)

## Objective

Build a secure Slack Reader middleware and OpenClaw skill that allows
real-time inbox-style scanning of one Slack workspace (including Slack
Connect channels and DMs), using a Slack User OAuth token (xoxp-), with
strict read-only enforcement and full error handling.

The system must: - Never allow posting to Slack. - Never expose Slack
token to OpenClaw. - Handle Slack Connect permission failures
gracefully. - Return structured, human-readable error reports including
channel links. - Be fully testable before requiring human input.

Development must be sequenced to avoid requiring human input until the
final integration stage.

------------------------------------------------------------------------

# Development Phases

------------------------------------------------------------------------

# PHASE 1 --- Core Middleware Skeleton (No Slack Required)

## Issue 1.1 --- Service Framework

Tasks: - Create service repository structure. - Use FastAPI (Python). -
Implement config loader with required env vars: - SLACK_USER_TOKEN (not
yet required at runtime) - OPENCLAW_API_KEY - PORT - LOG_LEVEL -
Implement structured JSON logging. - Implement request ID middleware. -
Implement /health endpoint.

Definition of Done: - Service runs locally. - /health returns version
and timestamp. - Missing config causes explicit startup error.

------------------------------------------------------------------------

## Issue 1.2 --- Slack API Client (Read-Only Enforcement)

Tasks: - Implement Slack client wrapper. - Hardcode allowed Slack
methods: - conversations.list - conversations.history -
conversations.replies - auth.test - Explicitly block any method starting
with: - chat. - reactions. - pins. - Implement rate limit handler
(Retry-After respected). - Implement error normalization layer mapping
Slack errors into structured internal errors.

Definition of Done: - Attempting disallowed method raises internal
exception. - Unit tests verify write methods cannot execute.

------------------------------------------------------------------------

## Issue 1.3 --- Error Model + Human Readable Reporting

Tasks: - Define error response schema: - conversation_id -
channel_permalink - slack_error_code - explanation_human -
action_required (boolean) - Implement Slack permalink generator using: -
https://YOURWORKSPACE.slack.com/archives/{channel_id} - Map Slack error
codes such as: - missing_scope - not_allowed_token_type -
no_permission - Generate human-readable explanation per error.

Definition of Done: - All Slack API failures return structured
diagnostic output. - Channel link always included when possible.

------------------------------------------------------------------------

## Issue 1.4 --- Redaction Module

Tasks: - Implement regex-based redaction for: - Slack tokens (xoxp-,
xoxb-, etc.) - OpenAI keys (sk-) - AWS keys - Emails - Long hex
strings - Return redaction count metadata. - Ensure sanitized text is
what leaves middleware.

Definition of Done: - Unit tests confirm redaction works. - Raw text
never returned externally.

------------------------------------------------------------------------

# PHASE 2 --- Conversation Inventory + Inbox Engine

------------------------------------------------------------------------

## Issue 2.1 --- Conversation Index

Tasks: - Implement conversations.list ingestion. - Store locally: -
conversation_id - name - type (im, mpim, public, private) -
is_slack_connect (detect via shared flag if present) -
last_activity_ts - Persist in SQLite.

Definition of Done: - Inventory stored locally. - Slack Connect channels
identifiable.

------------------------------------------------------------------------

## Issue 2.2 --- Message Cursor Engine

Tasks: - Maintain per-conversation cursor table: - last_fetched_ts -
Implement incremental fetch via conversations.history. - Deduplicate by
message ts. - Store sanitized messages only.

Definition of Done: - Repeated polling does not duplicate messages. -
Only new messages returned after first fetch.

------------------------------------------------------------------------

## Issue 2.3 --- Inbox Aggregation Endpoint

Implement:

GET /inbox

Returns: - Top conversations sorted by recent activity. - Last N
messages per conversation. - Structured error entries for failed
conversations.

Definition of Done: - Endpoint returns unified inbox view. - Slack
Connect failures appear as structured diagnostic objects, not crashes.

------------------------------------------------------------------------

## Issue 2.4 --- Conversation Endpoint

Implement:

GET /conversation?conversation_id=...

Returns: - Full message list since cursor. - Error diagnostics if
applicable.

Definition of Done: - Always returns either: - messages - structured
failure object

------------------------------------------------------------------------

# PHASE 3 --- OpenClaw Skill

------------------------------------------------------------------------

## Issue 3.1 --- Skill Scaffold

Tasks: - Create OpenClaw skill directory. - Implement actions: -
get_inbox() - get_conversation(conversation_id) - Attach X-API-Key
header to middleware calls.

Definition of Done: - Skill loads in OpenClaw without Slack token
present.

------------------------------------------------------------------------

## Issue 3.2 --- Configuration Guardrails

Tasks: - Fail if Slack token appears in OpenClaw config. - Require Slack
Reader base URL. - Require OpenClaw API key.

Definition of Done: - Slack token cannot accidentally enter agent
runtime.

------------------------------------------------------------------------

# PHASE 4 --- Slack Connect Resilience

------------------------------------------------------------------------

## Issue 4.1 --- Slack Connect Failure Handling

Tasks: - Detect shared channels via conversation metadata. - If history
fetch fails: - Store failure status - Include permalink - Include Slack
error code - Include explanation_human - Add retry cooldown (do not
hammer failing conversations). - Expose GET /diagnostics/slack-connect
endpoint listing: - conversation_id - name - error_code - explanation -
permalink

Definition of Done: - Slack Connect permission failures never crash
polling. - Diagnostics endpoint clearly shows affected channels.

------------------------------------------------------------------------

# PHASE 5 --- Final Integration (Requires Human Input)

Do not request human input until all previous phases pass local test
suite.

------------------------------------------------------------------------

## Required Human Inputs

When implementation is complete and tested, notify user via Discord
webhook.

Request the following:

1.  SLACK_USER_TOKEN (xoxp-...)
2.  Slack workspace domain (e.g., yourworkspace)
3.  Confirmation of granted scopes:
    -   channels:read
    -   channels:history
    -   groups:read
    -   groups:history
    -   im:read
    -   im:history
    -   mpim:read
    -   mpim:history
4.  OPENCLAW_API_KEY
5.  Discord webhook URL

------------------------------------------------------------------------

## Integration Validation Steps

After receiving credentials:

1.  Run auth.test.
2.  Enumerate conversations.
3.  Fetch history for:
    -   One public channel
    -   One private channel
    -   One DM
    -   One Slack Connect channel
    -   One Slack Connect DM
4.  Record results.
5.  Populate diagnostics endpoint.
6.  Confirm no crashes.

------------------------------------------------------------------------

# Completion Criteria

Project is complete when:

-   Inbox endpoint returns unified Slack feed.
-   Slack Connect failures are clearly reported with links and
    explanations.
-   No Slack write scopes used.
-   Slack token never enters OpenClaw runtime.
-   All endpoints return structured output, never raw Slack errors.

------------------------------------------------------------------------

Generated: 2026-03-02T18:05:12.330589 UTC
