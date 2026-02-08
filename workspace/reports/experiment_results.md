# AI Playground Experiment Results
## Shadow Hold Assistant - Sprint 1 Validation Experiments

---

## experiments_conducted

### Experiment 1: AI-Generated Form Component (Lovable)
**Date:** Feb 2-3, 2025
**Tool:** Lovable (AI-powered frontend builder)
**Hypothesis:** An AI can generate a production-ready React form for meeting scheduling from a plain English description.
**Method:** Described the Shadow Hold Assistant form requirements to Lovable in natural language. Requested fields for meeting subject, stakeholder emails, and 3 time options with date pickers and duration selectors.
**Prompt:** "Create a meeting proposal form with: subject field, stakeholder email input, and 3 time option rows each with date picker, time input, and duration selector (30/60/90/120 min). Include form validation and a submit button."
**Result:** Lovable generated a complete MeetingProposalForm.tsx with all fields, Tailwind styling, and basic validation. However, the generated submit handler used `mailto:` and localStorage instead of a webhook POST.
**Finding:** AI can generate complex UI components rapidly, but business logic (specifically how data should be transmitted) requires explicit specification. The AI made a reasonable assumption (email = mailto) that didn't match our use case.

---

### Experiment 2: Debugging AI-Generated Code with AI
**Date:** Feb 3, 2025
**Tool:** Claude (conversation-based debugging)
**Hypothesis:** AI can diagnose why a form isn't working as expected by analyzing the generated code.
**Method:** Pasted the Lovable-generated handleSubmit function and described the symptom (Apple Mail opens instead of sending to n8n webhook). Asked AI to identify the bug.
**Prompt:** "Why is my form opening Apple Mail instead of posting to my webhook? Here's the handleSubmit: [code paste]"
**Result:** AI immediately identified `window.location.href = mailto:${emails}` and the localStorage.setItem calls as the culprit. Explained the generated code was built for email-based submission, not webhook-based. Provided corrected async fetch() implementation.
**Finding:** AI is highly effective at diagnosing issues in AI-generated code. The diagnostic loop (AI generates → AI debugs) is faster than manual debugging for this type of integration issue.

---

### Experiment 3: n8n Workflow Generation from Natural Language
**Date:** Feb 4, 2025
**Tool:** Claude + n8n
**Hypothesis:** AI can describe an n8n workflow structure (nodes and connections) that a developer can implement without n8n expertise.
**Method:** Described the desired automation flow to Claude and asked for a step-by-step n8n workflow blueprint.
**Prompt:** "I need an n8n workflow that: receives a webhook POST with meeting subject, stakeholders, and 3 time options → saves meeting to Supabase → splits the time options → creates a Google Calendar event for each time option → saves calendar event IDs back to Supabase. Give me the exact node sequence."
**Result:** Claude provided a detailed workflow blueprint: Webhook node → Set node (parse body) → Supabase Insert (meetings) → Code node (split time options) → Loop Over Items → Google Calendar Create Event → Supabase Insert (time_options). This matched the actual n8n implementation almost exactly.
**Finding:** AI can effectively bridge the gap between business requirements and tool-specific implementation for workflow automation tools like n8n. The natural language to workflow translation was accurate and actionable.

---

### Experiment 4: Supabase Schema and Query Generation
**Date:** Feb 4-5, 2025
**Tool:** Claude
**Hypothesis:** AI can generate correct Supabase relational queries including joined table syntax from schema description.
**Method:** Described the meetings + time_options schema and asked for a query that fetches meetings with their related time options.
**Prompt:** "Given this schema: meetings(id, subject, status, created_at) and time_options(id, meeting_id, datetime, calendar_event_id, calendar_provider), write a Supabase JS query to fetch all meetings with their time options, ordered by created_at descending."
**Result:** AI generated the exact query: `.from("meetings").select("id, subject, status, created_at, time_options(id, datetime, calendar_event_id, calendar_provider)").order("created_at", { ascending: false })` - which worked correctly on first attempt.
**Finding:** AI has strong knowledge of Supabase's nested select syntax. Zero debugging needed. Saves significant time vs. reading Supabase documentation for relational queries.

---

### Experiment 5: Dashboard Component Generation
**Date:** Feb 5, 2025
**Tool:** Claude + Lovable
**Hypothesis:** AI can generate a complete React dashboard component that reads live data from Supabase and displays it with appropriate UI patterns (cards, badges, loading states).
**Method:** Described the dashboard requirements and provided the Supabase query as context. Asked AI to generate the full component.
**Prompt:** "Create a React TypeScript component that fetches meetings from Supabase using this query [query], displays each meeting in a Card with subject, status Badge, created date, and a list of time options. Include loading and error states. Use shadcn/ui components."
**Result:** Generated complete MeetingsDashboard.tsx with useEffect + useState hooks, conditional rendering for loading/error/empty states, proper TypeScript interfaces, and correct shadcn/ui Card and Badge usage. Component worked on first render.
**Finding:** AI excels at generating React components when given specific data structures and UI library constraints. The combination of "here's my data shape" + "use these specific components" produces high-quality, immediately usable code.

---

### Experiment 6: Prompt Engineering for Meeting Information Extraction
**Date:** Feb 6, 2025
**Tool:** Claude (AI playground testing)
**Hypothesis:** A structured prompt can reliably extract meeting scheduling data (subject, participants, time preferences) from natural language meeting requests.
**Method:** Used the testing prompt template (see workspace/prompts/testing_prompt.txt) with 5 different meeting scenarios. Measured accuracy of extracted fields and quality of suggested time slots.
**Test inputs:** Simple bilateral, multi-stakeholder, cross-timezone, urgent, and recurring meeting scenarios.
**Results:**
- Test 1 (Simple bilateral): 5/5 fields extracted correctly. Suggested times were reasonable business hours.
- Test 2 (Multi-stakeholder): Correctly identified all 5 participants. Noted the deadline constraint.
- Test 3 (Cross-timezone): Correctly avoided problematic time windows for 3 timezones. Suggested overlap window of 9-11am GMT.
- Test 4 (Urgent): Appropriately prioritized same-day slots. Correctly kept duration to 30 min.
- Test 5 (Recurring): Generated ISO-format recurring event pattern. Added appropriate end date.
**Finding:** Structured prompts with explicit output format requirements produce highly consistent, machine-parseable results. The AI handles edge cases (timezone conflicts, urgency) well when these constraints are present in the input.

---

### Experiment 7: Iterative Prompt Refinement for Calendar Event Description
**Date:** Feb 7, 2025
**Tool:** Claude
**Hypothesis:** Prompt refinement through 3-4 iterations produces significantly better calendar event descriptions than the initial prompt.
**Method:** Started with a basic prompt for generating Google Calendar event details from a meeting request. Refined based on output quality through 4 iterations.
**Iteration 1:** "Create a Google Calendar event for this meeting: [request]"
- Result: Generic title, no tentative flag mentioned, no attendees
**Iteration 2:** Added "Mark as tentative, include all participants, use transparent status"
- Result: Better but duration was inconsistent
**Iteration 3:** Added "Duration in minutes as integer, ISO 8601 format for start/end times"
- Result: Correct format but missed timezone handling
**Iteration 4:** Added timezone specification and "set transparency to 'transparent' for shadow hold purposes"
- Result: Complete, correctly formatted event payload ready for Google Calendar API
**Finding:** Each prompt refinement cycle approximately doubled output quality. Final prompt produces output that can be directly used as API payload with no reformatting. This validates the importance of iterative prompt engineering over one-shot prompting for production use cases.

---

## successful_patterns

1. **Explicit output format specification**: Providing exact field names and data types in the prompt eliminates post-processing. Example: "Return ISO 8601 datetime, not human-readable format."

2. **Context + constraint bundling**: Giving AI both the data schema AND the UI library constraints produces immediately usable code. "Use shadcn/ui Card and Badge components" vs. generic UI guidance.

3. **Diagnostic loop acceleration**: Using AI to debug AI-generated code is effective because the AI understands its own common patterns and assumptions. Symptom description alone is often sufficient.

4. **Workflow blueprint generation**: Describing automation requirements in business terms and asking for tool-specific (n8n) implementation works well when the tool is well-documented in training data.

5. **Iterative refinement over one-shot**: 3-4 refinement cycles on prompts produce significantly better results than trying to perfect the initial prompt. Start rough, refine based on output gaps.

---

## failure_modes

1. **Assumed transmission method**: Lovable AI assumed email (mailto) was the data transmission method because it's common in form builders. Without explicit "POST to webhook URL", AI defaulted to familiar patterns.

2. **Missing business logic context**: AI doesn't know what a "shadow hold" is from technical terms alone. Required explicit explanation of the concept before it could generate appropriate code.

3. **Timezone handling gaps**: Initial prompts didn't mention timezone context, causing AI to generate naive datetime handling. Multi-timezone scenarios require explicit constraints.

4. **Over-engineering tendency**: AI sometimes adds features not requested (e.g., adding a "clear form" button, extra validation messages). Requires "only implement what I described" instruction to avoid scope creep in generated code.

5. **Stale documentation assumptions**: AI occasionally suggested deprecated API patterns (older Google Calendar API syntax). Required verifying against current API docs.

---

## refined_approach

Based on these experiments, the optimal approach for AI-assisted development of the Shadow Hold Assistant:

**Phase 1 - Architecture decisions**: Use AI for brainstorming and trade-off analysis. Human makes final decisions.

**Phase 2 - Component generation**: Use AI with explicit schema + UI library constraints. Expect one revision cycle for business logic alignment.

**Phase 3 - Integration debugging**: Use AI for diagnostic analysis. Provide symptoms + code context together.

**Phase 4 - Prompt engineering**: Use iterative refinement (3-4 cycles). Define output format explicitly from iteration 1.

**Key principle**: AI is most effective when given concrete constraints (schema, library names, API formats) rather than abstract requirements. The more specific the input, the more production-ready the output.

**Next validation step**: Test the full pipeline prompt (natural language meeting request → shadow hold creation) end-to-end in the live app with real Google Calendar events to measure accuracy and user satisfaction.
