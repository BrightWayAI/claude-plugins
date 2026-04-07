---
name: daily-pulse
description: >
  Quick daily check across monitored Slack channels. Lighter than the full weekly scan — skims for anything urgent or noteworthy since yesterday. Use when the user says "daily pulse", "quick check", "anything happening today", "daily alignment", "what did I miss", or any variation of a quick cross-team status check.
---

# Daily Pulse Check

You are running a quick daily pulse — a lightweight version of the weekly alignment scan. Your job is to skim Slack channels for anything the user needs to know about TODAY, not produce a full cross-reference analysis.

## Pre-Flight Check

### Check Slack Connection

Verify that Slack MCP tools are available (`slack_read_channel`, `slack_search_channels`).

**If Slack is NOT connected:** Stop and tell the user:
"I need a Slack connection to run the pulse check. Add a Slack MCP server to your Claude Code settings, then try again."

### Check Org Context

Read the org context file at:
`${CLAUDE_PLUGIN_ROOT}/references/org-context.md`

**If the file does not exist at all, or contains `[NOT YET CONFIGURED]` markers:** Tell the user:
"You haven't set up your alignment scanner yet. Let's do that first."
Then invoke the Skill tool with skill `weekly-alignment-setup`. Once complete, continue with the pulse check.

**If configured:** Proceed.

## Step 1: Quick Channel Scan

For each **primary** channel in the org context:

1. Use `slack_read_channel` to read the last 24 hours of messages
2. Look for:
   - **Decisions made** — anything that changes direction or scope
   - **Urgent issues** — outages, blockers, escalations
   - **New work kicked off** — that other teams should know about
   - **Questions about ownership** — signals of confusion
   - **Deadlines mentioned** — especially imminent ones

For **secondary** channels: only check if something from a primary channel references them, or skip entirely.

## Step 2: Surface What Matters

Don't do a full cross-reference analysis. Instead, surface:

1. **Anything urgent** — blockers, outages, escalations that affect multiple teams
2. **Decisions that landed** — especially ones that change direction for other teams
3. **Things starting today** — new initiatives, launches, deploys
4. **Quick flags** — anything that looks like it could become a conflict but isn't one yet

## Step 3: Deliver the Pulse

Keep it short. Format:

---

**DAILY PULSE — [date]**

**🔴 Urgent** (if any)
- [item]

**📋 Decisions**
- [item]

**🚀 Starting / Shipping**
- [item]

**👀 Worth Watching**
- [item]

**✅ All Clear** — if nothing noteworthy, just say so. "Quiet day across your channels. Nothing flagged."

---

Deliver based on the user's preference from org context. If no delivery preference or in conversation, just show it inline.

If any item looks like it could be a real cross-team conflict, add a note:
"This might be worth a deeper look — say 'dig into [issue]' for a full investigation, or wait for your next weekly scan."

## Step 4: Save to History

After delivering the pulse, save a copy to:
`${CLAUDE_PLUGIN_ROOT}/history/pulses/[YYYY-MM-DD].md`

Keep it lightweight — just the pulse output as-is.

## Notes

- This is NOT the weekly scan. Don't try to do full cross-team analysis.
- Speed > thoroughness. Get in, get out, surface the signal.
- If something looks like a real conflict, flag it and suggest running the full `/weekly-alignment:scan` for deeper analysis.
