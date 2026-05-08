# BrightWay AI — Claude Plugin Marketplace

A curated library of Claude plugins that turn Claude into a daily ops layer for solo operators, consultants, and small teams. Memory, business development, daily planning, deliverable review, news curation, and cross-team alignment — each plugin works on its own, and they get sharper when paired together.

Compatible with **Cowork (Claude Desktop)** and **Claude Code**.

---

## Install

```
/plugin marketplace add BrightWayAI/claude-plugins
```

Then install plugins individually:

```
/plugin install claude-cortex@claude-plugins
/plugin install brightway-core@claude-plugins
/plugin install lead-engine@claude-plugins
... (etc — see "What's in the marketplace" below)
```

After installing, **run each plugin's `/setup-*` command** before using it for real. Setup interviews capture your context (CRM, ICP, voice, working hours, offerings, etc.) so the plugin adapts to *you* rather than running on generic defaults.

Updates flow automatically — push to a plugin's repo on GitHub → the marketplace picks it up on Cowork's next startup. No marketplace re-install needed for content changes.

---

## What's in the marketplace

Nine plugins, organized by what they do.

### Memory & knowledge

| Plugin | What it does | Setup |
|---|---|---|
| **[claude-cortex](https://github.com/BrightWayAI/claude-cortex)** | Always-on learning system. Passively observes preferences, captures knowledge, and adapts every conversation. The memory layer for everything else in this marketplace. | None — works out of the box |

**Hosts subagents:** `memory-librarian` (broad-query memory synthesis), `transcript-reviewer` (weekly Granola/call commitment delta).

### Business development & sales

| Plugin | What it does | Setup |
|---|---|---|
| **[lead-engine](https://github.com/BrightWayAI/lead-engine)** | Intent-based LinkedIn outbound. Catch buying signals, warm prospects, draft DMs in your voice, run a 3-touch cadence, generate pre-call briefs. | `/lead-setup` |
| **[bizdev-outreach](https://github.com/BrightWayAI/Biz-Dev)** | Research a contact across your CRM, email, and the web; draft personalized outreach in your voice. Knows when *not* to send. | `/setup` |
| **[weekly-outreach](https://github.com/BrightWayAI/weekly-outreach)** | Weekly relationship management and BD prep. Prioritized outreach queue (10–12 contacts), call prep for the week's external meetings, drafted messages, plus CRM tasks and calendar placeholders. | `/setup-outreach` |

**Hosts subagent:** `contact-researcher` (in lead-engine — single-contact deep dive across CRM, email, web).

### Marketing & content

| Plugin | What it does | Setup |
|---|---|---|
| **[news-curator](https://github.com/BrightWayAI/news-curator)** | Curate and draft a weekly LinkedIn news roundup post. Configurable per topic and audience — works for AI, climate, fintech, anything. | `/setup-news` |

**Hosts subagents:** `news-curator` (scan + rank), `post-assembler` (drafts in your voice).

### Daily operations

| Plugin | What it does | Setup |
|---|---|---|
| **[plan-tomorrow](https://github.com/BrightWayAI/plan-tomorrow)** | Calendar-first daily planning. Pulls CRM tasks, working memory, and inbox action items, then creates time blocks on your calendar with full context baked in. The calendar IS the plan. | `/setup-plan` |
| **[project-setup](https://github.com/BrightWayAI/project-setup)** | End-to-end client engagement initialization. Drive folder structure, Claude Project system prompt, phased project plan, memory node — all in one interview. | `/setup-projects` |

### Cross-team & deliverable QA

| Plugin | What it does | Setup |
|---|---|---|
| **[brightway-core](https://github.com/BrightWayAI/core)** | Shared business-ops toolkit. Pipeline analyst (subagent) ranks your CRM pipeline. Deliverable reviewer (slash command) runs structured QA on client decks/docs/spreadsheets against your brand. | `/setup-core` |
| **[weekly-alignment](https://github.com/BrightWayAI/weekly-alignment)** | Weekly cross-team alignment scanner for Slack. Surfaces overlapping initiatives, conflicting priorities, and decisions that affect other teams. Delivers a Monday morning brief. | `/setup` (via skills) |

**Hosts subagent:** `pipeline-analyst` (in brightway-core — CRM-agnostic pipeline ranking).

---

## How the plugins work together

Each plugin is independently useful, but several get sharper when paired. Subagents (specialist Claude agents that handle heavy research/analysis off the main conversation thread) are the connective tissue.

### Subagents and consumers

| Subagent | Lives in | Used by |
|---|---|---|
| `memory-librarian` | claude-cortex | `/search` (cortex) — broad cross-node queries |
| `transcript-reviewer` | claude-cortex | weekly call commitment audit |
| `contact-researcher` | lead-engine | `/bizdev-outreach`, `/lead-brief`, `/lead-pull`, `/weekly-outreach`, any "research [contact]" workflow |
| `pipeline-analyst` | brightway-core | `/weekly-outreach`, `/plan-tomorrow`, any pipeline-review workflow |
| `news-curator` + `post-assembler` | news-curator | `/ai-roundup` |

When a parent skill needs deep research, it delegates to a subagent rather than doing the research inline. This keeps the main conversation context clean and produces consistent structured output across skills.

### Recommended install combinations

**Solo consultant / founder running BD on Claude:**
```
claude-cortex + brightway-core + lead-engine + bizdev-outreach + weekly-outreach
```
Memory + pipeline analysis + signal-driven outbound + per-contact drafting + weekly prep.

**Agency operator with multiple client engagements:**
```
claude-cortex + brightway-core + project-setup + weekly-outreach + plan-tomorrow
```
Memory + pipeline + new-engagement onboarding + BD + daily calendar.

**Content-focused operator:**
```
claude-cortex + news-curator + bizdev-outreach
```
Memory + weekly LinkedIn roundup + per-contact outreach.

**Cross-team operator (manager / chief-of-staff):**
```
claude-cortex + weekly-alignment + plan-tomorrow + brightway-core
```
Memory + Slack alignment scan + daily calendar + deliverable QA.

**Minimum viable starter:** claude-cortex + brightway-core. Everything else builds on top.

---

## Setup flow

Most plugins ship with a `/setup-*` command that interviews you and saves your context to a `references/user-context.md` file (gitignored — never committed). Without setup, agents run with generic defaults. With setup, they're tuned to your firm.

| Plugin | Setup command | Captures |
|---|---|---|
| claude-cortex | (auto — works out of the box) | Memory layout configures itself per project |
| brightway-core | `/setup-core` | CRM, brand, voice, deliverable conventions |
| lead-engine | `/lead-setup` | Company, ICP, voice, value-adds, signal preferences |
| bizdev-outreach | `/setup` | Company, positioning, products, target market, voice |
| weekly-outreach | `/setup-outreach` | ICP, CRM custom properties, cadence tiers, weekly target |
| news-curator | `/setup-news` | Topic, audience, sources, voice, post format |
| plan-tomorrow | `/setup-plan` | Working hours, CRM, calendar conventions |
| project-setup | `/setup-projects` | Offerings catalog, drive layout, communication defaults |
| weekly-alignment | `/setup` | Slack channels, teams, risk patterns to watch |

**Don't skip setup.** All plugins return "run `/setup-*` first" if their context file is missing or empty.

---

## Develop / fork

Each plugin lives in its own GitHub repo. To customize one:

1. Fork the plugin repo (e.g., `BrightWayAI/lead-engine` → `yourname/lead-engine`).
2. Update `marketplace.json` in your marketplace fork (or create your own marketplace) to point at your repo instead.
3. Edit, commit, push. Cowork picks up your changes on next startup.

Templates inside plugins (e.g., `references/templates/` in `project-setup`) are intended to be edited per-firm. The starter content reflects BrightWay AI's offerings — replace it with your own.

---

## Issues & feedback

Each plugin manages its own issues:

- [claude-cortex](https://github.com/BrightWayAI/claude-cortex/issues)
- [lead-engine](https://github.com/BrightWayAI/lead-engine/issues)
- [bizdev-outreach](https://github.com/BrightWayAI/Biz-Dev/issues)
- [weekly-outreach](https://github.com/BrightWayAI/weekly-outreach/issues)
- [news-curator](https://github.com/BrightWayAI/news-curator/issues)
- [plan-tomorrow](https://github.com/BrightWayAI/plan-tomorrow/issues)
- [project-setup](https://github.com/BrightWayAI/project-setup/issues)
- [brightway-core](https://github.com/BrightWayAI/core/issues)
- [weekly-alignment](https://github.com/BrightWayAI/weekly-alignment/issues)

For marketplace-level issues (manifest problems, install errors, discovery), use [BrightWayAI/claude-plugins](https://github.com/BrightWayAI/claude-plugins/issues).

---

## License

Each plugin is MIT-licensed. See the `LICENSE` file in each repo.
