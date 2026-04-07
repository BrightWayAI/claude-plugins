---
name: weekly-alignment-setup
description: >
  Set up or update your Weekly Alignment Scanner. Interviews you about your org structure, picks Slack channels to monitor via the Slack API, and captures what kinds of cross-team misalignment to watch for. Run this on first use or anytime your org changes.
---

# Weekly Alignment Scanner — Setup

You are running the onboarding flow for the Weekly Alignment Scanner plugin. Your job is to understand the user's org through a short, conversational interview, then save their answers so the weekly scan runs without re-asking.

**Rules:**
- Ask ONE question at a time. Wait for each answer before moving on.
- Be conversational — this should feel like a quick intake call, not a form.
- When presenting options, use numbered lists so the user can just reply with numbers.
- After every selection question, follow up with an open-ended probe for nuance.

---

## Step 0: Check Slack Connection

Before asking any questions, check if Slack MCP tools are available (look for `slack_read_channel`, `slack_search_channels`, `slack_search_public_and_private`).

**If Slack is NOT connected:** Stop and tell the user:
"I need a Slack connection to set this up — the scanner reads your channels directly. Add a Slack MCP server to your Claude Code settings (or Cowork workspace), then run `/weekly-alignment-setup` again."

Do not proceed with the interview if Slack is not connected.

**If Slack IS connected:** Continue to Step 1.

---

## Step 1: Who Are You?

"Let's set up your weekly alignment scanner. First — what's your name, title, and what do you oversee? Just give me the quick version."

One open-ended question. Extract their name, title, and scope of responsibility from whatever they say.

---

## Step 2: Your Teams

"What teams do you have visibility into? Give me the team names and roughly what each one owns — I'll organize it from there."

One open-ended question. Extract team names, sizes (if mentioned), and ownership areas. Don't force a specific format.

---

## Step 3: Pick Slack Channels

Use `slack_search_channels` to pull the list of public channels in their workspace.

Present the channels as a numbered list, grouped sensibly (alphabetical is fine). Then ask:

"Here are the public channels in your workspace. Which ones should I scan every week? Just give me the numbers.

[numbered channel list]

Pick the ones where your teams discuss work, make decisions, and coordinate — team channels, project channels, cross-functional channels, etc."

After they select, follow up:

"Got it. Any of those that are more 'nice to check' vs. 'must read every week'? I'll treat those as secondary — I'll skim them for anything relevant but won't deep-read them."

Let them split their selection into primary (deep read) and secondary (skim). If they don't distinguish, treat all as primary.

**Then ask about private channels — give them equal weight:**

"Now — are there private channels I should also be scanning? These are often where the most important conversations happen — leadership channels, project war rooms, cross-team syncs. I can't pull these from the API, so just give me the channel names.

For each one, tell me if it's primary (deep read) or secondary (skim). Or say 'none' if public channels cover it."

Add any private channels to the same primary/secondary lists as the public ones.

---

## Step 4: What Goes Wrong

Present common misalignment patterns as a checklist, then probe for specifics.

"Now the important part — what kinds of cross-team problems should I watch for? Pick the ones that actually happen in your org:

1. **Duplicate work** — Two teams building the same thing without knowing
2. **Conflicting decisions** — One team decides X while another decides the opposite
3. **Uninformed stakeholders** — Decisions that affect a team that wasn't in the room
4. **Resource conflicts** — Same person or team pulled in multiple directions
5. **Dependency gaps** — One team blocked on work another team hasn't started
6. **Timeline mismatches** — Teams assuming different deadlines for the same thing
7. **Scope creep across teams** — One team's expanding scope quietly eating into another's territory
8. **Communication dead zones** — Decisions made in DMs or small threads that never surface

Just give me the numbers that apply."

After they select, follow up:

"Anything specific to your org that's not on that list? Think about the last time teams were misaligned and nobody caught it — what happened?"

This is where the real nuance comes. Let them tell a story. Extract specific patterns from it.

---

## Step 5: Current Risks

"Any specific tensions I should keep an eye on right now? Things like:
- A migration that might conflict with a feature launch
- Two teams competing for the same resource
- A deadline that depends on work another team hasn't started

Say 'nothing right now' if things are calm."

---

## Step 6: Delivery Preferences

Present options, then let them customize:

"Last one — how do you want your weekly brief delivered?

**When:**
1. Monday morning
2. Sunday evening
3. Other (tell me)

**Where:**
1. Slack DM to you
2. Post to a specific Slack channel
3. Slack canvas
4. Just show me in conversation

**Detail level:**
1. Short — just flag the conflicts, a few bullets
2. Medium — conflicts with context and suggested actions
3. Detailed — full analysis with message references and recommendations"

---

## Step 7: Write the Context File

Once all answers are collected, write the org-context file at:
`${CLAUDE_PLUGIN_ROOT}/references/org-context.md`

Use this format:

```markdown
# Org Context — Weekly Alignment Scanner

> Last updated: [current date]

---

## Your Role

**Name:** [name]

**Title:** [title]

**What you're responsible for:**
[extracted from their answer]

---

## Org Structure

**Teams:**
[formatted as bulleted list with **Team Name** (size if mentioned) — what they own]

---

## Slack Channels to Monitor

**Primary channels (deep read every scan):**
[bulleted list with #channel-name — brief description if available from Slack metadata]

**Secondary channels (skim for relevant activity):**
[bulleted list, or "None — all channels are primary."]

---

## What to Watch For

**Selected patterns:**
[bulleted list of the patterns they picked from the checklist]

**Org-specific patterns:**
[anything they added from the open-ended follow-up, formatted as bullets]

**Specific ongoing risks or tensions:**
[their current risks, or "None flagged currently."]

---

## Delivery Preferences

**When:** [their answer]

**Where:** [their answer]

**Detail level:** [their answer]
```

---

## Step 8: Confirm

After writing the file, show the user a summary:

"Your alignment scanner is configured:

- **Channels:** [count] primary, [count] secondary
- **Teams:** [team names]
- **Watching for:** [count] patterns
- **Delivery:** [when], [where], [detail level]

**To run your first scan now:** say 'run my weekly alignment check'
**To schedule it:** say '/schedule' and set it for [their preferred timing]
**To update this config:** run '/weekly-alignment-setup' again"
