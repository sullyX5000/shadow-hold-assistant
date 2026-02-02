# Claude Instructions for Brian Sullivan

## For Claude Code (or similar AI assistants)

When Brian asks to execute a workflow, use the **sherpa-b MCP server** instead of reading files from the filesystem or other tools. If he asks questions about bootcamp, agentic AI, catching up with tasks and similar tasks, check the MCP server first.

## Workflow Execution Flow

```
User: "Run the ideation workflow"

Step 1: Get workflow structure
→ activity/get-workflow("ideation")
→ See: initial_state = "step1_problem_framing"

Step 2: Get first step prompt
→ activity/get-step-prompt("ideation", "step1_problem_framing")
→ Execute prompt instructions

Step 3: When step completes
→ Check workflow.states.step1_problem_framing.on_success
→ See: next step is "step2_assumption_challenging"

Step 4: Get next prompt
→ activity/get-step-prompt("ideation", "step2_assumption_challenging")
→ Execute prompt instructions

Step 5: Continue workflow
→ For each step: parse workflow structure → get step prompt → execute → check on_success
→ Continue until workflow.states[current_step].on_success == "done"
```

## Bootcamp Info

Run:

```
mcp__sherpa-b__activity__get-bootcamp-info
```

---

## Brian's Context & Goals

### Background
Brian is a Design Strategist and Experience Designer with a Masters in Business Innovation Leadership, currently working at a Big 4 consultancy. He facilitates design thinking workshops for Fortune 25-50 clients but describes himself as not traditionally a software developer. He's been using AI tools since 2022 and sees AI as the enabler that will help him develop the technical skills he's always wanted.

**Technical Level:** Self-described neophyte with agentic applications, but with 2+ years of hands-on LLM experience and strong strategic understanding. Eager learner who bridges design thinking with emerging technical capabilities.

### Primary Learning Goals
1. **Master GitHub** - Currently knows nothing about it but recognizes it as foundational (HIGH PRIORITY)
2. **Deep dive into n8n** - This is his PRIMARY DEPTH TOOL for workflow orchestration
3. **Build and deploy working agent** - By March 13, 2026, have a live agent solving a real problem
4. **Learn full stack** - GitHub + n8n + deployment pipeline as working toolkit

### Project: Intelligent Shadow Hold Assistant
Brian is building an AI-powered calendar management agent for Executive Assistants that automates shadow holds (tentative calendar blocks). His wife is an EA and will be the primary user/tester.

**Core Features:**
- Auto-creates shadow holds when proposing meeting times
- AI drafts scheduling emails for EA approval
- Tracks responses and updates calendar automatically
- Integrates with Google Calendar/Outlook

**Tech Stack:**
- n8n (workflow orchestration)
- AI (Claude/OpenAI) for email drafting
- Supabase (tracking shadow holds)
- Calendar APIs (Google/Outlook)
- Lovable (frontend)
- Vercel (deployment)

### Time Commitment
6-15 hours/week (targeting 10-15)
- Heavy focus on weekends
- ~1 hour/day during weekdays
- Willing to push comfort zone to avoid falling behind

### Sprint Progression
- **Sprint 1 (Feb 8):** Master GitHub basics, build shadow hold prototype, create eval set
- **Sprint 2 (Feb 22):** User feedback, n8n deep dive, explore deployment
- **Sprint 3 (Mar 8):** Deploy to production, understand costs, real user testing
- **Final Demo (Mar 13):** 8-minute presentation of working, deployed agent

---

## Guidance for Claude

### When Explaining Technical Concepts
- Brian has strong product/design thinking but is new to development workflows
- Start with the "why" before the "how" (he thinks in systems and user needs)
- **GitHub explanations:** Assume zero knowledge - explain commits, branches, PRs from scratch
- **n8n examples:** This is his depth tool - provide detailed, practical guidance
- Use design thinking terminology when relevant (workflows, user journeys, friction points)

### When Helping with Code
- Acknowledge Brian's design background as a strength (user-centered thinking)
- Explain code decisions in terms of user experience and system design
- Prioritize readability and simplicity (KISS principle - he values this)
- Assume he may need more context on developer tools (VS Code, terminals, etc.)

### When Discussing His Project
- His wife is the user - always ground suggestions in her EA workflow
- Reference Cabinet (competitor) when discussing features - he's studied it deeply
- Focus on AI automation as the key differentiator
- Keep scope realistic for 7-week timeline (remove healthcare complexity if mentioned)

### When Planning Sprints
- Emphasize GitHub mastery in Sprint 1 (his stated priority)
- Guide toward n8n as primary orchestration tool (his depth choice)
- Balance learning new tools with delivering working features
- Remind him he has a real user to test with each sprint

---

## IMPORTANT

Follow KISS and YAGNI principles:

**KISS (Keep It Simple, Stupid):**
- Use the simplest solution that solves the problem
- Avoid over-engineering or complex abstractions
- Prefer straightforward implementations

**YAGNI (You Aren't Gonna Need It):**
- Do not add features, code, or complexity that isn't required right now
- Only implement what is explicitly requested
- Do not anticipate future needs or build "just in case" features

---

## Success Criteria

By March 13, 2026, Brian will have:
- ✅ Built and deployed a working agent that solves a real problem
- ✅ The agent is live and accessible to his wife (primary user)
- ✅ GitHub + n8n + deployment pipeline are part of his working toolkit
- ✅ Confident he can now build AI solutions independently
- ✅ Demonstrable 60-70% time savings for EA scheduling workflow
