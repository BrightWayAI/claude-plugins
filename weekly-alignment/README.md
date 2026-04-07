# Weekly Alignment Scanner

A Claude Code skill that monitors your Slack channels for cross-team misalignment and delivers a Monday morning brief.

## What It Does

If you manage across multiple teams, you're in a dozen Slack channels and your reports are each in a dozen different ones. The overlaps and conflicts between those channels are where things go wrong — two teams start solving the same problem, or one team makes a decision that contradicts something another team is already building.

This skill hands that job to Claude. It scans your Slack channels weekly and flags:

- **Duplicate work** — two teams building the same thing without knowing
- **Conflicting decisions** — choices made in one channel that contradict another
- **Unaware stakeholders** — decisions that affect teams who weren't in the room
- **Resource conflicts** — same person or team pulled in multiple directions
- **Dependency gaps** — one team blocked on work another hasn't started

## Setup

### Prerequisites

- [Claude Code](https://claude.ai/download) (CLI, desktop app, or IDE extension)
- A Slack MCP server connected to your workspace ([setup guide](https://modelcontextprotocol.io))

### Install

1. Copy this `weekly-alignment` folder into your Claude Code plugins directory:
   ```
   ~/.claude/plugins/weekly-alignment/
   ```
   Or clone it into any project and Claude will discover it automatically.

2. Run the setup interview:
   ```
   /weekly-alignment-setup
   ```
   This takes ~5 minutes. Claude will ask about your teams, which Slack channels to monitor, and what kinds of misalignment tend to happen in your world. The more specific you are, the better the output.

3. Your answers get saved so the interview doesn't repeat. Run `/weekly-alignment-setup` again anytime to update.

### Run It

**Manually:**
```
run my weekly alignment check
```

**On a schedule (recommended):**
```
/schedule
```
Set it to run every Monday morning. Now you start each week with a brief on what's misaligned before it becomes a problem.

## How It Works

1. **Reads** your configured Slack channels (last 7 days)
2. **Extracts** decisions, new work, blockers, and commitments from each channel
3. **Cross-references** across all channels to find conflicts, overlaps, and gaps
4. **Applies** your org-specific misalignment patterns (the ones you described during setup)
5. **Delivers** a prioritized brief — HIGH / MEDIUM / LOW — with specific references and suggested actions

## Customization

The setup interview captures your org's specific patterns. If your world changes — new teams, new channels, new risks — just run `/weekly-alignment-setup` again.

The skill is intentionally specific: it watches for the things YOU told it matter, not generic "team health" metrics. That's why the setup interview matters.

## Example Output

```
WEEKLY ALIGNMENT BRIEF — Week of March 3–7

TL;DR: Two significant conflicts found. Platform and Product are building
overlapping caching solutions. Data team's schema migration timeline
conflicts with the Q2 feature launch.

HIGH PRIORITY

### Duplicate Caching Work
Teams involved: Platform, Product
What's happening: In #platform-eng on Tuesday, @alice said they're building
a Redis cache layer for API responses. In #product-dev on Wednesday, @bob
said they're evaluating caching options and leaning toward Memcached.
Neither thread references the other.
Why it matters: Two teams will build, test, and maintain separate caching
infrastructure for overlapping use cases.
Suggested action: Get Platform and Product leads in a room this week to
decide who owns caching.
```

## License

MIT — use it, modify it, share it.
