# First Deployment Specifications
## Shadow Hold Assistant — Pilot Deployment

---

## deployment_context

**Why we're deploying:**
To validate whether Brian's wife (a professional EA) can use the Shadow Hold Assistant independently — without Brian present to guide her or verify outputs. The quality risk being tested is business hours boundary judgment: does the agent's output give her enough clarity to act confidently on a real scheduling decision without verifying the timezone math herself?

**Who's testing:**
Brian's wife — professional Executive Assistant at a Fortune 500 company. Non-technical user. Has bookmarked meet-propose-quick.lovable.app and used it once with Brian present. Committed to 1-2 real scheduling scenarios during her work week, with feedback via voice note or text.

**What makes this deployment successful (learning criteria, not metrics):**
- She acts on the agent's output without asking Brian to verify the timezone math
- If the business hours warning fires, she understands what it means and knows what to do
- The shadow hold appears on the shared Google Calendar correctly after form submission
- If something breaks, she tells Brian what happened (not just "it didn't work")

---

## infrastructure_specifications

### Backend Deployment

**Platform:** n8n cloud (migrate from local Docker)

**Why this migration:** The local Docker + Cloudflare quick tunnel setup requires Brian to manually start the tunnel before each testing session. The tunnel URL changes on restart, which would break the Lovable webhook URL mid-week. n8n cloud provides a stable, persistent webhook URL with no manual startup required.

**Migration steps:**
1. Export the current n8n workflow JSON from local instance:
   - In local n8n (localhost:5678), open the Shadow Hold workflow
   - Settings → Download as JSON → save as `shadow-hold-workflow.json`

2. Create n8n cloud account at n8n.io (free tier — up to 5 active workflows)

3. Import workflow to n8n cloud:
   - New workflow → Import from file → upload `shadow-hold-workflow.json`

4. Reconfigure credentials in n8n cloud (credentials don't export for security):
   - **Anthropic:** Add new credential → Anthropic → paste ANTHROPIC_API_KEY
   - **Supabase HTTP:** Update HTTP Request nodes with Supabase URL and service role key
   - **Google Calendar OAuth2:** Re-authenticate with Brian's Google account

5. Activate workflow in n8n cloud → copy the new production webhook URL

6. Update Lovable environment variable:
   - In Lovable project settings → Environment variables
   - Set `VITE_WEBHOOK_URL` = [new n8n cloud webhook URL]
   - Redeploy Lovable app (or trigger environment rebuild)

**Startup command:** None required — n8n cloud manages runtime automatically

**Environment variables to configure in n8n cloud credentials:**
| Variable | Where | Description |
|---|---|---|
| ANTHROPIC_API_KEY | Anthropic credential node | Claude claude-sonnet-4-6 access |
| SUPABASE_URL | HTTP Request nodes | `https://[project-id].supabase.co` |
| SUPABASE_SERVICE_ROLE_KEY | HTTP Request nodes | Service role key (bypasses RLS for writes) |
| Google OAuth2 | Google Calendar node | OAuth2 token for Brian's Google account |

**Health check:** Submit a test meeting proposal through the Lovable form after migration. Verify:
- n8n cloud execution log shows a successful run
- Supabase `meetings` table has a new row
- Google Calendar has a new shadow hold event
- Lovable UI shows the SHADOW HOLD ANALYSIS output

### Data Persistence

**Platform:** Supabase (existing project, no changes needed)
**Connection:** n8n cloud connects via HTTP Request nodes using Supabase REST API
**Tables in use:** meetings, time_options, workflow_logs
**Setup required:** None — tables already created and tested
**Backups:** None for v1 pilot — acceptable data loss risk for 1-week pilot with 1-2 test runs
**Explicit note:** If Supabase data is lost, the pilot can be re-run. No business-critical data at stake.

### Frontend Deployment

**Platform:** Lovable (existing, deployed at meet-propose-quick.lovable.app)
**Only change required:** Update VITE_WEBHOOK_URL environment variable to n8n cloud webhook URL (see Backend step 6 above)
**No code changes required** unless webhook URL update requires a redeploy

### Integration Configuration

**Google Calendar:**
- Shadow holds created on Brian's Google Calendar
- Brian shares this calendar with his wife so she can see the events
- OAuth2 token authenticated in n8n cloud with Brian's Google account
- Verify token is fresh before handing off (check expiry in Google Cloud Console)

**Anthropic API:**
- Claude claude-sonnet-4-6 (claude-sonnet-4-6)
- System prompt: final_prompt.txt (already in n8n workflow)
- Rate limits: Free tier allows ~1000 requests/day — sufficient for pilot

---

## access_and_monitoring

### Tester Access Instructions

Step-by-step for Brian's wife (written for a non-technical user):

1. Go to: **meet-propose-quick.lovable.app** (it's already in your bookmarks)
2. Fill in the meeting form:
   - Subject: the meeting name (e.g., "Client Strategy Call")
   - Participants: email addresses and their timezone/city
   - Proposed time: the UTC time you want to propose (or your local time + timezone)
   - Duration: how long the meeting will be
3. Click Submit
4. The SHADOW HOLD ANALYSIS will appear below the form in about 5-10 seconds
5. Read the output — especially the STATUS line (GOOD / WARNING / OUTSIDE HOURS)
6. If it looks right, the shadow hold has already been created on the shared calendar — check it there
7. If anything looks wrong or confusing, send Brian a voice note describing what happened

**Login:** No login required — the form is open access

### Quality Risk Monitoring

**Signal we're watching for:** Does she trust the business hours judgment output?
- Monitored by: Brian reviewing the n8n cloud execution log each morning
- What to look for: Execution succeeded → check the AI output in the log for business hours STATUS field
- Secondary signal: Her voice note feedback — does she mention verifying times, or does she act directly?

**Timezone accuracy monitoring:**
- Each execution log in n8n cloud shows the full Claude output
- Brian reviews the participant local times in each run against the submitted UTC time
- No automated monitoring — manual log review each morning (~5 minutes)

### Error Visibility

**Where errors appear:**
- n8n cloud execution history → shows failed runs with error messages
- Supabase workflow_logs table → execution metadata for each run
- Brian checks n8n cloud execution log each morning

**Tester notification:** None automated. If she experiences a problem (form submits but nothing happens, error message appears), she sends Brian a voice note describing what she saw. Brian diagnoses from execution log.

### Feedback Collection

**Method:** Voice notes or texts to Brian (no formal feedback form for this pilot)

**Questions to prompt (Brian shares with her before pilot starts):**
1. Did you trust the timezone times in the output, or did you check them manually?
2. Was the STATUS line (GOOD / WARNING / OUTSIDE HOURS) clear? Did you know what to do with it?
3. Did the shadow hold appear on the shared calendar as expected?
4. Anything that seemed wrong, confusing, or surprising?

**Feedback timing:** Same day or next morning after each use

---

## deployment_checklist

### Independence Verification
- [ ] n8n cloud workflow is active (green status in n8n cloud dashboard)
- [ ] Webhook URL is stable (does not change on restart — n8n cloud guarantee)
- [ ] Lovable VITE_WEBHOOK_URL updated to n8n cloud webhook URL
- [ ] Lovable app redeployed with new environment variable
- [ ] Google Calendar OAuth2 token verified fresh (not expired)
- [ ] n8n cloud credentials configured for Anthropic, Supabase, Google Calendar
- [ ] No localhost references in workflow nodes (all URLs point to cloud services)

### Remote Access Verification
- [ ] Brian submits a test form from his phone (different network than development machine) — verifies the form reaches n8n cloud
- [ ] Brian's wife submits a test form from her work device — verifies her access
- [ ] Check n8n cloud execution log confirms both test runs succeeded
- [ ] Verify shadow hold appears on shared Google Calendar from Brian's test run

### Quality Signal Verification
- [ ] Run TC8 scenario (4-party global meeting with NYC at ~4am) against n8n cloud workflow — verify business hours WARNING or OUTSIDE HOURS fires correctly after system prompt fix
- [ ] Check that n8n cloud execution log shows Claude's full output (confirms observability)
- [ ] Verify Supabase workflow_logs table receiving entries (confirms data pipeline intact)

### Rollback Plan
- If critical bug found during pilot:
  1. In Lovable environment variables, revert VITE_WEBHOOK_URL to local n8n webhook URL (if local Docker is running) OR
  2. Deactivate the n8n cloud workflow (stops processing new submissions)
  3. Send wife a quick message: "Pausing the tool for a fix — will be back in [timeframe]"
- Recovery time: < 15 minutes to deactivate or switch back to local

---

## known_limitations

The following are intentionally staying manual for this deployment. These are not technical debt — they are appropriate scope for a 1-user, 1-week pilot:

1. **Brian monitors execution logs manually each morning.** No automated alerting for failures. If the tool breaks and she doesn't send a voice note, Brian may not know until his morning check.

2. **All calendar events created on Brian's calendar.** Brian's wife views them on the shared calendar. She does not have direct Google Calendar write access through the tool for this pilot. A future version would create events on her executive's calendar directly.

3. **No error message to the tester if the webhook fails.** If the n8n cloud workflow errors, the Lovable form may show a generic error or nothing. Tester sends Brian a voice note and he diagnoses.

4. **No data persistence for tester-specific sessions.** All Supabase rows are in Brian's account. No per-user tracking. Acceptable for single tester.

5. **TC8 business hours fix unverified in production.** The system prompt update was made but TC8 has not been re-run against the cloud deployment. This is on the deployment checklist to verify before tester hand-off.

6. **Cloudflare tunnel dependency eliminated, but n8n cloud free tier has limits.** Free tier allows 5 active workflows and ~2,500 workflow executions/month. Vastly more than needed for this pilot.
