# First Deployment Design
## Shadow Hold Assistant

---

## testing_reality_check

### Who Has Tested It

**Brian (builder):** All 8 test cases in the evaluation dataset were run by Brian himself through the Lovable frontend. He submitted meeting proposals, read the SHADOW HOLD ANALYSIS output, and verified timezone math manually against known correct values. He was present for every session.

**Brian's wife (primary user / EA):** Has tested the agent in a live session with Brian present. She is a professional Executive Assistant at a Fortune 500 company — the exact target user. She confirmed the core product direction, validated that timezone handling is the hardest part of her current workflow, and established that human approval before anything goes external is non-negotiable.

### How Testing Happened

All testing was conducted with Brian present — either as the tester himself or sitting alongside his wife. The Cloudflare quick tunnel was active during each session (manually started by Brian before testing). The n8n Docker container was running on Brian's local machine. No testing has happened asynchronously without Brian's involvement.

This means: **the agent has never run for a user without Brian in the room or at the keyboard.**

### Scenarios Covered

- 8 timezone edge cases (TC1-TC8) including non-standard offsets, DST transitions, 4-party global meetings
- Basic happy path: single meeting proposal with 2 standard-timezone participants
- The TC8 discovery: business hours judgment failure on 4:00 AM NYC slot (now fixed)
- End-to-end pipeline: Lovable form → webhook → n8n → Claude → Supabase → Google Calendar

### Gaps Between Theoretical Coverage and Reality

1. **No unsupervised use:** Brian has always been present. The agent has not run when he was asleep, away, or unavailable.
2. **Brian formats inputs carefully:** As the builder, he knows what input format the webhook expects. His wife has seen the form, but he was there guiding her.
3. **Tunnel availability assumed:** Every test assumed the Cloudflare tunnel was running. A real user hitting the app when the tunnel is down gets a silent failure.
4. **One user profile:** All testing has been with Brian's wife's scheduling scenarios — a Fortune 500 EA in the US. No testing with the variation a stranger would bring.
5. **TC8 class failures may still exist:** The business hours fix was applied to the system prompt. It hasn't been re-tested against TC8 after the fix was confirmed in development.

---

## deployment_purpose

**Quality risk:** Timezone conversion accuracy + business hours boundary judgment. The system must not only calculate correct times for non-standard timezone participants, but must flag and explain when a proposed meeting time falls outside reasonable working hours for any participant.

**Why current testing hasn't exposed the key issues:**

When Brian tests, he knows the right answer — he can spot a wrong timezone conversion because he calculated it manually. His wife hasn't tested the system in her actual work context (scheduling real meetings for her real executive) — she tested a demo scenario while Brian was present. Neither type of testing reveals:
- Whether she would catch a subtle 30-minute timezone error without Brian to confirm it
- Whether the business hours warning language is clear enough for her to act on without asking Brian
- Whether the output format makes sense when she's under time pressure in her real workflow
- Whether she trusts the agent enough to let it create a shadow hold without double-checking every field

**How deployment specifically helps validate the quality risk:**

Deployment means Brian's wife uses the agent for at least one real scheduling scenario from her actual work — without Brian present to guide or confirm. This will surface:
- Whether business hours warnings are legible and actionable in real conditions
- Whether she trusts the timezone output or feels the need to verify manually (if she verifies every time, the value proposition is broken)
- Whether the end-to-end flow from form submission to calendar shadow hold feels reliable or fragile

My quality risk is business hours judgment. Testing so far hasn't exposed whether the EA actually trusts the output because I've always been there to confirm it. Deployment will show me whether she acts on the agent's judgment independently or second-guesses it.

---

## tester_profile

**Primary Tester: Brian's wife**
- Role: Professional Executive Assistant at a Fortune 500 company
- Location: US-based, primarily scheduling across US timezones with some international
- Technical comfort: Non-technical — uses web browsers, email, Google Calendar daily; has never used developer tools or run code
- Device: MacBook and iPhone for work
- Access to Lovable frontend: Yes — has bookmarked meet-propose-quick.lovable.app
- Commitment: Has agreed to use the tool for 1-2 real scheduling scenarios from her actual work, with feedback via a quick voice note or text to Brian

**Testing context:** She will use the agent during her normal work day when a scheduling need arises. Brian will not be present. She will form and submit 1-2 real meeting proposals, review the SHADOW HOLD ANALYSIS output, and note whether she would act on it or feel the need to verify.

---

## deployment_scope

**What she's committed to:**
- Use the Lovable app for at least 1 real scheduling scenario (not a demo) during her work week
- Send Brian a short voice note or text after each use: "Did you trust the output? Did you verify anything manually? Did anything break?"

**Testing timeframe:**
- Start: As soon as Brian confirms the tunnel is stable and the TC8 fix is verified
- Duration: 1 work week (5 business days)
- Feedback window: Same day or next morning via voice note

**What we're asking her to validate:**
1. Was the business hours flag (if triggered) clear enough to understand and act on?
2. Did she trust the timezone conversions in the output, or did she check them manually?
3. Did the shadow hold appear on the calendar correctly after form submission?

---

## learning_goals

**Assumptions being tested:**
1. The business hours warning language added to final_prompt.txt is clear enough for a non-technical EA to understand and act on without explanation
2. A one-click form submission → calendar shadow hold flow is simple enough for her to use independently in a real work context
3. The Cloudflare tunnel + Docker setup is stable enough to survive a 5-day deployment without manual intervention from Brian

**What makes this deployment successful as learning (not adoption metrics):**
- She acts on the agent's output without asking Brian to verify the timezone math — this confirms the trust threshold has been crossed
- If she does verify, we learn exactly what made her uncertain (language? format? specific timezone?)

**What feedback triggers pivot vs iterate:**
- PIVOT: She doesn't trust the output at all and manually verifies every time — the format or language needs a fundamental rethink
- ITERATE: She trusts most of it but caught one timezone that seemed off — specific improvements to surfacing confidence signals
- CONFIRM: She used it, trusted it, and the shadow hold appeared correctly — value proposition validated for her use case

---

## current_manual_dependencies

**What currently requires builder presence:**
1. **Cloudflare tunnel:** Must be manually started with `cloudflared tunnel --url http://localhost:5678` before any webhook can reach the n8n instance. If Brian's laptop sleeps, the tunnel drops.
2. **Docker container:** n8n runs in Docker Compose on Brian's local machine. If the machine reboots or Docker crashes, the workflow is unavailable.
3. **Webhook URL in Lovable:** The VITE_WEBHOOK_URL environment variable in Lovable points to the current Cloudflare tunnel URL. If the tunnel restarts, the URL changes and the form breaks.
4. **Google Calendar OAuth:** The OAuth2 credentials in n8n are scoped to Brian's Google account. If the token expires, calendar event creation fails silently.
5. **Anthropic API key:** Stored in n8n credentials. No expiry risk, but requires Brian's account to be in good standing.

---

## independence_requirements

**What MUST work without builder present for this deployment:**

1. **Stable public webhook URL:** The Lovable form must have a URL that doesn't change between sessions. Current Cloudflare quick tunnel URL changes on restart — this is the #1 blocker for independent use.
   - Minimum fix: Use a named Cloudflare tunnel with a persistent subdomain OR switch to n8n cloud for the deployment period
   - Decision for this deployment: Use n8n cloud (free tier) for the 1-week deployment window — eliminates Docker/tunnel dependency entirely

2. **Runtime must survive without manual restart:** For a 1-week deployment to one tester, the agent must stay alive during business hours without daily intervention.
   - n8n cloud handles this automatically — no self-hosting concern

3. **Calendar shadow holds must create successfully:** The Google Calendar integration must work with a non-expiring credential for the deployment window.
   - Verify OAuth token freshness before handing off to tester

4. **Tester can submit a form and get a result:** The Lovable frontend form must post to the stable webhook URL and the response must be visible in the SHADOW HOLD ANALYSIS section.

**What's explicitly staying manual for this deployment:**

- Brian monitors n8n execution logs manually — no automated alerting
- No error notification to tester if something fails — Brian checks each morning
- No data backup — Supabase free tier, acceptable to lose data from this pilot
- Brian manually verifies TC8 fix held after the system prompt update — no automated regression test
- Google Calendar integration creates events on Brian's calendar (not his wife's) for this pilot — she verifies by checking the calendar Brian shares with her

---

## platform_selections

### Delivery Mechanism: Lovable Frontend (existing)
meet-propose-quick.lovable.app — already deployed, already bookmarked by the tester. No change needed. The form UI is the right interface for a non-technical EA.

**Rationale:** She's already used it. Changing delivery mechanism would introduce friction and risk breaking her mental model of the tool. Keep it.

### Backend Runtime: n8n Cloud (migration from local Docker)
Switch the production workflow from local Docker to n8n cloud for the 1-week deployment window.

**Rationale:**
- Eliminates the Cloudflare tunnel dependency — public webhook URL is stable and persistent
- Eliminates Docker restart risk — n8n cloud manages uptime
- Free tier (up to 5 active workflows) is sufficient for one pilot workflow
- Migration is straightforward: export workflow JSON from local n8n → import to n8n cloud → update VITE_WEBHOOK_URL in Lovable environment variables

**Cost:** Free tier — $0 for this pilot deployment

**Trade-off:** n8n cloud credentials must be reconfigured (Anthropic API key, Supabase credentials, Google Calendar OAuth). One-time setup cost, then stable for the week.

### Data Storage: Supabase (existing, no change)
meetings, time_options, and workflow_logs tables already created and working. Free tier (500MB database) is sufficient.

**Rationale:** Already integrated, already tested, no migration needed. Supabase credentials carry over to n8n cloud workflow.

### Integration Architecture:
```
Lovable (meet-propose-quick.lovable.app)
  → POST to n8n cloud webhook URL (stable, persistent)
  → n8n cloud workflow
      → Claude claude-sonnet-4-6 via Anthropic API
      → Supabase (meetings, time_options, workflow_logs)
      → Google Calendar (shadow hold creation on Brian's shared calendar)
  → Response displayed in Lovable SHADOW HOLD ANALYSIS section
```

### Environment Configuration:
- `VITE_WEBHOOK_URL`: Update in Lovable environment variables to point to n8n cloud webhook URL
- `ANTHROPIC_API_KEY`: Set in n8n cloud credentials (Anthropic Chat Model node)
- `SUPABASE_URL` + `SUPABASE_SERVICE_ROLE_KEY`: Set in n8n cloud credentials (HTTP Request nodes)
- Google Calendar OAuth2: Reconfigure in n8n cloud with Brian's Google account

### Cost Estimate:
| Component | Tier | Monthly Cost |
|---|---|---|
| Lovable frontend | Existing paid plan | $0 incremental |
| n8n cloud | Free tier | $0 |
| Supabase | Free tier | $0 |
| Anthropic API | Pay-per-use | ~$0.05-0.20 for 10-20 test runs |
| Google Calendar | Free API | $0 |
| **Total** | | **< $1** |

---

## deployment_plan_summary

**Delivery mechanism:** Lovable frontend (existing) posting to n8n cloud webhook (new — migrated from local Docker for stable URL)

**Key platforms:** Lovable (frontend, existing), n8n cloud (backend migration, free tier), Supabase (data, existing), Google Calendar (calendar integration, existing)

**What's staying manual for this deployment:**
- Brian monitors execution logs each morning
- Brian manually verifies TC8 fix before handing off
- No automated error alerting to tester
- Tester shares Brian's Google Calendar (no separate calendar credential for her account)

**Next step:** Execute the migration from local n8n Docker to n8n cloud, update the Lovable webhook URL, verify end-to-end with one test run, then hand off to tester for 1-week pilot
