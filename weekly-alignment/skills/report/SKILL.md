---
name: report
description: >
  Deep dive into a specific cross-team conflict or alignment issue. Use when the user says "dig into", "deep dive", "tell me more about", "investigate the conflict between", "report on", or any variation of wanting a detailed analysis of a specific misalignment — not a broad scan, but a focused investigation of one issue.
---

# Alignment Deep Dive Report

You are producing a focused investigation into a specific cross-team conflict or alignment issue. Unlike the weekly scan (which covers everything) or the daily pulse (which skims), this is a thorough analysis of ONE issue.

## Pre-Flight Check

### Check Slack Connection

Verify that Slack MCP tools are available (`slack_read_channel`, `slack_search_channels`, `slack_search_public_and_private`).

**If Slack is NOT connected:** Stop and tell the user:
"I need a Slack connection to investigate this. Add a Slack MCP server to your Claude Code settings, then try again."

### Check Org Context

Read the org context file at:
`${CLAUDE_PLUGIN_ROOT}/references/org-context.md`

**If the file does not exist at all, or contains `[NOT YET CONFIGURED]` markers:** Tell the user:
"You haven't set up your alignment scanner yet. Let's do that first."
Then invoke the Skill tool with skill `weekly-alignment-setup`.

**If configured:** Proceed.

## Step 1: Understand the Issue

If the user gave you a specific issue (e.g., "dig into the caching conflict between Platform and Product"), use that.

If they said something vague (e.g., "investigate the thing from this week's scan"), ask:
"Which issue do you want me to dig into? Give me the teams involved or a short description."

## Step 2: Deep Read Relevant Channels

Identify which channels are relevant to this specific issue — both from the configured list and any others the user mentions.

For each relevant channel:

1. Use `slack_read_channel` to read the last **14 days** (not just 7 — go deeper than the weekly scan)
2. Use `slack_search_public_and_private` to search for keywords related to the issue
3. Use `slack_read_thread` to follow any threads where key decisions or discussions happened
4. Track:
   - **Timeline of events** — when did this issue first appear? How did it evolve?
   - **Key people** — who's involved, who made what decisions
   - **Decision points** — where did the paths diverge?
   - **Current state** — where does each team think they stand right now?

## Step 3: Reconstruct the Story

Build a narrative timeline:
- When did each team start their work?
- At what point did they diverge or overlap?
- Were there any moments where someone almost caught it? (e.g., a question in a thread that went unanswered)
- What's the current trajectory if nothing changes?

## Step 4: Assess Impact

- **What breaks if this isn't resolved?** Be specific — wasted sprints, conflicting deploys, customer-facing issues, etc.
- **How much time/effort has already been invested?** Try to estimate from the discussion timeline.
- **What's the blast radius?** Which teams are directly affected? Who else gets impacted downstream?
- **Is there a deadline that forces a decision?** A launch date, a dependency, a customer commitment?

## Step 5: Suggested Next Step

Based on the investigation, recommend ONE concrete next step. Be specific:
- Who should talk to whom
- What decision needs to be made
- By when (if there's a forcing function)

Don't try to produce multiple strategic options with tradeoff analysis — you're working from Slack messages, not strategy documents. One clear, actionable recommendation is more useful than three speculative ones.

## Step 6: Deliver the Report

Format:

---

**ALIGNMENT DEEP DIVE — [Short issue title]**
**Date:** [date]
**Teams:** [teams involved]

---

### Summary
[2-3 sentences — what's happening, why it matters, what to do]

### Timeline
[Chronological reconstruction with dates, channel references, and key quotes]

### Impact Assessment
- **Risk level:** HIGH / MEDIUM / LOW
- **Effort at risk:** [estimate]
- **Blast radius:** [teams affected]
- **Decision deadline:** [date or "no hard deadline"]

### Suggested Next Step
[One concrete recommendation — who should talk to whom, what decision needs to be made, and by when. E.g., "Schedule a 30-min sync between @alice and @bob this week to decide who owns caching. Share this report as pre-read. Decision needed before Thursday's deploy."]

---

Deliver based on the user's org context preferences. For reports, also offer to create a Slack canvas (`slack_create_canvas`) since these are longer documents worth sharing.

## Save to History

Save the full report to:
`${CLAUDE_PLUGIN_ROOT}/history/reports/[YYYY-MM-DD]-[short-slug].md`

Where `[short-slug]` is a kebab-case summary of the issue (e.g., `caching-conflict-platform-product`).

## Notes

- **Go deep.** This is the opposite of the daily pulse. Read threads, follow conversations, trace decisions back to their origin.
- **Name names.** Reference specific people, messages, and dates. This report needs to be actionable, not abstract.
- **Be fair.** Present both sides. Don't frame one team as "wrong" — frame the situation as a coordination gap.
- **Offer to update risks.** If this investigation reveals something that should be added to the ongoing risk tracker, offer to run `weekly-alignment:update-risks`.

After delivering the report, offer:
"Want me to add this to your tracked risks so the weekly scan keeps an eye on it? Just say 'yes' and I'll update your config."

If they say yes, invoke the Skill tool with skill `weekly-alignment:update-risks`.
