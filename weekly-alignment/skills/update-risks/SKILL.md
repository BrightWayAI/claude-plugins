---
name: weekly-alignment-update-config
description: >
  Quickly update any part of the alignment scanner config — risks, tensions, teams, channels, delivery preferences, or watch patterns — without re-running the full setup interview. Use when the user says "update risks", "add a risk", "new tension", "remove risk", "things changed", "update what to watch for", "update config", "change delivery", "update teams", "change channels", "update preferences", "add a channel", "remove a team", or any variation of modifying the scanner configuration.
---

# Update Scanner Config

You are helping the user quickly update their alignment scanner config — without re-running the full setup. This covers risks, channels, teams, patterns, delivery preferences, and any other quick config change.

## Pre-Flight Check

Read the org context file at:
`${CLAUDE_PLUGIN_DATA}/references/org-context.md`

**If the file does not exist at all, or contains `[NOT YET CONFIGURED]` markers:** Tell the user:
"You haven't set up your alignment scanner yet. Let's do that first."
Then invoke the Skill tool with skill `weekly-alignment-setup`. Once complete, return here.

**If configured:** Proceed.

## Step 1: Show Current State

Show the user their full current config, organized by section:

"Here's your current scanner config:

**Your Role:**
[name, title, responsibilities from org context]

**Teams:**
[list from org context]

**Primary Channels:**
[list from org context]

**Secondary Channels:**
[list from org context]

**Watch Patterns:**
[list from org context]

**Ongoing risks & tensions:**
[list from org context]

**Delivery Preferences:**
[when, where, detail level from org context]

What do you want to change?"

## Step 2: Collect Changes

Let the user tell you what to change conversationally. They can update ANY section:

- **Risks** — Add, remove, or update ongoing risks and tensions
- **Channels** — Add or remove primary/secondary channels, move channels between tiers
- **Teams** — Add, remove, or update team descriptions and ownership areas
- **Watch patterns** — Add or remove misalignment patterns to scan for
- **Delivery preferences** — Change when, where, or detail level of the brief

If they want to add channels, use `slack_search_channels` to verify the channel exists before adding it.

Accept multiple changes in one go. Confirm what you heard:

"Got it, here's what I'll update:
- **Add:** [items]
- **Remove:** [items]
- **Change:** [items]

Sound right?"

**Note:** For major org restructuring (like completely new team structure, new role, or wholesale channel overhaul), recommend running the full `/weekly-alignment-setup` instead — it's designed for that level of change.

## Step 3: Write Updates

Update the org context file at:
`${CLAUDE_PLUGIN_DATA}/references/org-context.md`

Only modify the sections that changed. Do NOT touch other sections unless the user explicitly asked to change them.

Update the "Last updated" date at the top of the file.

## Step 4: Confirm

"Updated. Your next scan will pick up these changes. Want me to run a quick pulse check now to see if anything's already showing up?"

If they say yes, invoke the Skill tool with skill `weekly-alignment-daily-pulse`.
