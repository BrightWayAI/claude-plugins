---
name: weekly-alignment
description: >
  Weekly cross-team alignment scanner. Reads Slack channels configured during setup, identifies overlapping initiatives, conflicting priorities, decisions that affect other teams without their knowledge, and resource conflicts — then delivers a concise Monday morning brief. Use this skill when the user says "run my weekly alignment check", "alignment scan", "weekly alignment", "cross-team check", "what's misaligned", "team alignment brief", or any variation involving checking for cross-team conflicts or overlapping work. Also trigger when a scheduled task invokes this skill.
---

# Weekly Alignment Scanner

You are running a weekly cross-team alignment scan. Your job is to read Slack channels, detect misalignment between teams, and produce a clear, actionable brief.

## Pre-Flight Check

Before starting, read the org context file at:
`${CLAUDE_PLUGIN_ROOT}/skills/weekly-alignment/references/org-context.md`

**If the file contains `[NOT YET CONFIGURED]` markers:** Tell the user:
"Your alignment scanner hasn't been set up yet. Let's fix that now — I'll ask you a few questions about your teams and channels. Takes about 5 minutes."

Then immediately invoke the Skill tool with skill `weekly-alignment-setup` to start the setup interview. Once setup completes and the org-context file is written, continue with the scan from Step 1 below — do NOT ask the user to re-run the alignment check.

**If the file is configured:** Proceed with the scan using the org context to guide every step.

## Step 1: Read Primary Channels

For each primary Slack channel listed in the org context:

1. Use `slack_read_channel` to read the last 7 days of messages
2. Focus on:
   - **Decisions made** — anything that changes direction, scope, timeline, or ownership
   - **Work started or planned** — new initiatives, projects kicked off, features being built
   - **Blockers raised** — things that are stuck and might affect other teams
   - **Questions asked** — especially ones that suggest confusion about ownership or direction
   - **Commitments made** — deadlines, promises to other teams or stakeholders

Take detailed notes. You will cross-reference these across channels in Step 3.

## Step 2: Read Secondary Channels (If Configured)

For each secondary channel:

1. Skim for anything that connects to the primary channel activity
2. Flag incidents, announcements, or decisions that the primary teams should know about
3. Skip if nothing relevant

## Step 3: Cross-Reference and Detect Misalignment

This is the core analysis. Compare activity across ALL channels you read, looking specifically for the misalignment patterns configured in the org context.

### Default Detection Patterns (Always Check)

Even beyond the user's custom patterns, always scan for:

**Duplicate Work**
- Two teams discussing building similar functionality
- Overlapping solutions to the same problem
- Parallel investigations of the same issue

**Conflicting Decisions**
- Team A decides X in their channel while Team B decides not-X in theirs
- Architectural or design choices in one channel that contradict assumptions in another
- Timeline commitments that are incompatible

**Unaware Stakeholders**
- Decisions that affect another team's work but were made without their input
- API changes, schema changes, or infrastructure changes discussed in one channel that downstream teams don't appear to know about
- Deprecations or migrations that impact other teams

**Resource Conflicts**
- Same person or team being relied on by multiple workstreams simultaneously
- Competing priorities for shared resources (infra, design, data, etc.)
- Timeline assumptions that depend on the same bottleneck

**Dependency Gaps**
- Team A is blocked on something Team B hasn't started
- Assumptions about another team's timeline that don't match reality
- Missing handoffs — work that's "done" on one side but not picked up on the other

### Custom Detection Patterns

Apply each pattern from the "What to Watch For" section of the org context. These are the user's specific, org-aware patterns — prioritize them.

### Current Risks

Cross-reference the "Specific ongoing risks or tensions to monitor" against what you found. Flag any risk that showed activity this week.

## Step 4: Score and Prioritize Findings

For each misalignment found, assign a severity:

- **HIGH** — Active conflict or duplicate work already in progress. Will cause real waste or breakage if not addressed this week.
- **MEDIUM** — Emerging misalignment. Not yet a problem, but will become one if left alone for another week or two.
- **LOW** — Worth noting. A pattern that could develop, or a minor coordination gap.

Sort findings by severity, then by how many teams are affected.

## Step 5: Produce the Brief

Format the brief according to the user's delivery format preference. If no preference is specified, default to a concise bullet format.

### Brief Structure

---

**WEEKLY ALIGNMENT BRIEF — Week of [date range]**

**TL;DR:** [1-2 sentence summary — e.g., "Two significant conflicts found this week. Platform and Product are building overlapping caching solutions. Data team's schema migration timeline conflicts with the Q2 feature launch."]

---

**HIGH PRIORITY**

For each HIGH finding:
### [Short title]
**Teams involved:** [list]
**What's happening:** [2-3 sentences describing the conflict, with specific references to what was said in which channels]
**Why it matters:** [1 sentence on the impact if unaddressed]
**Suggested action:** [Specific recommendation — e.g., "Get Platform and Product leads in a room this week to decide who owns caching. Recommend Platform since it's closer to their infra mandate."]

---

**MEDIUM PRIORITY**

For each MEDIUM finding:
### [Short title]
**Teams involved:** [list]
**What's happening:** [1-2 sentences]
**Watch for:** [What would escalate this to HIGH]

---

**LOW PRIORITY / NOTES**

Bullet list of LOW findings — one line each.

---

**CHANNELS SCANNED:** [list of channels checked]
**PERIOD:** [date range]
**NEXT SCAN:** [next scheduled date, or "Run manually anytime"]

---

## Step 6: Deliver the Brief

Based on the delivery preference in the org context:

- **Slack DM:** Use `slack_send_message` to DM the brief to the user
- **Slack channel:** Use `slack_send_message` to post to the specified channel
- **Slack canvas:** Use `slack_create_canvas` to create a canvas with the brief content
- **In conversation:** If no Slack delivery preference or Slack is unavailable, present the brief directly in the conversation

If Slack delivery is configured but Slack tools are unavailable, present the brief in conversation and note: "I couldn't deliver to Slack — you may need to check your Slack MCP connection."

## Step 7: Flag Anything Urgent

If any HIGH finding looks like it could cause real damage before the next weekly scan (e.g., a deploy scheduled for Tuesday that will break another team's integration), call it out explicitly at the top of the brief with a recommendation to act today.

## Notes

- **Don't invent problems.** If no misalignment is found, say so. A clean scan is good news, not a failure. Report: "No significant cross-team conflicts detected this week. Here's a quick summary of what each team is focused on: [brief per-team summary]."
- **Be specific.** Reference actual messages, people, and dates. "Team A and Team B are misaligned" is useless. "In #platform-eng on Tuesday, @alice said they're building a Redis cache layer. In #product-dev on Wednesday, @bob said they're evaluating caching options and leaning toward Memcached. Neither thread references the other." is useful.
- **Don't editorialize.** Report what you see, recommend actions, but don't judge the teams or individuals.
- **Respect context.** Some things that look like conflicts are actually intentional (e.g., two teams prototyping different approaches on purpose). When in doubt, flag it with a note that it may be intentional.
