# Evaluation Design Report
## Shadow Hold Assistant

---

## Prompt Testing Experience

### Testing Scope
- Tested the full end-to-end pipeline: Lovable form → n8n webhook → Supabase → Google Calendar
- 7 distinct AI playground experiments across multiple tools (Lovable, Claude, n8n)
- Tested form generation, workflow orchestration, query generation, debugging, dashboard creation, meeting info extraction, and iterative prompt refinement
- Multiple iterations per experiment (3-4 refinement cycles on prompts)

### Quality Observations

**What worked well:**
- AI understood the shadow hold concept and scheduling domain quickly - the basics were grasped easily
- Supabase relational queries generated correctly on first attempt
- n8n workflow blueprint from natural language matched actual implementation closely
- Dashboard component worked on first render with correct data fetching
- Structured prompts with explicit output formats produced consistent, machine-parseable results

**What struggled:**
- Lovable AI assumed email (mailto) was the data transmission method instead of webhook POST - required explicit specification of business logic
- Past dates could be selected in the form without validation - temporal constraints weren't enforced
- Timezone handling was not addressed in initial prompts, causing naive datetime handling
- AI occasionally over-engineered solutions or added unrequested features

**Consistency issues:**
- Form posted whatever was selected without validation (including past dates)
- AI-generated code made reasonable but incorrect assumptions about data flow (mailto vs webhook)
- No consistency issues noticed in the data itself once correctly posted - times and subjects arrived accurately

### Edge Cases Discovered
1. **Past date selection** - Form allows booking meetings on dates that have already passed
2. **Calendar status ambiguity** - OOO blocks should block availability, tentatives should be last-resort options, all-day "special day" events could receive a bypass
3. **Cross-timezone coordination** - Stakeholders across global timezones (EST, GST/Dubai, IST/India) create complex availability windows
4. **Non-standard timezone offsets** - India (UTC+5:30), Nepal (UTC+5:45), Newfoundland (UTC-3:30) have 30 or 15-minute offsets that are easy to miscalculate
5. **Google Calendar free/busy accuracy** - Untested whether API returns fully accurate free/busy data for all event types

---

## Quality Dimensions

Based on testing experience, the following quality dimensions matter most for the Shadow Hold Assistant:

### 1. Temporal Accuracy
- Only forward-looking dates should be selectable
- Time slot suggestions must account for meeting duration, not just start time availability
- Timezone conversions must handle non-standard offsets (30-min, 15-min increments)

### 2. Calendar Status Interpretation
- OOO blocks = unavailable (hard block)
- Tentative events = last-resort availability (soft block)
- All-day special events (birthdays, holidays) = configurable bypass
- Shadow holds from this system = should not block other proposals
- Confirmed meetings = hard block

### 3. Cross-Timezone Correctness
- Must accurately convert between all stakeholder timezones simultaneously
- Must flag "unreasonable hours" for each stakeholder (not just the EA's timezone)
- Must handle non-standard offsets (India +5:30, Nepal +5:45, Newfoundland -3:30)
- Current pain: EA manually shifts calendar view per timezone and records on paper - high error potential

### 4. Data Flow Integrity
- Form submissions must reach the webhook (not get intercepted by mailto or other browser behaviors)
- All time options must persist correctly in Supabase with proper timezone metadata
- Calendar events must be created with correct timezone-aware datetimes

### 5. Input Validation
- Past dates must be rejected
- Duration must fit within available time blocks
- Stakeholder email addresses must be valid format
- Required fields must be enforced before submission

---

## Quality Risk Hypotheses

*Ranked by participant priority (1 = highest concern)*

### Input Variability Risks

**Risk 1 (Priority #1): Timezone Conversion Errors**
The system miscalculates available time slots when stakeholders span non-standard timezone offsets. For example, suggesting 2:00pm Dubai / 5:00am EST when the actual offset is 9 hours, or miscalculating India's +5:30 offset as +5:00 or +6:00 - producing a slot that's 30 minutes off.
- **Connection to quality dimension:** Cross-Timezone Correctness
- **Potential impact:** EA's wife currently does timezone conversion manually and already makes mistakes. If the agent gets it wrong too, trust is lost immediately and adoption fails. Missed 30-minute offsets in India/Nepal timezones are particularly insidious because they're close enough to seem right.
- **Symptoms:** Meeting invites arrive for wrong times; stakeholders in non-standard timezone countries consistently report conflicts

**Risk 7 (Priority #7): Stakeholder Email Format Variations**
Stakeholder emails entered with extra spaces, semicolons instead of commas, display names mixed in ("John Smith <john@company.com>"), or distribution lists instead of individuals.
- **Connection to quality dimension:** Input Validation
- **Potential impact:** Calendar invites go to wrong people or nobody; system fails silently on malformed input
- **Symptoms:** Missing attendees on calendar events; bounced invitations

### Output Quality Risks

**Risk 4 (Priority #4): Multi-Stakeholder Availability Window Collapse**
When 3+ stakeholders across different timezones are involved, the system finds zero overlapping "reasonable hours" windows - or suggests times that are technically available but at 6am for one party and 11pm for another.
- **Connection to quality dimension:** Cross-Timezone Correctness, Temporal Accuracy
- **Potential impact:** The core value proposition breaks down for the most complex (and most valuable) scheduling scenarios. Too-strict filtering returns no options; too-loose filtering suggests impractical times.
- **Symptoms:** "No available times" for meetings that the EA knows should be possible; stakeholders declining because suggested times are unreasonable for their timezone

**Risk 6 (Priority #6): Webhook/Data Flow Silent Failure**
The form submits successfully (user sees a success toast) but the n8n webhook doesn't process it - no calendar events are created, no Supabase record exists. The EA thinks it worked but nothing happened.
- **Connection to quality dimension:** Data Flow Integrity
- **Potential impact:** Already experienced with the mailto bug. Silent failures mean the EA moves on thinking scheduling is handled, leading to missed meetings and lost trust.
- **Symptoms:** Success toast shown but no shadow holds appear on calendar; Supabase has no record of submission

### Context Sensitivity Risks

**Risk 2 (Priority #5): Calendar Status Misinterpretation**
The system treats a tentative event as "available" when it should be last-resort, or blocks on an all-day birthday event when it should bypass it. Different calendar providers (Google vs Outlook) may represent these statuses differently.
- **Connection to quality dimension:** Calendar Status Interpretation
- **Potential impact:** Wrong interpretation leads to suggesting times that are actually busy (embarrassing double-booking) or hiding valid options unnecessarily (frustrating).
- **Symptoms:** EA gets complaints about meetings booked over tentative commitments; system consistently misses valid time slots that the EA can see manually

### Boundary/Scope Risks

**Risk 3 (Priority #2): Past Date Acceptance**
The form allows selection of dates and times that have already passed, creating shadow holds for impossible meetings. Could also allow booking during weekends or holidays without flagging them.
- **Connection to quality dimension:** Input Validation, Temporal Accuracy
- **Potential impact:** Creates junk data in Supabase and phantom calendar events that clutter the executive's calendar. Wastes EA time cleaning up invalid proposals.
- **Symptoms:** Calendar shows shadow holds for yesterday; Supabase contains meeting records with past dates

### Consistency Risks

**Risk 5 (Priority #3): Shadow Hold Collision with Other Shadow Holds**
If the EA creates multiple meeting proposals for the same executive in quick succession, the shadow holds from Proposal A might block time that Proposal B needs - even though neither is confirmed yet.
- **Connection to quality dimension:** Calendar Status Interpretation
- **Potential impact:** Shadow holds are tentative by design. If they block each other, the EA can't explore multiple scheduling options simultaneously - defeating the purpose of the tool.
- **Symptoms:** Available time slots disappear after creating a proposal; EA has to manually delete shadow holds before creating new proposals

### Risk Impact Analysis

| Priority | Risk | Impact if Undetected | Likelihood |
|----------|------|---------------------|------------|
| #1 | Timezone Conversion Errors | Meetings at wrong times across global teams | High - non-standard offsets are common |
| #2 | Past Date Acceptance | Junk data, wasted effort | High - no validation exists currently |
| #3 | Shadow Hold Collisions | Tool becomes unusable for multi-proposal scenarios | Medium - depends on EA workflow pace |
| #4 | Availability Window Collapse | Core value proposition fails for complex meetings | Medium - increases with stakeholder count |
| #5 | Calendar Status Misinterpretation | Double-bookings or missed opportunities | Medium - varies by calendar provider |
| #6 | Silent Data Flow Failure | EA thinks scheduling is handled when it isn't | Low - but catastrophic when it occurs |
| #7 | Email Format Variations | Wrong or missing attendees | Low - most EAs use consistent formats |

---

## Priority Quality Risk

### Risk Statement
**Timezone Conversion Errors** - The system miscalculates available time slots when stakeholders span non-standard timezone offsets (e.g., India UTC+5:30, Nepal UTC+5:45, Newfoundland UTC-3:30), producing meeting suggestions that are 15-30 minutes off from the correct time.

**Scenario:** EA proposes a meeting with stakeholders in New York (EST), Dubai (GST), and Mumbai (IST). The system calculates Mumbai's offset as +5:00 instead of +5:30, suggesting a 2:30pm IST slot that actually corresponds to 2:00pm IST - meaning the Mumbai stakeholder joins 30 minutes late or misses the meeting entirely.

**Consequence for adoption:** Impact rated 4/5. If the agent produces incorrect timezone conversions, the EA's trust is broken immediately. She already makes manual timezone mistakes - the whole point of the tool is to eliminate that error source. Getting it wrong means the tool is worse than the manual process.

### Prioritization Rationale
- **Impact (4/5):** Incorrect meeting times cause real-world scheduling failures across global teams
- **Likelihood (Monthly/Quarterly):** Non-standard timezone stakeholders are encountered regularly but not daily - making this a recurring risk that's easy to miss in testing
- **Actionability (High):** Likely fixable with proper timezone library usage (moment-timezone, luxon) rather than manual offset calculation. Easy to address once identified.
- **Sprint relevance:** Directly affects Sprint 1 prototype credibility and Sprint 2 user validation with real stakeholders
- **Why this over Past Date Acceptance (#2):** While past dates are more frequent (no validation exists), they're a simpler UI fix. Timezone errors are more insidious - they can look correct while being subtly wrong, making them harder to catch without systematic testing.

### Testing Approach
- **Inputs that would reveal this risk:** Meeting requests involving stakeholders in non-standard timezone countries (India +5:30, Iran +3:30, Nepal +5:45, Newfoundland -3:30, Myanmar +6:30)
- **Output patterns to watch for:** Time slot suggestions that are off by 15 or 30 minutes; UTC calculations that round to nearest hour; timezone labels that don't match actual offsets
- **How to recognize it:** Compare system-suggested times against manual calculation for each stakeholder's local time; verify offset arithmetic for non-standard zones

### Success Criteria
- **Acceptable performance:** 100% accuracy on timezone conversions for all tested timezone pairs (no tolerance for "close enough" - 30 minutes off is a missed meeting)
- **Tolerance levels:** Zero tolerance for offset errors; timezone display must match IANA timezone database
- **Consistency needed:** Same input stakeholder set must produce identical time suggestions across multiple runs

---

## Test Case Design Methodology

### Chosen Generation Approach
**Failure Mode Reverse Engineering** - Starting from specific ways timezone math can go wrong and designing inputs that would trigger those failures. Chosen because the participant has strong domain knowledge of timezone edge cases from real EA scheduling experience, making it efficient to target exactly the risks that matter.

### Test Case Framework
Each test case targets a specific timezone failure mode with:
- Defined stakeholder locations and their UTC offsets
- A specific failure mode the test is designed to reveal
- Clear detection criteria for pass/fail

### Target Scenarios (8 Test Cases)

**TC1: India Half-Hour Offset**
- **Input:** Meeting between NYC (EST, UTC-5) and Mumbai (IST, UTC+5:30), 60 min duration
- **Expected Failure Mode:** System rounds +5:30 to +5:00, suggesting time 30 min off
- **Risk Target:** Non-standard offset rounding
- **Success Criteria:** Mumbai time shows correct :30 offset from UTC (e.g., 10:00am EST = 8:30pm IST, not 8:00pm)

**TC2: Nepal 45-Minute Offset**
- **Input:** Meeting between London (GMT, UTC+0) and Kathmandu (NPT, UTC+5:45), 30 min duration
- **Expected Failure Mode:** System rounds +5:45 to +6:00, producing 15-min error
- **Risk Target:** 45-minute offset handling
- **Success Criteria:** Kathmandu time correctly reflects :45 offset (e.g., 12:00pm GMT = 5:45pm NPT, not 6:00pm)

**TC3: Offset Direction Error**
- **Input:** Meeting between NYC (EST, UTC-5) and Dubai (GST, UTC+4), 60 min duration
- **Expected Failure Mode:** System subtracts offset instead of adding (or vice versa), producing 9-hour error instead of correct 9-hour gap
- **Risk Target:** Offset sign/direction handling
- **Success Criteria:** Dubai is correctly shown as 9 hours ahead of NYC (e.g., 9:00am EST = 6:00pm GST)

**TC4: DST Transition Week**
- **Input:** Meeting between NYC and London, scheduled for first Sunday of November (US DST ends) and last Sunday of March (US DST begins)
- **Expected Failure Mode:** During DST transition, offset between NYC and London changes from 5 to 4 hours (or vice versa); system uses stale offset
- **Risk Target:** Daylight saving time awareness
- **Success Criteria:** System correctly reflects 4-hour gap (EDT-GMT) in summer and 5-hour gap (EST-GMT) in winter for the specific meeting date

**TC5: Two Non-Standard Zones**
- **Input:** Meeting between Mumbai (IST, UTC+5:30) and Tehran (IRST, UTC+3:30), 60 min duration
- **Expected Failure Mode:** Both offsets rounded independently, compounding to incorrect gap between the two cities
- **Risk Target:** Multiple non-standard offset interaction
- **Success Criteria:** Gap between Mumbai and Tehran correctly shown as exactly 2 hours (e.g., 3:30pm IST = 1:30pm IRST, not 2:00pm)

**TC6: IST Ambiguity (India vs Ireland)**
- **Input:** Meeting between Dublin (IST/Irish, UTC+1 summer) and Mumbai (IST/India, UTC+5:30)
- **Expected Failure Mode:** System confuses the two IST abbreviations, applying wrong offset to one city
- **Risk Target:** Timezone abbreviation ambiguity
- **Success Criteria:** System uses IANA timezone names (Europe/Dublin, Asia/Kolkata) internally rather than ambiguous abbreviations; correct offset applied to each

**TC7: Newfoundland DST**
- **Input:** Meeting between Toronto (EST, UTC-5) and St. John's (NST, UTC-3:30 winter / NDT, UTC-2:30 summer), tested for both a January and July date
- **Expected Failure Mode:** Half-hour offset combined with DST shift creates complex conversion the system gets wrong
- **Risk Target:** Non-standard offset + DST combination
- **Success Criteria:** Winter: St. John's is 1:30 ahead of Toronto. Summer: St. John's is 1:30 ahead of Toronto (both observe DST). Offset gap stays consistent.

**TC8: 4-Party Global Meeting**
- **Input:** Meeting between NYC (EST), London (GMT), Mumbai (IST +5:30), and Tokyo (JST +9), 60 min duration
- **Expected Failure Mode:** System handles standard zones (NYC, London, Tokyo) correctly but breaks on Mumbai's +5:30, or fails to find reasonable-hours overlap for all 4
- **Risk Target:** Multi-party non-standard offset in complex scenario
- **Success Criteria:** All 4 local times are mathematically correct simultaneously; suggested time falls within 7am-9pm for all parties (or system correctly reports no reasonable window exists)

### Success Criteria Design
- **Pass:** All timezone conversions match manual calculation exactly (zero tolerance)
- **Fail:** Any conversion off by 15+ minutes
- **Partial:** Standard timezone pairs correct but non-standard offsets wrong (indicates library/configuration issue rather than fundamental design flaw)

---

## Learning Objectives

### Learning Outcomes
Running these 8 test cases will reveal:

1. **How the risk manifests:** Whether the system rounds non-standard offsets to the nearest hour, ignores DST transitions, confuses ambiguous timezone abbreviations, or applies offsets in the wrong direction. The specific failure pattern determines the fix.

2. **When it occurs:** TC1-TC2 isolate non-standard offset handling. TC3 isolates direction logic. TC4/TC7 isolate DST awareness. TC5-TC6 test compound scenarios. TC8 tests real-world complexity. If TC1-TC2 fail but TC3 passes, the issue is fractional offset handling, not fundamental timezone logic.

3. **How severe it is:** If errors are consistently 15-30 minutes (offset rounding), the fix is straightforward - use a proper timezone library with IANA database. If errors are variable or directional (TC3), there's a deeper architecture issue. If only TC4/TC7 fail, DST data needs updating.

4. **What to improve:** Test results map directly to fixes:
   - TC1/TC2 failures → Switch from manual offset math to timezone library (moment-timezone, luxon)
   - TC3 failure → Fix offset sign handling in conversion logic
   - TC4/TC7 failures → Ensure DST rules are current; use date-aware conversions
   - TC5 failure → Verify library handles zone-to-zone conversion (not just zone-to-UTC)
   - TC6 failure → Use IANA names (Asia/Kolkata) instead of abbreviations (IST)
   - TC8 failure → Debug multi-party overlap algorithm separately from timezone conversion

---

## Evaluation Artifacts Generated

### Files Created
1. **`workspace/data/evaluations_data.csv`** - Executable evaluation matrix with 8 test cases targeting timezone conversion errors
   - Pre-populated columns: test_case_id, risk_category, input_description, expected_challenge, learning_objective, input_data
   - Empty columns for execution: actual_output, quality_rating (1-5), notes, patterns_observed, improvement_ideas, completed_date

### How to Use the CSV
1. **Run tests:** For each row, use the `input_data` values to set up a meeting proposal in the Shadow Hold Assistant
2. **Record output:** Fill in `actual_output` with the system's suggested times for each stakeholder
3. **Rate quality:** Score 1-5 based on success criteria (5 = all times correct, 1 = major conversion error)
4. **Capture patterns:** Note recurring issues in `patterns_observed` (e.g., "all half-hour offsets rounded")
5. **Document fixes:** Use `improvement_ideas` to track what needs changing

---

## Next Steps After Evaluation

1. **Run the 8 test cases** against the Shadow Hold Assistant once timezone-aware scheduling is implemented
2. **Analyze failure patterns** - Do failures cluster around non-standard offsets? DST? Multi-party scenarios?
3. **Fix identified issues** - Use learning objectives to map failures to specific fixes
4. **Re-run failing tests** after fixes to verify resolution
5. **Expand evaluation** - Re-run this workflow for other priority risks (Past Date Acceptance, Shadow Hold Collisions) once timezone risk is addressed
6. **Scale to Sprint 2** - Add real-world test cases from your wife's actual scheduling scenarios
