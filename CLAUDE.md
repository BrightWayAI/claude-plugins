# Claude Code project notes — BrightWayAI plugin marketplace

This is the marketplace repo for the BrightWayAI plugin catalog. The actual plugin source code lives in separate GitHub repos under `BrightWayAI/*`; this repo only holds the catalog manifest (`.claude-plugin/marketplace.json`), the marketplace README, contribution docs, and cross-cutting design proposals.

## Layout

```
.claude-plugin/marketplace.json   ← the catalog (12 plugins)
README.md                          ← user-facing marketplace overview
CONTRIBUTING.md                    ← contributor guide for any plugin
SECURITY.md / LICENSE              ← standard repo files
docs/
├── multi-agent-patterns.md        ← pattern guide for chaining subagents inside plugins
└── proposals/                     ← future work / design specs awaiting pickup
    └── second-brain-extension.md  ← entity wiki pages, daily brief, /listen archive, etc.
```

## When picking up work on plugins

**Pending work lives in `docs/proposals/`.** Before starting any new feature, check that folder for an existing spec. Each proposal is a self-contained brief with motivation, architecture, per-plugin changes, implementation steps, and acceptance criteria — write was designed to be picked up cold by a future Claude Code session.

Currently open proposals:

- **`docs/proposals/second-brain-extension.md`** — entity wiki pages in cortex, interactive daily brief plugin, two-stage triage for memory commits, nightly `/listen` archive pipeline, `/end-day` orchestration chain. Based on Omar Ismail's second-brain pattern, adapted to this marketplace's existing stack.

## How plugins are organized

The marketplace contains 12 plugins, each in its own GitHub repo:

| Plugin | Repo | Purpose |
|---|---|---|
| claude-cortex | BrightWayAI/claude-cortex | Always-on memory + shared `/setup-identity` and `/setup-voice` |
| core-ops | BrightWayAI/core-ops | Pipeline analyst + forecast subagents, /diagnose, /log-agent-run, /agent-metrics, /register-schedules |
| lead-engine | BrightWayAI/lead-engine | LinkedIn intent-based outbound + contact-researcher subagent |
| bizdev-outreach | BrightWayAI/Biz-Dev | Per-contact research + drafted outreach |
| weekly-outreach | BrightWayAI/weekly-outreach | Weekly BD prep |
| referral-engine | BrightWayAI/referral-engine | Connector network + referral asks |
| news-curator | BrightWayAI/news-curator | Weekly LinkedIn AI roundup (news-curator + post-assembler subagents) |
| plan-tomorrow | BrightWayAI/plan-tomorrow | Calendar-first daily planning |
| client-status | BrightWayAI/client-status | Weekly client status drafts |
| project-setup | BrightWayAI/project-setup | New-engagement initialization |
| time-tracking | BrightWayAI/time-tracking | Calendar → time-log → invoices |
| weekly-alignment | BrightWayAI/weekly-alignment | Slack cross-team alignment scanner |

Each plugin has its own repo with `commands/`, `skills/`, `agents/`, `references/`, `CHANGELOG.md`, `SECURITY.md`, `LICENSE`. Local dev clones live as siblings to this marketplace repo in `~/lab-bench/`.

## Config-root convention

All plugins read user-specific config from `<config-root>/`, where `<config-root>` is the path stored in `~/Documents/.claude-plugin-config-root` (a single-line text file in the user's home directory, populated on first plugin setup). Layout:

```
<config-root>/
├── identity.md                                    (cortex /setup-identity)
├── voice.md                                       (cortex /setup-voice)
├── memory/                                        (cortex working memory)
├── plugins/
│   └── <plugin>.user-context.md                   (per-plugin config)
├── archive/YYYY-MM-DD/                            (raw daily archive — proposed in second-brain-extension)
├── daily-briefs/YYYY-MM-DD.md                     (interactive daily working doc — proposed)
└── time-log.csv                                   (time-tracking plugin)
```

This is the canonical write location for plugin runtime data. Don't write plugin runtime state anywhere else.

## Versioning

- Plugins are individually versioned in their own `plugin.json`.
- Marketplace itself doesn't track per-plugin versions (Cowork resolves the latest from each plugin's GitHub repo on next pull).
- Bump pattern: minor for new commands/agents, patch for fixes or non-breaking refactors, major for breaking API or config-layout changes.

## Recently completed work

- **v0.2 refactor (2026-05-11):** all 12 plugins migrated from plugin-folder-relative paths to `<config-root>/plugins/` paths, eliminating writes to Cowork's read-only mount. See each plugin's CHANGELOG for the v0.2.0 / v0.2.1 entries.
