# User Experience Specification

## user_context_schema

The EA's mental model when scheduling a meeting:
- Who needs to be there (participants + their locations)
- When is proposed (a specific time)
- How long (duration)
- What's it for (subject/purpose)

**UserContext fields:**

| Field | Description | Type | Provided by |
|---|---|---|---|
| `subject` | Meeting name/purpose | string | EA |
| `participants` | List of attendees with name, city, timezone | array | EA |
| `proposed_utc` | Proposed meeting time in UTC | ISO 8601 | EA |
| `duration_minutes` | Meeting length | integer | EA |

---

## agent_context_mapping

**UserContext → AgentContext:**

| UserContext Field | AgentContext Role | Used for |
|---|---|---|
| `subject` | Meeting identifier | Email draft subject line |
| `participants[].timezone` | Conversion targets | Timezone math |
| `participants[].name` | Participant labels | Local time display + email |
| `proposed_utc` | Source time | All timezone conversions |
| `duration_minutes` | Schedule block size | Calendar event + email copy |

**System-injected AgentContext (not from user):**
- Business hours rules (7AM-9PM good, 6-7AM/9-10PM warning, outside = bad)
- Output format template
- Scheduling email tone and structure guidance

---

## context_ownership

| Field | Owner | EA co-authors? |
|---|---|---|
| subject | EA | Yes — EA provides this |
| participants | EA | Yes — EA provides this |
| proposed_utc | EA (or executive) | Yes — EA enters the proposed time |
| Business hours rules | System | No — hardcoded in system prompt |
| Draft email | Agent | Yes — EA reviews and edits before sending |
| Meeting status | System | Partially — EA approves → status changes |

---

## interaction_flow

**Step 1 — EA triggers the workflow**
EA submits a form (or future Lovable UI) with: meeting subject, participants (name + timezone), proposed UTC time, duration.

**Step 2 — Agent analyzes (autonomous)**
Claude converts timezones, checks business hours, generates SHADOW HOLD ANALYSIS output with status (good/warning/bad) and draft email. ~10 seconds.

**Step 3 — EA reviews output (human judgment)**
EA reads the analysis. Sees local times for all participants. Sees status flag. Reviews draft email. Makes a judgment: proceed, adjust time, or cancel.

**Step 4 — EA approves and sends (human action)**
EA copies the draft email, makes edits if needed, and sends to participants. Shadow hold is already on the calendar from Step 2.

**Step 5 — EA updates status (future automation)**
When participants confirm, EA (or future automation) updates the Supabase meeting status from "pending" → "confirmed". Shadow hold becomes real calendar event.

---

## design_principles

1. **EA is always in the loop before anything goes external.** The agent proposes; the EA approves.
2. **The AI does the research; the EA brings the judgment.** Timezone math = AI. "Is this the right time for this relationship?" = EA.
3. **Meet the EA where she works.** Calendar and email are her tools — the output lands there, not in a new app she has to learn.
4. **Favor the guest.** When no good time exists, the suggested alternatives should optimize for the external participant's working hours first.
