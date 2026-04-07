---
name: weekly-alignment-setup
description: >
  Set up or update your Weekly Alignment Scanner. Interviews you about your org structure, Slack channels to monitor, and what kinds of cross-team misalignment to watch for — then saves your answers so the weekly scan knows exactly where to look and what matters. Run this on first use or anytime your org changes.
---

# Weekly Alignment Scanner — Setup

You are running the onboarding flow for the Weekly Alignment Scanner plugin. Your job is to understand the user's org through a conversational interview, then write their answers to the org-context file so the weekly scan can run without re-asking.

**Important:** Ask ONE question at a time. Wait for each answer before asking the next. Be conversational and specific — the better the answers, the more useful the weekly scan will be. When the user gives short answers, probe for specifics.

## Step 1: Collect Information

Ask the following questions in order, one at a time. Use AskUserQuestion for each.

### Question 1 — Who Are You?
"Let's set up your weekly alignment scanner. First — what's your name, title, and what are you responsible for? (e.g., 'I'm VP Eng, I oversee three teams that build our core product')"

### Question 2 — Your Teams
"What teams do you manage or have visibility into? For each one, give me:
- Team name
- Roughly how many people
- What they own

For example: 'Platform (8 people, owns infra and dev tooling), Product (6 people, owns user-facing features), Data (4 people, owns pipelines and analytics)'"

### Question 3 — How Teams Interact
"Where do these teams overlap or depend on each other? Think about:
- Shared codebases, APIs, or data pipelines
- Handoff points where one team's output is another's input
- Areas where two teams have opinions about the same thing

This is the stuff that causes problems when nobody's watching."

### Question 4 — Slack Channels (Primary)
"Now the important part — which Slack channels should I monitor every week? List the channels where your teams discuss work, make decisions, and coordinate. Include:
- Each team's main channel
- Any cross-team or leadership channels
- Active project channels for current initiatives

Be specific — channel names like #platform-eng, #product-dev, etc."

### Question 5 — Slack Channels (Secondary)
"Any secondary channels I should check when something looks relevant? These might be channels you don't read every day but where important context sometimes surfaces — like #incidents, #announcements, #random-but-actually-useful, etc.

Say 'none' if the primary list covers it."

### Question 6 — What Goes Wrong
"This is the most important question. What kinds of cross-team misalignment actually happen in your org? Be specific about real patterns you've seen. For example:
- 'Two teams started building the same caching layer without knowing'
- 'Product commits to timelines without checking with Platform on feasibility'
- 'Data team changes a schema and downstream consumers don't find out until things break'
- 'Decisions get made in DMs and never surface to affected teams'

The more specific you are, the better I'll be at catching these."

### Question 7 — Current Risks
"Any specific tensions or risks I should be watching right now? Things like:
- A migration that might conflict with a feature launch
- Two teams that are both trying to hire from the same budget
- A deadline that depends on work another team hasn't started yet

Say 'none right now' if things are calm — I'll still watch for the patterns from the last question."

### Question 8 — Delivery Preferences
"Last one — how and when do you want your weekly brief?
- **When:** e.g., 'Monday morning at 8am ET' or 'Sunday night'
- **Where:** e.g., 'DM me in Slack', 'Post to #leadership', 'Create a Slack canvas'
- **Format:** e.g., 'Short bullets — just flag the conflicts', 'Detailed with recommendations', 'Somewhere in between'"

## Step 2: Write the Context File

Once all answers are collected, write the org-context file at:
`${CLAUDE_PLUGIN_ROOT}/skills/weekly-alignment/references/org-context.md`

Use this format:

```markdown
# Org Context — Weekly Alignment Scanner

> Last updated: [current date]

---

## Your Role

**Name:** [their answer]

**Title:** [their answer]

**What you're responsible for:**
[their answer]

---

## Org Structure

**Teams you manage or oversee:**
[formatted as bulleted list with **Team Name** (size) — what they own]

**Key relationships between teams (shared dependencies, handoff points, frequent friction):**
[their answer, formatted as bullets]

---

## Slack Channels to Monitor

**Primary channels (check every scan):**
[formatted as bulleted list with #channel-name — description of what happens there]

**Secondary channels (check when relevant):**
[formatted as bulleted list, or "None — primary channels cover it."]

---

## What to Watch For

**Types of misalignment that tend to happen in your org:**
[formatted as bulleted list — their specific patterns]

**Specific ongoing risks or tensions to monitor:**
[formatted as bullets, or "None flagged currently — monitoring for general patterns only."]

---

## Delivery Preferences

**When to deliver:** [their answer]

**Where to deliver:** [their answer]

**Format preference:** [their answer]
```

## Step 3: Confirm & Next Steps

After writing the file, show the user a summary:

"Your alignment scanner is configured. Here's what I captured:

- **Monitoring:** [count] primary channels, [count] secondary channels
- **Teams:** [list team names]
- **Watch patterns:** [count] misalignment patterns configured
- **Delivery:** [when], [where], [format]

**To run your first scan now:** say 'run my weekly alignment check'

**To schedule it automatically:** say '/schedule' and set it to run [their preferred timing]

**To update this config later:** run '/weekly-alignment-setup' again"

## Step 4: Verify Slack Connection

Check if Slack MCP tools are available (look for tools like `slack_read_channel`, `slack_search_public`, `slack_search_public_and_private`).

- **If found:** "Slack is connected — you're all set."
- **If not found:** "I don't see a Slack connector set up yet. You'll need to add a Slack MCP server to your Claude Code settings before the scan can read your channels. Once that's connected, everything else is ready to go."
