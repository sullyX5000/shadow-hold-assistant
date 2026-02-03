# Problem Definition: Intelligent Shadow Hold Assistant

## Problem Statement

Executive assistants spend excessive time manually managing calendar coordination - creating tentative holds, tracking responses, and preventing double-bookings - when this process could be largely automated with AI-powered intelligent shadow hold management.

**Gap:** EAs currently spend 60-70% of their scheduling time on manual coordination tasks that an AI agent could handle automatically.

## Target Users

**Primary User:** Executive Assistants in business environments
- **Specific Context:** EAs managing complex calendars for executives with frequent meeting requests
- **Technical Level:** Comfortable with standard productivity tools (Outlook, Google Calendar, email)
- **Specific Needs:**
  - Visual tracking of pending vs confirmed meetings
  - Prevention of double-booking during confirmation wait periods
  - Reduction of email composition time
  - Automated response monitoring

**Test User:** Brian's wife (Executive Assistant) who currently uses Cabinet

## Current State

### User Process

**Step-by-step current workflow:**

1. **Request Received:** EA receives scheduling request via email
2. **Check Calendars:** EA manually checks executive's calendar for availability
3. **Propose Times:** EA drafts email proposing 2-3 available time slots to participants
4. **Manual Tentative Block:** EA manually creates "tentative" calendar event (current workaround)
5. **Wait for Responses:** EA monitors inbox for participant confirmations/declines
6. **Track Manually:** EA keeps mental or written notes of who has/hasn't responded
7. **Follow-up:** EA manually sends reminder emails if no response after 48 hours
8. **Convert or Remove:**
   - If confirmed: EA manually converts tentative event to real meeting
   - If declined: EA manually removes tentative event and starts over
9. **Repeat:** Process repeats 10-20 times per week

**Time per scheduling cycle:** 15-20 minutes (current)
**Frequency:** 10-20 scheduling requests per week
**Total time:** ~3-6 hours/week on just scheduling coordination

### User Existing Tools

**Currently using:**
- **Cabinet** - EA productivity platform with shadow hold capability (manual)
  - Provides calendar integration (Google/Outlook)
  - Has templates for common workflows
  - Requires manual shadow hold creation (extra clicks)
  - No AI automation
  - No automatic email drafting
  - No automatic response tracking

- **Email** (Gmail/Outlook) - Manual composition and monitoring
- **Calendar** (Google Calendar/Outlook) - Manual event creation

**Cabinet's limitations:**
- Manual shadow hold management (EA must create each one)
- No AI email drafting (templates only)
- Slow email workflow (ping-pong takes 24-48 hours)
- No automatic follow-ups
- Doesn't learn preferences over time
- No proactive suggestions

### Trigger

**What initiates the process:**
- Incoming email requesting meeting
- Executive asks EA to schedule something
- Recurring meeting needs rescheduling

### Frequency

**How often:**
- 10-20 scheduling requests per week
- 2-4 requests per business day
- Higher frequency during quarterly planning periods

### Friction Points

**Slowdowns and manual effort causing problems:**

1. **Manual Shadow Hold Creation**
   - EA must remember to create tentative event
   - Extra clicks and context switching
   - Easy to forget, leading to double-booking risk

2. **Email Composition Time**
   - Each scheduling email takes 3-5 minutes to compose
   - Similar structure but requires customization
   - Mental overhead of professional language

3. **Response Tracking Cognitive Load**
   - Must manually track who responded
   - Can't tell at a glance what's pending vs confirmed
   - Requires checking email constantly

4. **Double-Booking Risk**
   - While waiting for confirmation from Person A, Person B requests same slot
   - Without visual reminder, EA might accidentally propose same time
   - Results in embarrassing conflicts

5. **No Automatic Follow-ups**
   - EA must manually track when to send reminders
   - Things fall through cracks
   - Delays scheduling completion

6. **Email Ping-Pong Delays**
   - Back-and-forth takes 24-48 hours
   - Slows down decision-making
   - Executive availability changes during wait

## Desired State

### User Success Criteria

**Specific metrics defining success:**

1. **Time Reduction:** 70% reduction in scheduling coordination time (3-6 hours → 1-2 hours/week)
2. **Speed:** Reduce scheduling cycle from 24-48 hours to 4-6 hours
3. **Error Prevention:** Zero double-bookings during pending confirmation periods
4. **Automation Rate:** AI handles 70% of scheduling work automatically, EA approves/adjusts 30%
5. **Response Rate:** 100% follow-up on non-responses (vs current ~70%)

**Measurable outcomes:**
- Shadow holds created automatically (vs manual)
- Emails drafted in 90 seconds (vs 3-5 minutes)
- No double-booking incidents (vs ~2-3 per month currently)
- EA approval time per request: 2 minutes (vs 15-20 minutes end-to-end)

### Expected Impact

**How the user's work improves:**

1. **Cognitive Load Reduction**
   - No more mental tracking of pending responses
   - Visual clarity of calendar state (pending vs confirmed)
   - Reduced constant email checking

2. **Time Reclaimed**
   - 2-4 hours/week freed up for strategic EA work
   - Less time on tactical coordination
   - More time for executive support

3. **Stress Reduction**
   - No fear of double-booking
   - Confidence that follow-ups won't be missed
   - Peace of mind that system is tracking everything

4. **Professional Quality**
   - AI-drafted emails maintain professional tone
   - Consistent communication quality
   - Faster responsiveness to executives and stakeholders

5. **Scalability**
   - Can handle increased meeting volume without proportional time increase
   - Emergency rescheduling becomes 10x faster
   - EA can support multiple executives if needed

### Constraints

**Resource limitations and non-negotiable aspects:**

1. **Budget Constraint**
   - Bootcamp project = minimal/zero cost for MVP
   - Must use free tiers: Supabase free, Vercel hobby, etc.
   - AI costs must stay under $10/month during testing

2. **Technical Constraint**
   - Brian is learning to code (not experienced developer)
   - Must use accessible tools: n8n (no-code/low-code), not pure code
   - 7-week timeline to working MVP

3. **User Control Requirement**
   - EA must approve before emails are sent (no autonomous sending)
   - EA can edit AI-drafted content
   - EA can override AI suggestions
   - Human-in-the-loop at critical decision points

4. **Integration Constraint**
   - Must integrate with standard calendars (Google/Outlook)
   - Must work with standard email (no custom email server)
   - Can't require executive to change tools

5. **Privacy Constraint**
   - Calendar data is sensitive
   - Email content may contain confidential info
   - Must handle data securely (but not PHI/HIPAA for MVP)

6. **Time Constraint**
   - Sprint 1 (Week 2): Basic prototype
   - Sprint 2 (Week 4): User validation
   - Sprint 3 (Week 6): Deployed and usable
   - Final Demo (Week 7): Presenting working system

7. **Scope Constraint (MVP)**
   - Single calendar integration (start with Google OR Outlook, not both)
   - English language only
   - No mobile app (web-based only)
   - No team collaboration features (single EA use)
   - No AI preference learning (manual rules only for MVP)
