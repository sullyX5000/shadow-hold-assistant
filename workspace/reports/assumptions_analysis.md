# Assumptions Analysis: Shadow Hold Assistant

## Step 2: Challenge Your Assumptions - Results

### Validated Constraints
*These are confirmed truths you must design around*

#### 1. Time Reduction Target (70%)
**Status:** Validated - solving for a real logistical challenge

**Implication:** The pain is real and significant. Double-booking scrambles derail productivity across multiple high-salary individuals. The 70% target isn't arbitrary - it reflects genuine operational pain.

#### 2. Stakeholder Approval Requirements
**Status:** Validated - Approval needs to come from every stakeholder involved in the meeting, which can be challenging for multiple schedules and global time zones

**Implication:** Multi-party coordination across time zones is the core complexity. Your solution must handle asynchronous responses from multiple stakeholders, not just binary yes/no from one person.

**Note:** *This may differ from EA approval of AI-drafted emails - clarify if EA also needs to approve before sending, or just stakeholders approving meeting times.*

#### 3. Double-Booking Impact
**Status:** Validated - Painful enough that it derails someone's productivity when they have to stop what they're doing and scramble to find matching calendar time for 3+ people

**Implication:** The cost isn't just EA time - it's executive time + stakeholder time multiplied by salary rates. One double-booking incident could cost hundreds/thousands in lost productivity.

**Design Insight:** Shadow holds aren't a "nice to have" - they're essential collision prevention for high-value time.

#### 4. Speed vs. Executive Bottleneck
**Status:** Validated - the executives and stakeholders are the bottleneck

**Critical Finding:** If executives are the bottleneck, then optimizing EA speed won't solve the problem. Your agent should focus on:
- Making it easier for executives to respond quickly (mobile-friendly confirmations?)
- Tracking who hasn't responded so EA can nudge the right people
- Reducing cognitive load on executives when reviewing time proposals

**Strategic Question:** Should your MVP target executive experience, not just EA experience?

---

### Flexible Assumptions
*These could be adjusted based on technical constraints or user feedback*

#### 5. AI Email Quality ("all of the above")
**Status:** Flexible - needs relationship nuances, executive preferences, and organizational politics

**Implication:** This is harder than "professional tone." Your AI needs contextual memory:
- Who gets formal vs. warm tone
- What information should/shouldn't be mentioned
- Relationship history and sensitivities

**MVP Approach:** Start with templated emails the EA can customize, not fully autonomous AI drafting. Build the context database first (stakeholder profiles, tone preferences), then layer AI on top.

**Risk:** If you over-promise AI email quality and under-deliver, your wife won't trust the tool.

#### 6. Single Calendar Integration
**Status:** Flexible - they use Outlook in the office, but executive leadership may be using Google

**Critical Finding:** This is a major scope issue. If executives use Google and EA uses Outlook:
- You need *both* integrations for MVP
- OR you need to choose which user's pain you're solving (EA vs. executive)
- Single calendar integration may not be viable

**Decision Point:** Which calendar system is the primary source of truth? Where does the EA create the shadow holds - in Outlook, Google, or both?

**Recommendation:** Talk to your wife about the actual calendar workflow before committing to technical stack.

#### 7. n8n as Primary Tool
**Status:** Flexible - both solving problem AND learning n8n, recognized as part of bootcamp training

**Implication:** You're balancing two goals: (1) solve wife's problem, (2) learn n8n. Both are valid, but they may conflict.

**Strategic Guidance:**
- If n8n can't handle the requirements, you'll need to choose: switch tools or reduce scope
- Bootcamp encourages learning, but doesn't mandate specific tools
- Consider: n8n for orchestration, but you may need custom code for calendar API complexity

**Question to explore:** Is n8n good at real-time event monitoring (like watching for email responses)? Or is it better for triggered workflows?

#### 8. Cabinet Usage & Competition
**Status:** Flexible - yes she uses Cabinet for its effectiveness, but unsure if other competitors exist

**Strategic Finding:** She already uses Cabinet's shadow hold feature successfully. This means:
- The shadow hold concept is validated (she finds it valuable)
- Your competitor has proven the feature works
- Your differentiation must be AI automation, not the shadow hold concept itself

**Competitive Analysis Needed:**
- What does Cabinet charge?
- What friction points does Cabinet have that your AI could eliminate?
- Could you integrate with Cabinet rather than replace it?

#### 9. Root Cause - Executive Behavior
**Status:** Flexible - double-bookings happen due to poor communication from executives and their inability/discipline to follow their calendar

**Critical Insight:** This reveals the real problem isn't EA workflow - it's executive calendar discipline.

**Implications:**
- Shadow holds won't help if executives ignore them and double-book anyway
- Your solution may need executive-facing features (reminders, notifications, mobile alerts)
- Consider: Should executives see tentative blocks clearly marked as "PENDING CONFIRMATION"?

**Strategic Question:** Is your MVP building an EA tool or a behavior change tool for executives?

---

### Untested Beliefs
*You haven't validated these yet - they're technical risks for MVP*

#### 10. Email Monitoring Can Be Automated Reliably
**Status:** Untested - assumption

**Why This Is Critical:** This is the core technical risk of your entire MVP. Email parsing is notoriously difficult:
- Varied response formats ("Yes", "Works for me", "Can we push 30 min?")
- Threading issues (reply to wrong email)
- Ambiguous responses ("Let me check with my team" = yes or no?)
- Spam filtering and deliverability
- Privacy concerns (reading executive email)

**MVP Risk:** If you can't reliably monitor email responses, your entire automatic shadow hold conversion feature fails.

**Recommendation:** Validate this assumption in Sprint 1 before building full system:
1. **Week 1 Task:** Build a simple email response parser
2. Test with 20 real scheduling email threads from your wife
3. Measure: Can you accurately categorize responses as Confirmed/Declined/Pending?
4. If accuracy < 80%, pivot to manual EA confirmation instead of automatic

**Alternative Approach:** Instead of monitoring email, send confirmation links:
- AI drafts email with "Click here to confirm" button
- Link goes to simple webpage with Yes/No/Propose Alternative
- Much more reliable than parsing free-text email responses
- Still saves EA time (no email composition, automatic tracking)

---

## Strategic Recommendations

Based on your categorization, here are the highest-priority items to address before building:

### 🔴 Critical Path Items (Must Validate in Sprint 1)

1. **Email Response Monitoring Feasibility**
   - Build proof-of-concept parser with real email samples
   - If this fails, pivot to confirmation link approach
   - Don't build the full system until you validate this works

2. **Multi-Calendar Reality Check**
   - Interview your wife: "Walk me through creating a shadow hold today. Which calendar do you use? Where does the executive see it?"
   - If answer is "both Outlook and Google," your MVP scope just doubled
   - Consider: Can you start with shadow holds in ONE calendar and validate value first?

3. **EA vs. Executive as Primary User**
   - Your assumptions reveal executives are the bottleneck AND the cause of double-booking
   - Decide: Is this an EA tool (helps EA work faster) or an executive tool (changes executive behavior)?
   - Different user = different features

### 🟡 Scope Refinement (Week 1-2)

4. **AI Email Drafting Complexity**
   - Don't promise AI-generated context-aware emails for MVP
   - Start with: EA-authored templates + AI fills in [date/time/names]
   - Build contextual awareness in Sprint 2 after validating basic flow

5. **Cabinet Integration vs. Replacement**
   - Since your wife uses Cabinet successfully, explore: Can you build an AI layer ON TOP of Cabinet?
   - Might be faster than rebuilding calendar integration from scratch
   - Check Cabinet's API documentation

### 🟢 Keep As-Is (Already Well-Scoped)

6. **n8n for Orchestration**
   - Your dual goal (solve problem + learn n8n) is clear
   - Keep n8n for workflow orchestration
   - But stay flexible: if n8n can't handle real-time email monitoring, supplement with custom code

7. **70% Time Reduction Target**
   - Well-validated pain point
   - Clear success metric for final demo

---

## Next Steps in Ideation Workflow

You've now completed **Step 2: Challenge Your Assumptions**.

The next steps in the ideation workflow are:
- **Step 3:** Solution Ideation (generate multiple approaches)
- **Step 4:** Evaluation Criteria (define how to choose between approaches)
- **Step 5:** Solution Selection (pick your MVP approach)
- **Step 6:** Implementation Planning (break down into sprints)

Before moving forward, I recommend you:
1. **Clarify the multi-calendar question with your wife** (Outlook vs. Google)
2. **Test email response parsing** with 5-10 real examples
3. Then proceed to Step 3 with a clearer technical understanding

Would you like to continue to Step 3 now, or pause to investigate the multi-calendar and email parsing questions first?
