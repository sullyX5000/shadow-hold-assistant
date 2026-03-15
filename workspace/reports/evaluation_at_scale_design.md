# Evaluation at Scale Specification
## Shadow Hold Assistant

---

## evaluation_reality_check

### Quality Risk - Current Understanding

**Primary Risk: Timezone Conversion Errors**
The system must correctly calculate available time slots when stakeholders span non-standard timezone offsets (India UTC+5:30, Nepal UTC+5:45, Newfoundland UTC-3:30). This risk has been validated and largely addressed in the MVP, but the evaluation dataset revealed an important adjacent failure: even when timezone math is correct, the system lacked business hours judgment — TC8 (4-party global meeting) passed technically but failed practically because it suggested a 4:00 AM NYC slot without flagging it as unreasonable.

**Evolution since earlier workflows:**
The risk understanding has sharpened from "will the system get the timezone math right?" to "will the system produce practically useful suggestions?". Timezone arithmetic is now a solved problem (7/8 pass rate, all standard and non-standard offsets handled correctly). The new frontier is semantic quality: does the output demonstrate scheduling judgment, not just computational accuracy?

### Current Evaluation Approach

**Methods used today:**
- 8 manually-designed test cases run end-to-end through the Lovable frontend → n8n webhook → Claude AI Agent → Supabase
- Manual inspection of AI Agent output text (SHADOW HOLD ANALYSIS format)
- CSV tracking of pass/fail per test case with notes

**Pain points:**
- Completely manual — each test case requires human interpretation of free-text AI output
- No automated detection of "4am suggests" — a human must read and judge whether the suggested time is reasonable
- No way to distinguish "technically correct but practically useless" from "truly good" without reading the full response
- Running all 8 tests takes 30-40 minutes manually
- No ability to run tests without active Cloudflare tunnel (local Docker dependency)

**Observed failure patterns:**
- TC8 failure: AI accurately calculated all 4 timezones but didn't apply business hours reasoning to flag the impractical result
- The failure mapped to a context delivery issue: the system prompt didn't give the AI Agent explicit criteria for "reasonable hours" or instructions to warn when overlap doesn't exist
- Fix applied: added 7am-9pm business hours constraint and flagging logic to the system prompt

**Desired automation:**
- A way to evaluate AI Agent outputs without running the full pipeline each time
- LLM-as-judge to automate the "is this output practically useful?" question
- Ability to run evaluation cases against the AI Agent in isolation (not through the full webhook flow)

---

## output_property_rubric

### Rubric Overview

The Scheduling Intelligence Agent is the single point where quality risk lives. All upstream nodes (webhook, input parsing) are deterministic. All downstream nodes (Supabase write, Calendar create) execute correctly but with wrong data if the AI Agent fails. This means the rubric must evaluate the AI Agent's SHADOW HOLD ANALYSIS output across dimensions that directly map to the ways context management can break.

**Connection to quality risk:** The priority risk is timezone conversion + business hours judgment. A rubric that only checks timezone math would miss the TC8 class of failure. The rubric must also capture whether the agent exercised scheduling judgment.

### Agent Rubrics

**Agent: Scheduling Intelligence Agent**

Input: Structured JSON with subject, participants (with timezone), proposed_utc (ISO 8601), duration_minutes
Output: SHADOW HOLD ANALYSIS text block with per-participant local times and a scheduling judgment

---

#### Dimension 1: Timezone Conversion Accuracy
**What it measures:** Whether each participant's local time in the output is arithmetically correct given their IANA timezone and the proposed_utc timestamp.

- **Score 5 (Reliable):** All participant local times are exactly correct. Non-standard offsets (+5:30, +5:45, -3:30) are preserved without rounding. DST-aware conversions are correct for the meeting date. IANA names used, not ambiguous abbreviations.
- **Score 3 (Borderline):** Standard timezone participants are correct but one non-standard offset is off by 30 minutes. Ambiguous abbreviation used but correct offset applied. Times look plausible but manual verification reveals a small discrepancy.
- **Score 1 (Severe):** One or more participants show clearly wrong local times (off by an hour or more, wrong date, or visibly impossible offset). Meeting scheduled for 3:00 AM without acknowledgment. Timezone abbreviation conflict produces wrong city's offset.

**Context pillar mapping:** Low scores indicate acquisition failure (timezone data not correctly incorporated into context) or shaping failure (conversion logic applied incorrectly during reasoning).

---

#### Dimension 2: Business Hours Judgment
**What it measures:** Whether the agent correctly identifies and communicates whether the proposed time falls within reasonable working hours (7 AM - 9 PM) for each participant, and whether it flags or warns when a participant is outside those bounds.

- **Score 5 (Reliable):** Output explicitly evaluates business hours for all participants. Times outside 7am-9pm are flagged with clear language ("OUTSIDE BUSINESS HOURS" or equivalent). When no overlap exists, the agent says so and suggests alternatives or explains the tradeoff. Output helps the EA make an informed decision.
- **Score 3 (Borderline):** Business hours check present but incomplete — flags some participants outside bounds but misses others, or uses vague language ("early morning for NYC") without explicit judgment. Alternatives mentioned but not specific enough to act on.
- **Score 1 (Severe):** Output presents mathematically correct times but makes no mention of whether 4:00 AM NYC is problematic. No flagging, no warning, no acknowledgment that the proposal requires human judgment. EA would need to independently realize the time is unreasonable.

**Context pillar mapping:** Low scores indicate shaping failure (system prompt didn't provide business hours reasoning criteria) or delivery failure (agent has the right information but doesn't surface the relevant judgment in output).

---

#### Dimension 3: Output Parsability and Completeness
**What it measures:** Whether the SHADOW HOLD ANALYSIS output follows the expected format so that the downstream Code node (JavaScript parser) can correctly extract per-participant times, and whether all required fields are present.

- **Score 5 (Reliable):** Output matches expected SHADOW HOLD ANALYSIS format exactly. All participants listed with their local time, timezone label, and status indicator. Subject, duration, and proposed UTC time present. No extraneous text that would break downstream parsing.
- **Score 3 (Borderline):** Output is mostly correct format but has minor variations — extra line breaks, slightly different field labels, one participant listed differently. Downstream parser can extract most data but may fail on edge cases.
- **Score 1 (Severe):** Output is free-text prose without structure, or uses a completely different format. Missing multiple required participants. Downstream Code node cannot parse and workflow fails silently (the TC8-class "technical pass / practical fail" scenario).

**Context pillar mapping:** Low scores indicate delivery failure (agent produces correct reasoning but in wrong format for downstream consumption) or acquisition failure (system prompt format instructions not loaded correctly).

---

### Failure Indicators

| Score Range | Likely Pillar | What to Check |
|---|---|---|
| Dim 1 ≤ 2 | Acquisition or Shaping | Is the IANA timezone in the input JSON? Is the system prompt using a proper timezone library or manual math? |
| Dim 2 ≤ 2 | Shaping or Delivery | Does the system prompt include business hours constraints? Does the output format template require a judgment section? |
| Dim 3 ≤ 2 | Delivery | Is the output format template in the system prompt correct? Did a model update change default output style? |
| Dim 1 = 3 AND Dim 2 = 1 | Shaping (compound) | Timezone math works but no judgment criteria in system prompt — classic TC8 failure mode |

---

## anchor_examples

### Level Definitions

- **Level 5 (Reliable):** Output that a professional EA would trust to act on without further verification
- **Level 3 (Borderline):** Output that requires the EA to do additional checking before acting — something feels off but isn't obviously wrong
- **Level 1 (Severe):** Output that would cause a real scheduling failure if acted on — a meeting at the wrong time, a missed warning, or a broken pipeline

---

### Example Matrix

#### Example 1: Gold Standard (5-5-5)

**Name:** 3-Party Meeting, All Standard Zones, Clear Overlap
**Input Context:**
```json
{
  "subject": "Q2 Strategy Sync",
  "participants": ["alice@corp.com (America/New_York)", "bob@corp.com (Europe/London)", "carol@corp.com (Asia/Singapore)"],
  "proposed_utc": "2026-04-15T14:00:00Z",
  "duration_minutes": 60
}
```

**Agent Output:**
```
SHADOW HOLD ANALYSIS

Subject: Q2 Strategy Sync
Duration: 60 minutes
Proposed UTC: 2026-04-15 14:00 UTC

PARTICIPANT LOCAL TIMES:
- Alice (America/New_York): 10:00 AM EDT ✓ Business hours
- Bob (Europe/London): 3:00 PM BST ✓ Business hours
- Carol (Asia/Singapore): 10:00 PM SGT ✓ Within acceptable hours

STATUS: REASONABLE — all participants within 7 AM - 10 PM local time
SHADOW HOLD: Creating at 10:00 AM EDT on April 15, 2026
```

**Scores:**
- Dimension 1: 5 — All conversions correct (UTC+0 BST, UTC+8 SGT, UTC-4 EDT all applied correctly)
- Dimension 2: 5 — Explicitly evaluates business hours for all 3 participants, flags Singapore as "within acceptable hours" (late but not unreasonable), clear status line
- Dimension 3: 5 — Perfect SHADOW HOLD ANALYSIS format, all fields present, parseable by downstream Code node

**Which pillar succeeded:** All pillars functioning — acquisition got the timezone data, shaping applied correct reasoning, delivery produced usable structured output

---

#### Example 2: Severe Failure — Wrong Timezone Math (1-3-3)

**Name:** Mumbai Offset Rounding Error
**Input Context:**
```json
{
  "subject": "Budget Review",
  "participants": ["exec@corp.com (America/New_York)", "cfo@corp.com (Asia/Kolkata)"],
  "proposed_utc": "2026-03-10T09:00:00Z",
  "duration_minutes": 60
}
```

**Agent Output:**
```
SHADOW HOLD ANALYSIS

Subject: Budget Review
Duration: 60 minutes

PARTICIPANT LOCAL TIMES:
- exec@corp.com: 4:00 AM EST
- cfo@corp.com: 2:00 PM IST

STATUS: WARNING — New York participant outside business hours
```

**Scores:**
- Dimension 1: 1 — Mumbai (Asia/Kolkata) should be 2:30 PM IST at UTC+5:30, not 2:00 PM IST. The +5:30 offset was rounded to +5:00, producing a 30-minute error.
- Dimension 2: 3 — Does flag that NYC is outside business hours, but Mumbai time is wrong which makes the judgment unreliable
- Dimension 3: 3 — Format mostly correct but missing participant names (just email addresses) and proposed_utc field

**Which pillar failed:** Acquisition failure — the +5:30 offset was not correctly applied. The IANA timezone name was in the input but the conversion used rounded offset math instead of library-based calculation.

---

#### Example 3: Severe Failure — No Business Hours Judgment (5-1-5)

**Name:** TC8 Class Failure — 4-Party Global Meeting
**Input Context:**
```json
{
  "subject": "Global Leadership Summit",
  "participants": ["nyc@corp.com (America/New_York)", "lon@corp.com (Europe/London)", "mum@corp.com (Asia/Kolkata)", "tok@corp.com (Asia/Tokyo)"],
  "proposed_utc": "2026-04-01T09:00:00Z",
  "duration_minutes": 60
}
```

**Agent Output:**
```
SHADOW HOLD ANALYSIS

Subject: Global Leadership Summit
Duration: 60 minutes
Proposed UTC: 2026-04-01 09:00 UTC

PARTICIPANT LOCAL TIMES:
- nyc@corp.com: 5:00 AM EDT (America/New_York, UTC-4)
- lon@corp.com: 10:00 AM BST (Europe/London, UTC+1)
- mum@corp.com: 2:30 PM IST (Asia/Kolkata, UTC+5:30)
- tok@corp.com: 6:00 PM JST (Asia/Tokyo, UTC+9)

SHADOW HOLD: Creating at 5:00 AM EDT on April 1, 2026
```

**Scores:**
- Dimension 1: 5 — All 4 timezone conversions are exactly correct, including Mumbai +5:30
- Dimension 2: 1 — No mention that 5:00 AM EDT is outside business hours. No warning. No flag. No suggestion of alternatives. EA would receive this output and not know the proposal is impractical.
- Dimension 3: 5 — Perfect format, all fields present, fully parseable

**Which pillar failed:** Shaping failure — the business hours evaluation criteria were not in the system prompt. The agent had the correct input data (acquisition OK) and produced correct format (delivery OK) but did not apply the required reasoning during shaping.

---

#### Example 4: Borderline — Vague Judgment (4-3-4)

**Name:** Late Evening Warning — Unclear Language
**Input Context:**
```json
{
  "subject": "Product Roadmap",
  "participants": ["sf@corp.com (America/Los_Angeles)", "sg@corp.com (Asia/Singapore)"],
  "proposed_utc": "2026-05-20T23:00:00Z",
  "duration_minutes": 30
}
```

**Agent Output:**
```
SHADOW HOLD ANALYSIS

Subject: Product Roadmap
Duration: 30 minutes

PARTICIPANT TIMES:
- sf@corp.com: 4:00 PM PDT
- sg@corp.com: 7:00 AM SGT (next day)

NOTE: Singapore participant is in early morning hours.
```

**Scores:**
- Dimension 1: 4 — Times are correct (UTC-7 PDT, UTC+8 SGT), but "next day" date transition not explicitly shown in ISO format
- Dimension 2: 3 — Acknowledges Singapore is "early morning" but doesn't explicitly say whether this is within acceptable hours, doesn't flag, doesn't recommend alternatives. EA must decide what to do with vague note.
- Dimension 3: 4 — Format mostly correct but "PARTICIPANT TIMES" vs expected "PARTICIPANT LOCAL TIMES" label and missing proposed_utc field could cause downstream parsing issues

**Which pillar failed:** Delivery borderline — information is present but not surfaced with enough clarity to guide EA action without additional interpretation.

---

#### Example 5: Mixed Score — Format Failure with Good Judgment (2-5-1)

**Name:** Correct Judgment, Broken Format
**Input Context:**
```json
{
  "subject": "Vendor Negotiation",
  "participants": ["ceo@corp.com (America/Chicago)", "vendor@company.com (Europe/Berlin)"],
  "proposed_utc": "2026-06-10T15:00:00Z",
  "duration_minutes": 45
}
```

**Agent Output:**
```
For the Vendor Negotiation meeting, Chicago (CDT) would be at 10:00 AM and Berlin (CEST) would be at 5:00 PM.

Both times are within business hours — this is a good time to propose. I would suggest creating a shadow hold for 10:00 AM CDT on June 10, 2026. The proposed 45-minute duration fits comfortably within both participants' working hours.
```

**Scores:**
- Dimension 1: 2 — Chicago CDT is UTC-5, so 15:00 UTC = 10:00 AM CDT (correct). Berlin CEST is UTC+2, so 15:00 UTC = 17:00 CEST = 5:00 PM (correct). But no timezone metadata shown — can't verify without manual calculation.
- Dimension 2: 5 — Clear judgment that both times are within business hours, explicit conclusion that this is a good proposal time
- Dimension 3: 1 — Prose format cannot be parsed by the downstream Code node. No SHADOW HOLD ANALYSIS header, no structured fields, no parseable output. Pipeline will fail silently.

**Which pillar failed:** Delivery failure — good reasoning and correct times, but output format is completely wrong for downstream consumption. Classic case where LLM "helpfully" reformatted the response.

---

### Reasoning Notes

The most important pattern from this example matrix is the **independence of dimensions**. A score of 5 on Dimension 1 does not predict the score on Dimension 2 (TC8 is the proof). Dimension 3 failures can mask otherwise good performance. This means evaluation must score all three dimensions independently — a high average score can hide a critical failure.

---

## llm_as_judge_prompt

The LLM-as-judge prompt has been saved to workspace/prompts/llm_as_judge_prompt.txt

This judge prompt covers the Scheduling Intelligence Agent — the single agent in the Shadow Hold Assistant architecture. The three rubric dimensions (Timezone Conversion Accuracy, Business Hours Judgment, Output Parsability) were designed to surface the specific failure modes identified in the evaluation dataset, with anchor examples drawn from real test cases (including the TC8 failure that motivated this workflow).
