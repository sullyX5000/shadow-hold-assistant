# Context Management Design

## implementation_reality_check

### implementation_reality
Built: n8n webhook workflow with one LLM call (Claude Sonnet via Anthropic Chat Model).

**Planned vs. Actual:**
- Planned: webhook triggers AI analysis
- Actual: webhook → AI Agent → Supabase → Google Calendar (matches plan, fully implemented)

The AI Agent receives the complete webhook JSON body as its user message via `JSON.stringify($('Webhook').item.json.body)`. The system prompt (final_prompt.txt) contains all scheduling rules, business hours tiers, and output format instructions.

### user_context_flow_analysis
Single LLM call. Information flow:
- **System context:** Static scheduling rules (business hours: 7AM-9PM good, 6-7AM/9-10PM warning, outside = bad), output format template, timezone handling instructions
- **User context:** Dynamic per-request data — subject, participants (name/city/timezone), proposed_utc, duration_minutes

No conversation history. No memory between requests. Each call is fully self-contained.

### quality_risk_connection
The quality risk (timezone conversion + business hours boundary judgment) lives entirely in this one LLM call. If the AI gets the timezone wrong or misclassifies a business hours boundary, there's no recovery step downstream. This makes the system prompt the single most critical quality lever.

---

## llm_call_inventory

### llm_call_analysis

**LLM Call #1: Shadow Hold Scheduling Analysis**
- **Core purpose:** Convert UTC proposed time to all participant local times, assess business hours viability, flag issues, suggest alternatives, draft scheduling email
- **Why LLM needed:** Multi-timezone reasoning with natural language output (draft email) can't be replaced with simple logic
- **Success criteria:** Correct local times for all participants, accurate business hours tier (good/warning/bad), actionable draft email
- **Failure modes:** Wrong timezone offset, incorrect DST handling, wrong business hours classification, vague/generic email draft
- **Quality risk connection:** 10/10 — this is the entire intelligence layer
- **Status:** ✅ Essential

### calls_to_remove_or_defer
None. Single LLM call is correctly scoped. No redundant calls identified.

---

## context_schemas

**LLM Call #1 — Input Schema:**

| Context Variable | Relevance to Quality | Required | Type | Source |
|---|---|---|---|---|
| `subject` | Meeting identification; used in email draft | Yes | string | User (webhook) |
| `participants` | Core data for timezone conversion | Yes | array of {name, city, timezone} | User (webhook) |
| `proposed_utc` | The time to convert and evaluate | Yes | ISO 8601 timestamp | User (webhook) |
| `duration_minutes` | Affects scheduling email copy | No | integer | User (webhook) |
| Business hours rules | Defines good/warning/bad tiers | Yes | Static (system prompt) | System |
| Output format template | Structures response for downstream parsing | Yes | Static (system prompt) | System |

**Context waste identified:** None. The webhook payload is small and all fields are used. The system prompt (final_prompt.txt) is the only potential bloat — it includes full output format instructions plus business hours rules — but given the single-call architecture, this is appropriate. No redundant context passing.

**Data flow gaps:** None critical. One future improvement: pass `meeting_id` from Supabase insert back into subsequent nodes for proper relational linking between meetings and time_options tables.
