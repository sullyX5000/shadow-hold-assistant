# Implementation Design

## learning_since_last_interaction

### user_research_insights
Wife's interview (primary user: EA at Fortune 500) confirmed the core product direction and helped revise the concept. Key findings:
- The EA always favors the prospective client/guest when scheduling conflicts arise
- Global timezone complexity is the hardest part of her current workflow — she handles it manually
- Human approval is non-negotiable: she wants to review and approve before anything goes to external parties
- She currently relies on notes, email records, and instinct — no automated tooling
- Value proposition is less about "doing it for her" and more about providing best-in-class scheduling standards with AI assistance

### evaluation_testing_results
8 test cases executed (TC1-TC8). Results:
- Technical pass rate: 8/8 (100%)
- Practical pass rate: 7/8 (87.5%)
- TC8 failed (practical): 4-party global meeting — NYC participant would be at 4:00 AM, which the system did not flag as impractical

Key patterns from testing:
- Timezone conversion math was solid across all cases including non-standard offsets (+5:30, +5:45, +3:30)
- IANA timezone names prevented IST/CST ambiguity completely
- DST transitions handled correctly including combined non-standard + DST (Newfoundland)
- The gap was not conversion accuracy — it was business hours boundary judgment

This sharpened the quality risk definition: the risk is not just "will it convert correctly?" but "will it flag genuinely unreasonable meeting times?"

### prompt_experimentation_findings
Iterated from prompt_v1.txt to final_prompt.txt. Key changes:
- Added structured output format (SHADOW HOLD ANALYSIS → PROPOSED MEETING → LOCAL TIMES → STATUS → CONSIDERATIONS)
- Added tiered business hours validation: good (7AM-9PM), warning (6-7AM or 9-10PM), bad (outside that range)
- Improved parsing so participant timezone data maps cleanly to the right output sections
- Added explicit instruction to suggest alternative times when no good overlap exists

### implementation_progress_status
Built end-to-end n8n workflow: Webhook → AI Agent (Claude Sonnet) → Supabase (meetings table) → Code in JavaScript → Google Calendar → Logging rows

What worked well:
- AI Agent node with Anthropic Chat Model processed scheduling requests correctly from first successful run
- System prompt + JSON webhook data approach produced clean structured output
- Supabase multi-table logging (meetings, time_options, workflow_logs) came together once credential issue was resolved

What was harder than expected:
- Supabase field mapping after inserting the AI Agent node — `$json` references broke because the data source changed
- Service role key vs anon key distinction for RLS bypass was not obvious
- n8n expression syntax (`$('Webhook').item.json.body.subject`) required discovering the `.body.` nesting through inspection

When each piece finally connected, the overall system logic became clear and intuitive.

### updated_quality_risk_focus
Priority quality risk: **timezone conversion accuracy + business hours boundary judgment**

No change in priority. TC8 finding refined our understanding: the risk is not arithmetic error in timezone math (that was solid) but whether the system correctly identifies and flags times that are technically valid but practically unreasonable for human participants.

The business hours tiering (good/warning/bad) added in final_prompt.txt directly addresses this. The quality risk remains the right focus heading into production testing.

---

## delivery_context_design

### workflow_analysis
**Pain points in EA's current workflow:**
- Manual timezone lookups across multiple participants (context-switching)
- Copy-pasting availability into emails manually
- No automated tracking of shadow holds or tentative blocks
- No system memory — relies on personal notes and email search

**Flow points (comfortable, efficient areas):**
- She knows her executive's preferences intuitively
- She has strong instinct for which times "feel right" for different participant types
- She manages the final approval gate confidently — she just wants the research done for her

### delivery_mechanism
n8n workflow triggered via webhook, integrated with:
- Google Calendar (for shadow hold event creation)
- Supabase (for tracking meeting state and time options)
- Claude Sonnet via Anthropic API (for timezone analysis and email drafting)

EA interacts via a simple web form (future: Lovable frontend) that sends the webhook payload.

### interaction_model
Human-in-the-loop: EA reviews Claude's scheduling analysis and draft email before anything is sent externally. The agent proposes; the EA disposes.

### agency_vs_autonomy
| Task | Mode | Rationale |
|------|------|-----------|
| Timezone conversion | Autonomous | Pure math — no judgment needed |
| Business hours check | Autonomous | Rules-based, low stakes |
| Draft scheduling email | Autonomous | EA reviews before sending |
| Final send to participants | Human agency | Non-negotiable — EA must approve |
| Calendar block creation (shadow hold) | Autonomous | Tentative blocks are low-risk, reversible |

### user_touchpoints
- **Input:** EA submits meeting subject, participants, proposed UTC time, duration
- **Intermediate:** EA reviews Claude's analysis and draft email
- **Output:** Shadow hold on calendar + tracking row in Supabase + approved email ready to send

---

## backend_design

### technical_approach
n8n workflow orchestration with Claude as the intelligence layer. Non-technical EA never sees the backend — she interacts only with the input form and output email draft.

### data_flow_architecture
```
Webhook (JSON payload)
  → AI Agent node (Claude Sonnet)
      System: final_prompt.txt (scheduling rules, business hours tiers)
      User: JSON.stringify(webhook body)
  → Supabase: meetings table (subject, status, proposed_utc, duration, ai_response)
  → Code node: parse AI output into structured time options
  → Google Calendar: create shadow hold event
  → Supabase: time_options table (per-participant local times)
  → Supabase: workflow_logs table (execution tracking)
```

### integration_points
- Anthropic API: Claude Sonnet for scheduling intelligence
- Supabase: PostgreSQL with RLS (service role key required for write access)
- Google Calendar API: OAuth2 for event creation

### decision_points
- Business hours tier (good/warning/bad) determines STATUS field in output
- If no good overlap exists → Claude suggests alternatives
- Weekend detection → warning flag

### human_judgment_points
EA reviews the full SHADOW HOLD ANALYSIS output before approving email send. The n8n workflow pauses at the approval gate (future implementation — current MVP auto-continues).

---

## mvp_scope_definition

### first_implementation_target
Validate that Claude can reliably convert timezones, flag business hours issues, and draft a usable scheduling email — with the EA reviewing before anything goes external.

### core_path_only
Webhook with participant + time data → Claude analysis → Supabase log → Google Calendar shadow hold → EA review

### hardcoded_elements
- Business hours window: 7AM-9PM (hardcoded in system prompt)
- Status tiers: good/warning/bad (hardcoded logic in prompt)
- Meeting status: always starts as "pending"

### definition_of_done
- [x] Webhook receives scheduling request
- [x] Claude correctly converts timezones for all participants
- [x] Business hours flags appear in output
- [x] Meeting logged to Supabase meetings table
- [x] Google Calendar shadow hold created
- [ ] EA can approve/reject via simple interface
- [ ] Approved email drafted and ready to send

### deferred_features
- Email send automation (v2) — EA manually sends for now
- Multi-turn conversation memory (v2)
- Frontend approval UI (v2 — Lovable integration)
- TC9-TC16 edge cases (v2 testing)

---

## implementation_platform_selection

### platform
n8n (self-hosted via Docker)

### platform_rationale
- Visual workflow builder matches Brian's design thinking background
- Strong native integrations (Google Calendar, Supabase, Anthropic)
- Self-hosted = no vendor lock-in, full data control
- Webhook trigger fits the EA's push-based workflow

### platform_tradeoffs
| Pro | Con |
|-----|-----|
| Visual, no-code friendly | Requires Docker/self-hosting knowledge |
| Strong integration library | Local deployment limits accessibility |
| Easy to iterate workflows | Cloudflare tunnel needed for external webhooks |

### alternative_platforms_considered
- Make.com: easier hosting but less control
- LangGraph: more powerful but too code-heavy for current stage
- Direct API: too much custom infrastructure

### implementation_approach

#### technical_approach_summary
n8n orchestrates the workflow; Claude handles intelligence; Supabase provides state persistence; Google Calendar is the delivery surface for shadow holds. Each component does one thing well.

#### implementation_timeline_considerations
MVP is built and running. Next phase: EA approval interface and email sending automation.
