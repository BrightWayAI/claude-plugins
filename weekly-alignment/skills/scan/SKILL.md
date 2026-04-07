---
name: scan
description: >
  Weekly cross-team alignment scanner. Reads Slack channels configured during setup, identifies overlapping initiatives, conflicting priorities, decisions that affect other teams without their knowledge, and resource conflicts — then delivers a concise Monday morning brief. Use this skill when the user says "run my weekly alignment check", "alignment scan", "weekly alignment", "cross-team check", "what's misaligned", "team alignment brief", or any variation involving checking for cross-team conflicts or overlapping work. Also trigger when a scheduled task invokes this skill.
---

# Weekly Alignment Scanner

You are running a weekly cross-team alignment scan. Your job is to read Slack channels, detect misalignment between teams, and produce a clear, actionable brief.

## Pre-Flight Check

### Check Slack Connection

Before anything else, verify that Slack MCP tools are available (look for `slack_read_channel`, `slack_search_channels`, `slack_search_public_and_private`).

**If Slack is NOT connected:** Stop and tell the user:
"I need a Slack connection to run the alignment scan. Add a Slack MCP server to your Claude Code settings (or Cowork workspace), then try again."

Do not proceed without Slack.

### Check Org Context

Read the org context file at:
`${CLAUDE_PLUGIN_ROOT}/references/org-context.md`

**If the file does not exist at all, or contains `[NOT YET CONFIGURED]` markers:** Tell the user:
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

**For high-traffic channels (if a channel has very high message volume):**
1. First use `slack_search_public_and_private` to search within the channel for keywords related to: decisions, launches, migrations, deadlines, blockers, new projects, and any org-specific patterns from the config
2. Use `slack_read_thread` to follow the most relevant threads
3. Only fall back to full `slack_read_channel` for channels with manageable volume

This keeps the scan focused and avoids hitting API limits on busy channels.

## Step 2: Read Secondary Channels (If Configured)

For each secondary channel:

1. Skim for anything that connects to the primary channel activity
2. Flag incidents, announcements, or decisions that the primary teams should know about
3. Skip if nothing relevant

## Step 3: Cross-Reference and Detect Misalignment

This is the core analysis. Compare activity across ALL channels you read, looking specifically for the misalignment patterns configured in the org context.

### Prior Scan Context

Before cross-referencing, check `${CLAUDE_PLUGIN_ROOT}/history/` for recent scan files. If prior scans exist:
- Note any HIGH or MEDIUM findings that were flagged in previous weeks and appear again — these are **recurring conflicts** and should be escalated in severity
- Call out in the brief: "⚠️ This issue was also flagged on [date(s)]. It may need executive attention."

If no prior scans exist, skip this step.

Also check `${CLAUDE_PLUGIN_ROOT}/history/pulses/` for daily pulse files from the past 7 days. If pulses exist:
- Review what was flagged during the week
- Check if any pulse flags developed into actual conflicts
- Reference pulse findings in the brief where relevant: "This was first spotted in a daily pulse on [date]"

And check `${CLAUDE_PLUGIN_ROOT}/history/reports/` for any recent deep dive reports. If reports exist:
- Reference prior investigations: "A deep dive on this issue was done on [date]"
- Compare current state to what was found in the report

**Resolution tracking:** If a HIGH or MEDIUM finding from the previous scan is NOT detected this week, note it in the brief:
"✅ [Issue title] (flagged [date]) — not detected this week. May be resolved — confirm with the teams involved."

This closes the loop on previously flagged issues.

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

## Step 4: Prioritize and Produce the Brief

For each finding, assign severity:
- **HIGH** — Active conflict or duplicate work in progress. Will cause waste or breakage if not addressed this week.
- **MEDIUM** — Emerging misalignment. Will become a problem in 1-2 weeks if ignored.
- **LOW** — Worth noting. A pattern developing, or a minor coordination gap.

Sort by severity, then by number of teams affected.

**Formatting the brief:**

Respect the user's **detail level** preference from org context:
- **Short** — TL;DR + bullet list of findings with severity tags. No elaboration.
- **Medium** — TL;DR + each finding gets 2-3 sentences of context and a suggested action.
- **Detailed** — TL;DR + full analysis per finding: what happened, which messages/people, why it matters, suggested action with specifics.

**Urgent findings:** If any HIGH finding could cause real damage before the next weekly scan (e.g., a deploy scheduled for Tuesday that will break another team's integration), lead with it. Put it above the TL;DR with a clear "ACT TODAY" label and a recommended action.

Regardless of detail level, always include:
- A TL;DR (1-2 sentences summarizing the week)
- Severity tags on every finding
- Specific channel/message references (not vague summaries)
- Suggested actions for HIGH items
- Channels scanned and date range at the bottom
- A "WHAT'S NEXT" footer:
  - To dig deeper into any finding: "dig into [issue name]"
  - To update what I'm tracking: "update risks"
  - For a quick check tomorrow: "daily pulse"

Don't invent a rigid template — let the detail level and findings drive the shape.

## Step 5: Deliver the Brief

Based on the delivery preference in the org context:

- **Slack DM:** Use `slack_send_message` to DM the brief to the user
- **Slack channel:** Use `slack_send_message` to post to the specified channel
- **Slack canvas:** Use `slack_create_canvas` to create a canvas with the brief content
- **In conversation:** If no Slack delivery preference or Slack is unavailable, present the brief directly in the conversation

If Slack delivery is configured but Slack tools are unavailable, present the brief in conversation and note: "I couldn't deliver to Slack — you may need to check your Slack MCP connection."

## Step 6: Save to History

After delivering the brief, save a copy to:
`${CLAUDE_PLUGIN_ROOT}/history/[YYYY-MM-DD].md`

The saved file should be the full brief content as-is.

## Notes

- **Don't invent problems.** If no misalignment is found, say so. A clean scan is good news, not a failure. Report: "No significant cross-team conflicts detected this week. Here's a quick summary of what each team is focused on: [brief per-team summary]."
- **Be specific.** Reference actual messages, people, and dates. "Team A and Team B are misaligned" is useless. "In #platform-eng on Tuesday, @alice said they're building a Redis cache layer. In #product-dev on Wednesday, @bob said they're evaluating caching options and leaning toward Memcached. Neither thread references the other." is useful.
- **Don't editorialize.** Report what you see, recommend actions, but don't judge the teams or individuals.
- **Respect context.** Some things that look like conflicts are actually intentional (e.g., two teams prototyping different approaches on purpose). When in doubt, flag it with a note that it may be intentional.
