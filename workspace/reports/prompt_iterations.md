# Prompt Iterations: Shadow Hold Assistant

**Date:** 2026-02-14

## Iteration History

### Version 1 (prompt_v1.txt) - Initial Prompt

**What changed:** First version, written from scratch based on evaluation results.

**Key design decisions:**
- Structured around 3 jobs: Convert times, Check reasonableness, Flag issues
- Added business hours validation (7 AM - 9 PM good, 6-7 AM / 9-10 PM warning, outside = bad)
- Required IANA timezone names to prevent abbreviation ambiguity (TC6 learning)
- Included alternative time suggestions when issues found
- Added draft email output with [SHADOW HOLD] subject prefix

**Test results against TC8 (4-party global meeting):**
- Correctly flagged NYC at 4:00 AM as "BAD" (before 6 AM threshold)
- Identified 14-hour timezone span as problematic
- Suggested alternative UTC times with all local conversions shown
- This directly addresses the TC8 failure from the evaluation round

**Quality improvement:**
- TC8 went from PASS (technical) / FAIL (practical) to full PASS
- Business hours validation catches impractical meeting times
- Alternative suggestions give the EA actionable next steps

### Observations

**What the prompt does well:**
- Clear structure makes output consistent and scannable
- Business hours tiers (good/warning/bad) match real EA decision-making
- Shadow hold framing sets correct expectations in email drafts

**Areas for future iteration:**
- Output sometimes uses abbreviations (EST, IST) despite IANA rule - could tighten this or accept abbreviations in display output while using IANA internally
- Could add participant preference handling (e.g., "Kenji prefers mornings")
- Email draft tone could be customizable per organization

## Current Best Version

**final_prompt.txt** = prompt_v1.txt (single iteration was sufficient to address the critical TC8 finding)
