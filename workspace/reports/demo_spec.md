# Demo Specification — Shadow Hold Assistant

## audience_focus

### priority_audience
- **Primary segment:** Future users — EAs, Chiefs of Staff, and companies that employ executive assistants
- **Secondary segment:** Anyone evaluating what's now possible when a non-developer uses AI tools seriously
- **What they want:** Proof that this solves a real scheduling problem; evidence it actually works
- **Success metric:** Someone from the audience reaches out after the demo asking "how do I get this for my EA?"

### key_achievement
As a Design Strategist with zero development background, Brian pushed his technical skills to the maximum and built a working, deployed AI agent — for a real user (his wife, a professional EA) — in 7 weeks.

This is not a prototype. It runs end-to-end:
- Receives a scheduling request via webhook
- Uses Claude to convert timezones, validate business hours, and draft a scheduling email
- Logs the meeting to Supabase
- Creates a shadow hold on Google Calendar

**The breakthrough:** He didn't just learn tools — he shipped something real.

### core_message
*"A non-developer built a live AI agent that solves a real problem for a real user in 7 weeks. Here's how."*

---

## narrative_structure

### chosen_arc
Option A: "The Non-Developer's Journey"

### 3_act_structure

**ACT 1 — Setup (1.5 min): "Who I Was"**
- Identity: Design Strategist at Big 4, zero development background
- The real problem: Wife is a professional EA — hours/week on manual timezone lookups, handwritten scheduling emails, shadow holds tracked mentally
- The gap: Existing tools (Cabinet etc.) don't use AI for the intelligence layer

**ACT 2 — The Journey (4 min): "What I Built"**
- The decision: Build it yourself using AI tools
- The climb: GitHub, Docker, n8n, Supabase, Anthropic API — all learned in 7 weeks
- The breakthrough: AI Agent producing first correct scheduling output (execution #10 — end-to-end success)
- **[LIVE DEMO HERE]** Send a webhook → Claude analyzes timezones + flags business hours → shadow hold appears on Google Calendar

**ACT 3 — Resolution (2.5 min): "What It Means"**
- Results: Handles complex multi-timezone scheduling with tiered business hours judgment (7/8 test cases pass, TC8 edge case identified and fixed)
- Real user validation: Wife tested it — this is a real product, not a prototype
- The bigger point: If a designer with no dev background can ship a working AI agent in 7 weeks, the bar has fundamentally changed
- Close: What's next (EA approval UI, email automation, broader rollout)

---

## visual_storyboard

### ACT 1 — Setup (0:00–1:30)

| Time | What's on screen | What you're saying |
|---|---|---|
| 0:00–0:30 | **Slide: Your intro** — Name, title, "Design Strategist. Not a developer." | Set up who you are and why that matters |
| 0:30–1:00 | **Slide: The problem** — Simple diagram of EA's current manual workflow (meeting request → manual timezone lookup → write email → track in notes) | Describe your wife's daily reality |
| 1:00–1:30 | **Slide: The gap** — Logos of existing tools (Cabinet, Calendly) with "Missing: AI intelligence layer" | Explain why existing tools fall short |

### ACT 2 — The Journey (1:30–5:30)

| Time | What's on screen | What you're saying |
|---|---|---|
| 1:30–2:00 | **Slide: The stack** — GitHub, Docker, n8n, Supabase, Anthropic logos | "7 weeks ago I had never used any of these" |
| 2:00–2:30 | **Slide: The n8n workflow** — Screenshot of your workflow canvas (Webhook → AI Agent → Supabase → Calendar) | Walk through the architecture simply |
| 2:30–5:30 | **[RECORDED DEMO]** Screen recording: terminal curl → n8n Executions tab (all green) → Supabase row with ai_response → Google Calendar shadow hold event | Narrate over the recording as it plays |

### ACT 3 — Resolution (5:30–8:00)

| Time | What's on screen | What you're saying |
|---|---|---|
| 5:30–6:00 | **Slide: Results** — "7/8 test cases pass. TC8 (4-timezone global meeting) identified and fixed." | Show it was rigorously tested |
| 6:00–6:30 | **Slide: Real user** — Photo of your wife (optional) or quote from her feedback | "My wife is the user. She tested it." |
| 6:30–7:30 | **Slide: The bigger point** — Architecture diagram with the phrase "1 agent. 3 services. Built by a designer." | Land the core message |
| 7:30–8:00 | **Slide: What's next** — Roadmap: EA approval UI → email automation → deploy for other EAs | Close with vision and call to action |

---

## script

### ACT 1 — Setup (0:00–1:30)

*[Slide: Your intro]*

"My name is Brian Sullivan. I'm a Design Strategist at a Big 4 consultancy. I facilitate design thinking workshops for Fortune 25 companies. I am not a developer.

Seven weeks ago, I had never used GitHub. I didn't know what Docker was. I'd never heard of n8n.

Today I'm going to show you a working AI agent I built — for a real user, solving a real problem, running right now."

*[Slide: The problem]*

"My wife is a professional Executive Assistant. Part of her job is scheduling complex meetings — executives, prospects, partners — across multiple timezones, multiple continents.

Every time she does it, she manually looks up what 2pm New York is in London, Mumbai, Tokyo. She writes the scheduling email herself. She creates a tentative calendar block — called a shadow hold — by hand. And she tracks the whole thing in her head.

It takes 20 to 30 minutes for a complex global meeting. She does this multiple times a week."

*[Slide: The gap]*

"Tools like Cabinet exist. Calendly exists. But none of them do the intelligence work — the timezone judgment, the business hours check, the draft email. They're scheduling pipes. Not scheduling intelligence.

That's the gap I decided to fill."

---

### ACT 2 — The Journey (1:30–5:30)

*[Slide: The stack]*

"So I enrolled in this bootcamp and made a decision: I was going to build it myself.

Seven weeks ago, none of this meant anything to me. [point to stack logos] GitHub, Docker, n8n, Supabase, the Anthropic API.

Today, every one of these is part of my working toolkit. Not because I'm suddenly a developer — but because AI tools lowered the bar enough that a designer could cross it."

*[Slide: n8n workflow canvas]*

"Here's what I built. A five-node workflow in n8n.

A webhook receives a scheduling request. An AI Agent — powered by Claude — analyzes the timezones, checks business hours for every participant, and drafts a scheduling email. Supabase logs the meeting. Google Calendar creates the shadow hold automatically.

That's it. One agent. Three services. Built by a designer."

*[RECORDED DEMO plays]*

"Let me show you it running.

I'm sending a real scheduling request — three participants, New York, London, Mumbai — proposed meeting at 2pm UTC.

Watch Claude's output. It converts every timezone. It checks whether 9am New York, 2pm London, 7:30pm Mumbai fall within reasonable working hours. It flags the status. It drafts the email.

Now watch Supabase — there's the new row, with the full AI analysis stored.

And Google Calendar — the shadow hold just appeared."

---

### ACT 3 — Resolution (5:30–8:00)

*[Slide: Results]*

"I didn't just build this and hope it worked. I designed an evaluation dataset — 8 test cases covering edge cases like non-standard timezone offsets, DST transitions, and 4-party global meetings.

7 out of 8 passed. The one that failed revealed a real gap — business hours weren't being flagged for a 4am New York participant. I caught it, fixed it, shipped it.

That's the methodology: build, test, iterate."

*[Slide: Real user]*

"My wife has tested this. She's the user this was built for.

[INSERT HER QUOTE HERE when you have it]

This is not a prototype. This is a tool built for a specific person, tested by that person, solving their actual problem."

*[Slide: The bigger point]*

"Here's what I want you to take away from this.

Seven weeks ago, I couldn't build an AI agent. Today I can. Not because I became a developer — but because the tools changed.

If a Design Strategist with zero dev background can ship a working, tested, deployed AI agent in 7 weeks... what does that mean for your team? For your clients? For the problems you've been waiting for someone else to solve?

The bar has changed. We're all builders now."

*[Slide: What's next]*

"Next: an approval interface so my wife can review and send the email directly from the tool. Then email automation. Then making this available to other EAs.

The agent is live. The problem is real. The user is waiting.

Thank you."
