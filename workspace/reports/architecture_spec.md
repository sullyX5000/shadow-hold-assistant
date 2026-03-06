# Architecture Specification

## agent_classification

**System has 1 Agent and 3 Services.**

### Agent: Scheduling Intelligence Agent
Handles reasoning, judgment, and natural language output. Makes decisions about business hours viability, drafts emails, suggests alternatives when no good overlap exists.

- Input: Structured JSON (subject, participants, proposed_utc, duration)
- Output: Structured text (SHADOW HOLD ANALYSIS format)
- Decision points: business hours tier, weekend flag, alternative time suggestions
- Technology: Claude Sonnet via Anthropic API (n8n AI Agent node)

### Service 1: Webhook Receiver
Receives and validates incoming scheduling requests. Stateless.
- Technology: n8n Webhook node

### Service 2: Calendar Service
Creates shadow hold events on Google Calendar. Stateless per call.
- Technology: n8n Google Calendar node (OAuth2)

### Service 3: Data Persistence Service
Stores meeting state, time options, and execution logs. Stateful.
- Technology: Supabase (PostgreSQL) — tables: meetings, time_options, workflow_logs

---

## architecture_control_flow

```
Webhook (trigger)
    │
    ▼
AI Agent (Scheduling Intelligence)
    │  Input: JSON payload from webhook body
    │  System: Business hours rules + output format
    │  Output: SHADOW HOLD ANALYSIS text
    │
    ▼
Supabase: meetings table
    │  Stores: subject, status="pending", proposed_utc, duration, ai_response
    │
    ▼
Code node (JavaScript)
    │  Parses AI output → structured time_options per participant
    │
    ▼
Google Calendar
    │  Creates: shadow hold event at proposed_utc
    │
    ▼
Supabase: time_options table
    │  Stores: per-participant local times from AI output
    │
    ▼
Supabase: workflow_logs table
    │  Stores: execution metadata for observability
    │
    ▼
[EA Review — future approval gate]
```

---

## architecture_boundaries

| Boundary | What crosses it | Direction |
|---|---|---|
| Webhook → AI Agent | Full JSON payload | In |
| AI Agent → Supabase | ai_response (text), meeting metadata | Out |
| AI Agent → Calendar | Event details (time, title) | Out |
| AI Agent → time_options | Parsed participant times | Out |

**Handoff contracts:**
- Webhook body must include: subject (string), participants (array), proposed_utc (ISO 8601), duration_minutes (integer)
- AI Agent output must follow SHADOW HOLD ANALYSIS format for downstream Code node to parse correctly

---

## service_composition

**Data flow summary:**
1. Single source of truth for each request: the webhook payload
2. AI Agent enriches the request → produces the scheduling analysis
3. Supabase persists state for traceability and future EA review UI
4. Google Calendar is the delivery surface (the actual shadow hold)

**System-wide quality risk point:** The AI Agent node is the single point where quality risk lives. All other nodes are deterministic. If the AI Agent fails (wrong timezone, wrong business hours tier), the downstream nodes execute correctly but with wrong data. This is why the system prompt (final_prompt.txt) is the primary quality lever.
