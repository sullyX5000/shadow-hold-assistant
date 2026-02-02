# Participant Profile

## Background
**Brian Sullivan** brings a rich cross-disciplinary background spanning industrial design, UX research, and strategic consulting. He started in industrial design, developing physical products from pharmaceutical concepts to environment-scale projects, then transitioned to UX research. With a Masters in Business Innovation Leadership, Brian currently works as a Design Strategist and Experience Designer at a global Big 4 consultancy, where he facilitates rapid transformation workshops with Fortune 25-50 clients using design thinking methodologies.

Brian describes himself as not traditionally a software developer, but someone who has always wanted to learn technical skills. He sees AI as a powerful enabler for growing his technical abilities and is particularly excited about mastering the current technology landscape.

## Experience
Brian has been actively using AI tools since 2022, starting with OpenAI and now exploring Claude Code. He regularly experiments with AI tools for personal use and is deeply involved professionally—serving on multiple internal workstreams and committees focused on both internal AI solutions and client-facing AI partnership initiatives.

**Current technical level:** Self-described neophyte with agentic applications, but with 2+ years of hands-on LLM experience and strong strategic understanding of AI applications in enterprise contexts. Eager learner who bridges design thinking with emerging technical capabilities.

## Bootcamp Goals
Brian wants to explore and master the current AI technology landscape to grow his technical abilities, specifically focusing on building agentic applications. He's moving from AI tool user to AI solution builder.

## Personal Motivation
Brian is driven by curiosity and sees AI as the key enabler that can finally allow him to develop the technical skills he's always wanted. He's motivated by the potential to bridge his design and strategy background with hands-on technical implementation.

## Technical Approach
Exploring and learning - open to different approaches but leaning toward understanding the full technical stack. Brian is comfortable with ambiguity and eager to experiment.

## Current Tech Stack
- OpenAI (since 2022)
- Claude Code (exploring)
- Design thinking facilitation tools
- Enterprise consulting and transformation platforms

## Desired Tech Stack
Brian has identified a modern agentic AI stack he wants to learn:
- **GitHub** - Currently knows nothing about it but recognizes its power for learning progression and accessing community resources (high priority)
- **n8n** for workflow automation and orchestration (PRIMARY DEPTH TOOL)
- **Pinecone** for vector databases and semantic search
- **Supabase** (and exploring alternatives) for database management
- **Vercel** for hosting and deployment
- **Lovable** for frontend development
- **Claude Code** for AI agent development

## Project Idea

### Concept
**Intelligent Shadow Hold Assistant for Executive Assistants**

An AI-powered calendar management agent that automates the creation and tracking of shadow holds (tentative calendar blocks) for executive assistants coordinating complex schedules.

### Problem Space
Executive assistants face calendar coordination chaos when scheduling meetings:
- Manually creating tentative calendar events while waiting for confirmations
- Tracking who has responded and who hasn't
- Risk of double-booking during the confirmation waiting period
- Time-consuming email back-and-forth

**Target Users:** Executive Assistants (starting with Brian's wife as primary user/tester)

### Proposed Approach

**Core MVP Features:**
1. **Intelligent Shadow Holds**
   - Auto-creates tentative calendar blocks when EA proposes meeting times
   - Visually distinct from confirmed events
   - Automatically tracks confirmation status

2. **AI Email Automation**
   - AI drafts scheduling request emails
   - EA reviews and approves before sending
   - Reduces email composition time by 70%

3. **Response Tracking**
   - Monitors email responses automatically
   - Updates shadow hold status based on confirmations/declines
   - Auto-converts to real event when all participants confirm
   - Auto-removes if declined or no response after 7 days

4. **Calendar Integration**
   - Google Calendar and/or Outlook integration
   - Real-time sync of shadow holds and confirmed events

**Technical Stack (Planned):**
- n8n for workflow orchestration
- AI (Claude/OpenAI) for email drafting
- Supabase for tracking shadow holds and response status
- Google Calendar / Outlook API integration
- Lovable for frontend interface
- Vercel for deployment

**Out of Scope for Bootcamp MVP:**
- Healthcare-specific features (Epic integration, HIPAA compliance, PHI handling)
- Multi-calendar unification beyond standard business calendars
- Advanced scheduling AI (preference learning comes later)

### Success Criteria
By final demo (March 13):
- Working agent that creates shadow holds automatically
- AI drafts scheduling emails that EA can approve
- Tracks responses and updates calendar accordingly
- Deployed and usable by Brian's wife
- Demonstrably saves 60-70% of scheduling coordination time

### Project Status

**Starting Maturity:** Idea only (conceptual stage)

**Existing Work:** None - starting from scratch during bootcamp

**Current Challenges:**
- No technical implementation yet
- Need to learn GitHub, n8n, calendar APIs, deployment pipeline
- First time building an AI agent end-to-end

**Why This Project:**
- Solves a real problem for an actual user (wife)
- Competes with existing solution (Cabinet) but with AI-first approach
- Aligns perfectly with learning goals (GitHub, n8n, full stack, deployment)
- Clear progression: Sprint 1 (basic shadow holds) → Sprint 2 (AI email) → Sprint 3 (deployment + user testing)

## Goals

### Time Commitment
6-15 hours/week (targeting recommended range of 10-15 hours)
- Heavy focus on weekends
- Approximately 1 hour/day during weekdays
- Willing to push comfort zone to avoid falling behind

### Sprint Goals

**Sprint 1 (Week 2 - Feb 8):**
- Master GitHub basics: commits, branches, pull requests, using repos to learn from others
- Identify a real problem to solve with an agent
- Build simple working prototype
- Create initial evaluation set

**Sprint 2 (Week 4 - Feb 22):**
- Collect user feedback on prototype
- Refine system architecture
- Dive deep into n8n as primary orchestration tool
- Explore deployment options

**Sprint 3 (Week 6 - Mar 8):**
- Deploy the agent to production
- Understand costs and scaling considerations
- Be comfortable with the full stack end-to-end
- Collect real user feedback

**Final Demo (Week 7 - Mar 13):**
- Present a working, deployed agent that solves a real problem
- 8-minute public presentation with live demo

### Check-In Rhythm
Heavy weekends plus about 1 hour each weekday (10-15 hours/week total)

### Success Criteria
By March 13, 2026: **Built and deployed a working agent that solves a real problem**

This means:
- The agent is live and accessible to users
- It solves a genuine problem (not just a demo)
- Brian is confident he can now build AI solutions independently
- GitHub + n8n + deployment pipeline are part of his working toolkit
