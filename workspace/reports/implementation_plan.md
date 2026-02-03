# Implementation Plan: Shadow Hold Assistant
## Hybrid Approach: Progressive Enhancement → Stakeholder Self-Service

**Selected Strategy:** Start simple (Sprint 1), add core differentiator (Sprint 2), polish with AI (Sprint 3)

**Target:** Multi-stakeholder consensus finder with one-click confirmations (Outlook integration)

---

## Calendar Abstraction Strategy

**Decision:** Build multi-calendar support from day one using provider abstraction pattern.

### Why This Approach

✅ **Start with Google Calendar** (Sprint 1)
- Easier API, faster development
- You can test immediately with your personal calendar
- Broader user testing pool

✅ **Add Outlook** (Sprint 2)
- Once foundation is proven
- When wife validates the concept
- More impressive final demo

✅ **Abstraction Benefits**
- Write business logic once, works with any calendar
- Easy to add more calendars later (Apple, Outlook, etc.)
- Professional architecture (demonstrates engineering skill)

### Calendar Provider Interface

All calendar operations go through a standard interface:

```javascript
// Pseudocode: Calendar Provider Interface
interface CalendarProvider {
  // Authentication
  authenticate(credentials) → token

  // Shadow hold operations
  createEvent(datetime, title, tentative=true) → event_id
  updateEvent(event_id, changes) → success
  deleteEvent(event_id) → success
  convertToConfirmed(event_id, attendees) → success

  // Query operations
  getEvents(start_date, end_date) → events[]
  checkAvailability(datetime, duration) → boolean
}
```

### Sprint 1: Google Calendar Provider

```javascript
// Implement GoogleCalendarProvider
class GoogleCalendarProvider implements CalendarProvider {
  authenticate(credentials) {
    // OAuth 2.0 flow with Google
    // Return access token
  }

  createEvent(datetime, title, tentative=true) {
    // Call Google Calendar API
    // POST /calendars/primary/events
    // Set transparency: "transparent" for tentative
    return event_id;
  }

  // ... implement other methods
}
```

### Sprint 2: Outlook Provider

```javascript
// Add OutlookCalendarProvider
class OutlookCalendarProvider implements CalendarProvider {
  authenticate(credentials) {
    // Microsoft Graph OAuth
    // Return access token
  }

  createEvent(datetime, title, tentative=true) {
    // Call Microsoft Graph API
    // POST /me/events
    // Set showAs: "tentative"
    return event_id;
  }

  // ... implement other methods
}
```

### Database Support

```sql
-- Store which calendar provider each EA uses
CREATE TABLE calendar_providers (
  id UUID PRIMARY KEY,
  ea_id UUID,
  provider_type TEXT, -- 'google' or 'outlook'
  credentials_encrypted TEXT, -- Encrypted OAuth tokens
  is_primary BOOLEAN,
  created_at TIMESTAMP
);

-- Generic event_id field (works with any calendar)
CREATE TABLE time_options (
  id UUID PRIMARY KEY,
  meeting_id UUID,
  datetime TIMESTAMP,
  calendar_event_id TEXT, -- Can be Google or Outlook event ID
  calendar_provider_id UUID REFERENCES calendar_providers(id),
  created_at TIMESTAMP
);
```

### n8n Implementation

In n8n, you'll use HTTP Request nodes that call your calendar provider abstraction:

**Workflow 1: Create Shadow Holds**
```
1. Webhook receives meeting proposal
2. Query: Which calendar provider does this EA use?
3. Switch node: Branch based on provider_type
   - If 'google' → Call Google Calendar API node
   - If 'outlook' → Call Microsoft Graph API node
4. Store calendar_event_id in database
5. Return success
```

**Sprint 1:** Only implement Google branch
**Sprint 2:** Add Outlook branch

---

## Sprint 1 MVP: Manual Foundation (Week 2 - Due Feb 8)

### Goal
Get shadow holds working end-to-end with manual workflow using **Google Calendar**. Build abstraction layer so Outlook can be added in Sprint 2.

### User Flow (Sprint 1)
1. EA opens dashboard and enters:
   - Meeting subject
   - 3-4 proposed time slots
   - List of stakeholders (names + emails)
2. System creates shadow holds in Outlook (all proposed times)
3. System shows EA a tracking dashboard
4. **EA manually sends email** to stakeholders (copy/paste time options)
5. **EA manually marks** which option stakeholders chose in dashboard
6. **EA manually clicks "Finalize"** when consensus is reached
7. System converts chosen shadow hold → real Outlook meeting
8. System deletes other shadow holds

### Features (Sprint 1)
- ✅ Simple web form to create meeting proposal
- ✅ Outlook API integration (create tentative events)
- ✅ Database to track meeting proposals and time options
- ✅ EA dashboard showing all pending meetings
- ✅ Manual tracking: EA clicks "Stakeholder X confirmed Option 2"
- ✅ One-click finalization: Convert shadow hold → meeting invite
- ❌ No automated emails (EA does this manually)
- ❌ No stakeholder interface (EA tracks via dashboard)
- ❌ No AI features

### Technical Stack (Sprint 1)
- **Frontend:** Lovable (EA dashboard only)
  - Form: Create meeting proposal
  - Dashboard: List of pending meetings with status
  - Manual tracking: Click to mark stakeholder responses
- **Backend:** n8n (2 workflows)
  - Workflow 1: Create shadow holds via Calendar Provider
  - Workflow 2: Finalize meeting (convert + cleanup)
- **Database:** Supabase
  - `meetings` table
  - `time_options` table (with `calendar_event_id` - generic field)
  - `stakeholders` table
  - `responses` table (manually populated by EA)
  - `calendar_providers` table (stores which calendar each EA uses)
- **Calendar:** Google Calendar API (Sprint 1) + Outlook API (Sprint 2)
  - **Abstraction Layer:** Calendar Provider Interface
  - **Sprint 1 Implementation:** Google Calendar provider
  - **Sprint 2 Addition:** Outlook provider
  - Operations: Create tentative events, convert to confirmed, delete unused

### Success Metrics (Sprint 1)
- Shadow holds appear in EA's Outlook calendar
- EA can manually track responses via dashboard
- System successfully converts chosen option to meeting
- System cleans up unused shadow holds
- **Time Savings:** ~2-3 min per meeting (shadow hold creation + cleanup automation)
- **Target:** 20-30% time reduction

### Learning Goals (Sprint 1)
- ✅ GitHub basics: commits, branches, push/pull
- ✅ Outlook API authentication and event creation
- ✅ n8n workflow basics (trigger → action → response)
- ✅ Supabase database setup and queries
- ✅ Lovable frontend basics (forms, tables, buttons)

### Deliverables (Sprint 1 - Feb 8)
- [ ] GitHub repository created and initialized
- [ ] Outlook API credentials configured
- [ ] Supabase database schema created
- [ ] EA dashboard deployed (Lovable)
- [ ] n8n workflows: create shadow holds + finalize meeting
- [ ] Working demo: EA can create proposal → shadow holds appear in Outlook
- [ ] Demo video recorded for bootcamp check-in

---

## Sprint 2: Stakeholder Self-Service + Outlook Support (Week 4 - Due Feb 22)

### Goal
1. Add the core differentiator: stakeholders click buttons instead of EA manually tracking responses
2. Add Outlook calendar provider to support wife's work calendar

### New User Flow (Sprint 2)
1. EA creates proposal (same as Sprint 1)
2. System creates shadow holds (same as Sprint 1)
3. **NEW:** System automatically sends emails to stakeholders with confirmation buttons
4. **NEW:** Stakeholders click buttons → system records responses in database
5. **NEW:** EA dashboard updates in real-time as stakeholders respond
6. **NEW:** System auto-detects consensus (all stakeholders chose same time)
7. **NEW:** System auto-finalizes when consensus reached (or EA manually finalizes if no consensus)

### New Features (Sprint 2)
- ✅ Automated email sending with embedded confirmation buttons
- ✅ Stakeholder response page (mobile-friendly)
- ✅ Real-time dashboard updates (EA sees responses as they happen)
- ✅ Consensus detection logic (find time slot with 100% agreement)
- ✅ Automatic finalization when consensus reached
- ✅ Email notifications to EA when consensus detected
- ✅ **Outlook Calendar Provider** (add Microsoft Graph API support)
- ✅ **Calendar Provider Selection** (EA can choose Google or Outlook in settings)
- ❌ No AI email drafting yet (templated emails only)

### Technical Additions (Sprint 2)
- **Frontend Additions (Lovable):**
  - Stakeholder response page: `/respond?token=xyz`
  - Real-time dashboard (polling or websockets)
  - Visual indicators: "3/5 responded, Option 2 has consensus ✓"
- **Backend Additions (n8n):**
  - Workflow 3: Send stakeholder emails with unique response links
  - Workflow 4: Handle stakeholder button clicks → record in DB
  - Workflow 5: Check for consensus after each response
  - Workflow 6: Auto-finalize when consensus detected
- **Database Additions (Supabase):**
  - Add `response_tokens` table (secure unique links)
  - Update `responses` table with clicked_at timestamp
  - Add consensus status to `meetings` table
- **Email:** SendGrid or Outlook SMTP
  - Templated email with HTML buttons
  - Links: `https://yourapp.com/respond?token=abc123&option=1`

### Multi-Stakeholder Consensus Logic
```javascript
// After each stakeholder response, check for consensus
function checkConsensus(meetingId) {
  const meeting = getMeeting(meetingId);
  const timeOptions = getTimeOptions(meetingId);
  const stakeholders = getStakeholders(meetingId);

  for (const option of timeOptions) {
    const responses = getResponses(option.id);

    // Did ALL stakeholders confirm this specific time?
    if (responses.length === stakeholders.length) {
      // CONSENSUS FOUND!
      meeting.status = 'consensus_reached';
      meeting.selected_option_id = option.id;
      autoFinalize(option.id); // Convert shadow hold → meeting
      notifyEA(meetingId); // Email EA: "Consensus reached!"
      return true;
    }
  }

  // No consensus yet - check if everyone responded
  const totalResponses = countAllResponses(meetingId);
  if (totalResponses === stakeholders.length * timeOptions.length) {
    // Everyone responded but no consensus
    meeting.status = 'no_consensus';
    notifyEA(meetingId); // Email EA: "No consensus, manual intervention needed"
  }

  return false;
}
```

### Success Metrics (Sprint 2)
- Stakeholders receive emails with working buttons
- Stakeholder clicks are recorded in database
- EA dashboard updates in real-time (or near real-time)
- Consensus detection works correctly (tested with 3+ stakeholders)
- Automatic finalization triggers when consensus reached
- **Time Savings:** ~10-12 min per meeting (no manual email, no manual tracking, faster stakeholder responses)
- **Target:** 60-70% time reduction

### Learning Goals (Sprint 2)
- ✅ Email automation and deliverability
- ✅ URL token generation and security
- ✅ Real-time updates (polling vs. websockets)
- ✅ Complex n8n workflows with branching logic
- ✅ Database queries for aggregation (consensus detection)
- ✅ **Microsoft Graph API** (Outlook integration)
- ✅ **Multi-provider architecture** (abstraction patterns)

### Deliverables (Sprint 2 - Feb 22)
- [ ] Stakeholder response page deployed
- [ ] Automated email sending working
- [ ] Real-time dashboard updates implemented
- [ ] Consensus detection logic tested with multiple scenarios
- [ ] Auto-finalization working end-to-end
- [ ] **Outlook Calendar Provider implemented and tested**
- [ ] **Calendar provider selection UI** (EA can switch between Google/Outlook)
- [ ] User testing with wife: 3-5 real scheduling scenarios (using her Outlook calendar)
- [ ] User testing with others: 3-5 scenarios (using Google Calendar)
- [ ] Feedback document: What works, what needs improvement
- [ ] Demo video for bootcamp check-in (showcasing both calendar providers)

---

## Sprint 3: AI Enhancement + Deployment (Week 6 - Due Mar 8)

### Goal
Add AI email drafting, polish UX, deploy to production, and prepare for final demo.

### New User Flow (Sprint 3)
1. EA creates proposal (same as Sprint 2)
2. **NEW:** AI drafts the stakeholder email based on meeting context
3. **NEW:** EA reviews and edits AI-drafted email before sending
4. System sends customized email to stakeholders (rest of flow same as Sprint 2)

### New Features (Sprint 3)
- ✅ AI email drafting (Claude API)
- ✅ EA approval workflow for AI-drafted emails
- ✅ Stakeholder context/preferences (tone, formality)
- ✅ Email templates library (EA can choose starting point)
- ✅ Analytics dashboard (time saved, response rates, consensus rates)
- ✅ Mobile-optimized stakeholder response page
- ✅ Production deployment (Vercel + Supabase production tier)
- ✅ Error handling and logging
- ✅ Cost monitoring dashboard

### Technical Additions (Sprint 3)
- **AI Integration:**
  - Claude API for email drafting
  - Prompt engineering: Include meeting context, stakeholder info, tone preferences
  - EA approval interface: edit AI-drafted email before sending
- **Frontend Polish (Lovable):**
  - Email preview and edit interface
  - Analytics dashboard (charts, metrics)
  - Mobile responsive design for stakeholder page
- **Backend Additions (n8n):**
  - Workflow 7: AI email drafting
  - Error handling and retry logic for all workflows
  - Logging and monitoring
- **Database Additions (Supabase):**
  - `stakeholder_preferences` table (tone, timezone, etc.)
  - `email_templates` table
  - `analytics_events` table (track time saved, response times)
- **Deployment:**
  - Lovable frontend → Vercel
  - n8n → Cloud hosting (n8n.cloud or self-hosted)
  - Supabase → Production tier
  - Domain name and SSL certificate

### AI Email Drafting Prompt Example
```
You are drafting a meeting scheduling email on behalf of an Executive Assistant.

Context:
- Meeting subject: {subject}
- Participants: {participant_names}
- Proposed times: {time_options}
- Meeting duration: {duration}
- EA's name: {ea_name}
- Executive's name: {executive_name}

Stakeholder preferences:
- {stakeholder_name}: {tone_preference}, {relationship_context}

Draft a professional email that:
1. Proposes the time options clearly
2. Includes clickable confirmation buttons (we'll add these after)
3. Matches the appropriate tone for each stakeholder
4. Is concise (3-4 sentences max)

Email:
```

### Success Metrics (Sprint 3)
- AI-drafted emails are accurate and professional (90%+ approval rate from EA)
- EA editing time < 30 seconds per email
- System is stable and handles errors gracefully
- Production deployment is accessible to real users
- Cost per scheduling cycle < $0.10
- **Time Savings:** ~12-15 min per meeting (email drafting + tracking + faster cycle time)
- **Target:** 70-80% time reduction (exceeds goal!)

### Learning Goals (Sprint 3)
- ✅ AI API integration (Claude)
- ✅ Prompt engineering for real-world use case
- ✅ Production deployment pipeline
- ✅ Cost monitoring and optimization
- ✅ User analytics and metrics
- ✅ Full stack debugging in production

### Deliverables (Sprint 3 - Mar 8)
- [ ] AI email drafting integrated and tested
- [ ] EA approval workflow implemented
- [ ] Production deployment completed
- [ ] Domain name configured (shadowhold.app or similar)
- [ ] Cost monitoring dashboard showing per-meeting costs
- [ ] Analytics dashboard showing time savings metrics
- [ ] User testing with wife: 10-15 real meetings scheduled through system
- [ ] Final demo script and presentation prepared
- [ ] Demo video recorded
- [ ] Bootcamp final demo (Mar 13) ready

---

## Final Demo (Week 7 - Mar 13)

### Demo Script (8 minutes)

**Slide 1: The Problem (1 min)**
- EAs spend 3-6 hours/week on scheduling coordination
- Manual shadow holds, email tracking, double-booking risks
- My wife's pain point: "I spend more time coordinating meetings than supporting executives"

**Slide 2: The Solution (1 min)**
- Shadow Hold Assistant: AI-powered multi-stakeholder consensus finder
- One-click confirmations instead of email ping-pong
- Automatic shadow holds in Outlook

**Slide 3: Live Demo (4 min)**
- **Step 1 (30 sec):** EA creates meeting proposal
  - Show form: subject, 3 time options, 5 stakeholders
  - Click "Create Proposal"
- **Step 2 (30 sec):** AI drafts email
  - Show AI-generated email with professional tone
  - EA makes quick edit
  - Click "Approve and Send"
- **Step 3 (1 min):** Stakeholder perspective
  - Open stakeholder email on phone (mobile view)
  - Click confirmation button for Option 2
  - Show instant dashboard update
- **Step 4 (1 min):** Consensus detection
  - Simulate 3 more stakeholders clicking Option 2
  - Show dashboard: "Consensus reached!"
  - Show Outlook: Shadow hold converted to real meeting
- **Step 5 (1 min):** Analytics
  - Show time saved: 12 minutes per meeting
  - Show metrics: 15 meetings scheduled, 70% time reduction

**Slide 4: Technical Architecture (1 min)**
- n8n orchestration (show workflow diagram)
- Outlook API integration
- Supabase database
- Claude AI for email drafting
- Lovable frontend
- Deployed on Vercel

**Slide 5: Results & Learning (1 min)**
- Successfully deployed and used by real user (wife)
- 70% time reduction validated with real data
- Learned: GitHub, n8n, Outlook API, AI integration, deployment pipeline
- Next steps: Add Google Calendar support, multi-timezone handling, preference learning

---

## Architecture Overview (All Sprints)

### System Components

```
┌─────────────────┐
│   EA Dashboard  │ (Lovable → Vercel)
│   - Create      │
│   - Track       │
│   - Approve     │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│   n8n Cloud     │ (Workflow Orchestration)
│   - Create      │
│   - Email       │
│   - Track       │
│   - Finalize    │
└────┬──────┬─────┘
     │      │
     ↓      ↓
┌──────┐  ┌──────────┐
│Outlook│  │ Supabase │
│  API  │  │    DB    │
└──────┘  └──────────┘
     ↑
     │
┌────────────────────┐
│ Stakeholder Page   │ (Lovable → Vercel)
│ - View options     │
│ - Click confirm    │
└────────────────────┘
```

### Database Schema (Final)

```sql
-- Meetings table
CREATE TABLE meetings (
  id UUID PRIMARY KEY,
  ea_id UUID,
  subject TEXT,
  duration_minutes INTEGER,
  status TEXT, -- 'pending', 'consensus_reached', 'no_consensus', 'finalized'
  selected_option_id UUID,
  created_at TIMESTAMP,
  finalized_at TIMESTAMP
);

-- Time options table
CREATE TABLE time_options (
  id UUID PRIMARY KEY,
  meeting_id UUID REFERENCES meetings(id),
  datetime TIMESTAMP,
  outlook_event_id TEXT, -- Shadow hold event ID
  created_at TIMESTAMP
);

-- Stakeholders table
CREATE TABLE stakeholders (
  id UUID PRIMARY KEY,
  meeting_id UUID REFERENCES meetings(id),
  name TEXT,
  email TEXT,
  tone_preference TEXT, -- 'formal', 'casual', 'warm'
  relationship_context TEXT -- Optional context for AI
);

-- Response tokens table (security)
CREATE TABLE response_tokens (
  id UUID PRIMARY KEY,
  stakeholder_id UUID REFERENCES stakeholders(id),
  token TEXT UNIQUE, -- Cryptographically secure random string
  expires_at TIMESTAMP,
  used_at TIMESTAMP
);

-- Responses table
CREATE TABLE responses (
  id UUID PRIMARY KEY,
  stakeholder_id UUID REFERENCES stakeholders(id),
  time_option_id UUID REFERENCES time_options(id),
  confirmed BOOLEAN,
  clicked_at TIMESTAMP
);

-- Email templates table (Sprint 3)
CREATE TABLE email_templates (
  id UUID PRIMARY KEY,
  ea_id UUID,
  name TEXT,
  template_text TEXT,
  created_at TIMESTAMP
);

-- Analytics events table (Sprint 3)
CREATE TABLE analytics_events (
  id UUID PRIMARY KEY,
  meeting_id UUID REFERENCES meetings(id),
  event_type TEXT, -- 'created', 'email_sent', 'stakeholder_responded', 'consensus_reached', 'finalized'
  timestamp TIMESTAMP,
  metadata JSONB
);
```

### n8n Workflows (Final)

**Workflow 1: Create Meeting Proposal**
- Trigger: HTTP POST from Lovable frontend
- Action 1: Create meeting record in Supabase
- Action 2: For each time option, call Outlook API to create tentative event
- Action 3: Store outlook_event_id in time_options table
- Response: Return meeting_id to frontend

**Workflow 2: Send Stakeholder Emails (Sprint 2+)**
- Trigger: HTTP POST (after EA approves in Sprint 3, or auto in Sprint 2)
- Action 1: For each stakeholder, generate secure token
- Action 2: Draft email (template in Sprint 2, AI in Sprint 3)
- Action 3: Send email via SendGrid/Outlook
- Action 4: Log analytics event

**Workflow 3: Handle Stakeholder Response (Sprint 2+)**
- Trigger: HTTP POST from stakeholder response page
- Action 1: Validate token
- Action 2: Record response in database
- Action 3: Call Workflow 5 (check consensus)
- Response: "Thank you, your response has been recorded"

**Workflow 4: Check Consensus (Sprint 2+)**
- Trigger: Called by Workflow 3
- Action 1: Query database for all responses to this meeting
- Action 2: For each time option, count stakeholder confirmations
- Action 3: If option has 100% confirmation → Call Workflow 6 (auto-finalize)
- Action 4: If all stakeholders responded but no consensus → Notify EA

**Workflow 5: Finalize Meeting**
- Trigger: HTTP POST (manual in Sprint 1, auto in Sprint 2+)
- Action 1: Get selected time_option_id
- Action 2: Call Outlook API to convert tentative event → confirmed event
- Action 3: Add all stakeholders as meeting attendees
- Action 4: For each unused time option, delete Outlook event
- Action 5: Update meeting status to 'finalized'
- Action 6: Log analytics event

**Workflow 6: AI Email Drafting (Sprint 3)**
- Trigger: HTTP POST from frontend (EA clicks "Draft Email")
- Action 1: Gather context (meeting details, stakeholder preferences)
- Action 2: Call Claude API with prompt
- Action 3: Return drafted email to frontend for EA approval
- Response: AI-drafted email text

---

## Sprint-by-Sprint Feature Matrix

| Feature | Sprint 1 | Sprint 2 | Sprint 3 |
|---------|----------|----------|----------|
| **Shadow Holds** | ✅ Manual creation | ✅ Automatic | ✅ Automatic |
| **EA Dashboard** | ✅ Basic | ✅ Real-time | ✅ Analytics added |
| **Email to Stakeholders** | ❌ Manual | ✅ Automated (template) | ✅ AI-drafted |
| **Stakeholder Response** | ❌ EA tracks manually | ✅ One-click buttons | ✅ Mobile-optimized |
| **Consensus Detection** | ❌ Manual | ✅ Automatic | ✅ Automatic |
| **Finalization** | ✅ Manual trigger | ✅ Auto + manual | ✅ Auto + manual |
| **Outlook Integration** | ✅ Basic | ✅ Full CRUD | ✅ Full CRUD |
| **AI Features** | ❌ None | ❌ None | ✅ Email drafting |
| **Deployment** | 🟡 Dev only | 🟡 Staging | ✅ Production |

---

## Risk Mitigation

### Technical Risks

**Risk 1: Outlook API authentication complexity**
- *Mitigation:* Start with personal Microsoft account for Sprint 1
- *Fallback:* If enterprise OAuth fails, use service account approach
- *Timeline:* Resolve by end of Week 1 (Sprint 1)

**Risk 2: n8n learning curve**
- *Mitigation:* Start with simple 2-workflow MVP in Sprint 1
- *Fallback:* If n8n is too complex, use simple Node.js Express server
- *Timeline:* Assess by mid-Sprint 1

**Risk 3: Real-time dashboard updates**
- *Mitigation:* Use simple polling (every 5 sec) for Sprint 2 MVP
- *Fallback:* If latency is issue, upgrade to websockets in Sprint 3
- *Timeline:* Implement polling first, optimize later

**Risk 4: Email deliverability (spam filters)**
- *Mitigation:* Use Outlook SMTP with proper SPF/DKIM records
- *Fallback:* If emails go to spam, use SendGrid with verified domain
- *Timeline:* Test in Sprint 2 Week 1

**Risk 5: AI email quality**
- *Mitigation:* EA approval workflow (human-in-the-loop)
- *Fallback:* EA can always edit or use templates instead of AI
- *Timeline:* Sprint 3, but not blocking for Sprint 2 demo

### Schedule Risks

**Risk 6: Sprint 1 takes longer than 2 weeks**
- *Mitigation:* Cut Sprint 1 scope to absolute minimum (manual tracking is OK)
- *Fallback:* Push stakeholder buttons to Sprint 3, keep Sprint 2 for polish
- *Timeline:* Re-assess by Feb 4 (mid-Sprint 1)

**Risk 7: Bootcamp coursework takes more time than expected**
- *Mitigation:* Prioritize bootcamp assignments over polish features
- *Fallback:* Sprint 3 becomes "deployment + demo prep" only (skip AI)
- *Timeline:* Weekly check-ins

---

## Cost Estimate

### Sprint 1 (MVP)
- Lovable: $0 (free tier)
- Supabase: $0 (free tier)
- n8n: $0 (free tier, 5,000 workflow executions/month)
- Outlook API: $0 (included with Microsoft 365)
- **Total: $0/month**

### Sprint 2 (Stakeholder Self-Service)
- Lovable: $0 (still within free tier)
- Supabase: $0 (still within free tier)
- n8n: $0 (still within free tier)
- SendGrid: $0 (free tier, 100 emails/day)
- **Total: $0/month**

### Sprint 3 (Production)
- Lovable → Vercel: $20/month (Hobby plan)
- Supabase: $25/month (Pro plan for production)
- n8n: $0 (free tier should still suffice) or $20/month if needed
- Claude API: ~$5/month (estimated 100 email drafts @ $0.05 each)
- Domain name: $12/year (~$1/month)
- **Total: $51-71/month** (production)

**Bootcamp Budget:** Stay on free tiers for Sprints 1-2, only upgrade for Sprint 3 production deployment if needed.

---

## GitHub Learning Integration

Since you currently know nothing about GitHub, here's how Git fits into each sprint:

### Sprint 1 GitHub Tasks
- [ ] Create GitHub account
- [ ] Install Git on local machine
- [ ] Initialize repository: `git init`
- [ ] First commit: `git commit -m "Initial commit"`
- [ ] Create GitHub repository online
- [ ] Push to GitHub: `git push origin main`
- [ ] Make daily commits (minimum 1 per day)
- [ ] Write clear commit messages: "Add Outlook API integration" not "fixed stuff"

### Sprint 2 GitHub Tasks
- [ ] Create feature branch: `git checkout -b feature/stakeholder-responses`
- [ ] Commit changes to branch
- [ ] Push branch to GitHub
- [ ] Create Pull Request (PR)
- [ ] Merge PR to main
- [ ] Learn: branches, PRs, merging

### Sprint 3 GitHub Tasks
- [ ] Set up CI/CD with GitHub Actions (optional)
- [ ] Use GitHub Issues to track bugs
- [ ] Tag release versions: `v1.0.0`
- [ ] Write README.md with project documentation
- [ ] Explore other bootcamp participants' repos
- [ ] Star and learn from relevant open-source projects

---

## Success Metrics (Overall)

### Quantitative Metrics
- **Time Saved:** 70% reduction in scheduling coordination time (target: 3-6 hrs → 1-2 hrs/week)
- **Cycle Time:** Reduce scheduling from 24-48 hours to 4-6 hours
- **Adoption:** Wife uses system for 100% of scheduling requests by Sprint 3
- **Reliability:** 95%+ success rate (meetings finalized without manual intervention)
- **Cost:** < $0.25 per meeting scheduled

### Qualitative Metrics
- **User Satisfaction:** Wife reports reduced stress and cognitive load
- **Technical Confidence:** Brian can now build AI agents independently
- **Bootcamp Goals:** Mastered GitHub, n8n, full-stack deployment

---

## Next Steps (This Week - Starting with Google Calendar)

### Immediate Actions (Today - Sunday Feb 2)

**1. Set up GitHub** (1 hour)
- Create GitHub account: https://github.com/signup
- Install Git: `brew install git` (Mac) or download from git-scm.com
- Configure Git:
  ```bash
  git config --global user.name "Brian Sullivan"
  git config --global user.email "your.email@example.com"
  ```
- Initialize repo in your project folder:
  ```bash
  cd /path/to/shadow-hold-assistant
  git init
  git add .
  git commit -m "Initial commit: Ideation documents"
  ```
- Create GitHub repository (via website)
- Push to GitHub:
  ```bash
  git remote add origin https://github.com/yourusername/shadow-hold-assistant.git
  git branch -M main
  git push -u origin main
  ```

**2. Set up Google Calendar API** (1 hour)
- Go to Google Cloud Console: https://console.cloud.google.com
- Create new project: "Shadow Hold Assistant"
- Enable Google Calendar API
- Create OAuth 2.0 credentials (Desktop app)
- Download credentials JSON file
- Save in project folder (DO NOT commit to Git - add to .gitignore)

**3. Set up development accounts** (30 min)
- Lovable: https://lovable.dev (sign up)
- Supabase: https://supabase.com (sign up with GitHub)
- n8n: https://n8n.io/cloud (sign up, free tier)

### Monday-Tuesday (Feb 3-4) - Build EA Dashboard

**Day 1: Lovable Frontend** (2-3 hours)
1. Create new Lovable project
2. Build simple form with fields:
   - Meeting subject (text input)
   - Time option 1, 2, 3 (datetime pickers)
   - Stakeholder emails (comma-separated text input)
3. Add "Create Proposal" button
4. Build dashboard page showing list of meetings
5. Deploy to Lovable hosting

**Day 2: Supabase Database** (2 hours)
1. Create Supabase project
2. Create tables:
   ```sql
   CREATE TABLE meetings (
     id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
     subject TEXT NOT NULL,
     status TEXT DEFAULT 'pending',
     created_at TIMESTAMP DEFAULT now()
   );

   CREATE TABLE time_options (
     id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
     meeting_id UUID REFERENCES meetings(id),
     datetime TIMESTAMP NOT NULL,
     calendar_event_id TEXT,
     calendar_provider TEXT DEFAULT 'google',
     created_at TIMESTAMP DEFAULT now()
   );

   CREATE TABLE stakeholders (
     id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
     meeting_id UUID REFERENCES meetings(id),
     name TEXT,
     email TEXT NOT NULL
   );
   ```
3. Get Supabase API key
4. Test connection from Lovable

### Wednesday-Thursday (Feb 5-6) - n8n + Google Calendar

**Day 3: n8n Workflow Setup** (2-3 hours)
1. Create n8n account (cloud version)
2. Create Workflow 1: "Create Shadow Holds"
   - Trigger: Webhook (POST request)
   - Node 1: Parse incoming JSON (meeting details)
   - Node 2: For each time option...
   - Node 3: Call Google Calendar API (create event)
   - Node 4: Store event_id in Supabase
   - Node 5: Return success response
3. Test with Postman or curl

**Day 4: Connect Everything** (2-3 hours)
1. Update Lovable form to call n8n webhook
2. Add Google Calendar authentication to n8n
3. Test end-to-end:
   - Fill out form in Lovable
   - Submit
   - Check Google Calendar for tentative events
   - Check Supabase for records
4. Debug any issues

### Weekend (Feb 8-9) - Polish and Demo

**Saturday: Manual Finalization** (2-3 hours)
1. Build dashboard page in Lovable
2. Show list of pending meetings
3. Add button: "Finalize Meeting"
4. Create n8n Workflow 2: "Finalize Meeting"
   - Convert selected shadow hold to confirmed event
   - Delete other shadow holds
   - Update database status
5. Test complete cycle

**Sunday: Demo and Commit** (1-2 hours)
1. Record demo video:
   - Create meeting proposal
   - Show shadow holds in Google Calendar
   - Manually track responses in dashboard
   - Finalize meeting
   - Show cleanup
2. Commit all code to GitHub
3. Push to GitHub
4. Share demo in bootcamp Slack

### This Weekend (Feb 2-3)
- Complete Google Calendar API setup
- Test authentication and basic event creation
- Get first shadow hold appearing in YOUR Google Calendar
- Commit progress to GitHub

---

## Summary: Why This Hybrid Approach Works

✅ **Progressive:** Start simple, add complexity incrementally
✅ **De-risked:** Working MVP at each sprint (not all-or-nothing)
✅ **Learning-focused:** Match bootcamp sprint structure
✅ **User-validated:** Real user feedback after Sprint 1 and Sprint 2
✅ **Goal-achieving:** Reaches 70% time savings by Sprint 3
✅ **Demo-ready:** Impressive final demo with AI + automation
✅ **n8n depth:** Three sprints of progressive n8n learning
✅ **GitHub integration:** Git workflow learned incrementally

You get the **fast start** of Approach #5 with the **powerful end goal** of Approach #3.

Ready to start Sprint 1?
