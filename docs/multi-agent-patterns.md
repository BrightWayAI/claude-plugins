# Multi-Agent Orchestration Patterns

Patterns for chaining subagents inside a single skill or slash command. Use this when a workflow needs research → analyze → synthesize → draft → review across multiple specialist agents.

This is a pattern doc for plugin authors, not a runtime artifact. The patterns here are already used implicitly in `news-curator/ai-roundup` (news-curator → user-pick → post-assembler) and `weekly-outreach` (pipeline-analyst → contact-researcher per top-N). Documenting them so future plugins can use the same shape consistently.

---

## Why orchestration matters

Single-agent invocations work for narrow tasks: "research this contact," "synthesize across memory." Real workflows are chains: "find candidates → rank them → let the user pick → research the picks → draft from the research → review the draft."

Doing this inline in a skill bloats the parent's context with raw data at every step. Doing it as a chain of subagents keeps each step's context clean and produces structured handoffs the parent can render and gate.

---

## The canonical pattern

### Shape

```
Skill (parent context)
  ↓
  Step A: Delegate to Agent 1 (e.g., research / scan)
  ↓
  Returns structured output A
  ↓
  Step B: User gate (review, pick, edit) — OPTIONAL
  ↓
  Step C: Delegate to Agent 2 (e.g., synthesize / draft) with output A as input
  ↓
  Returns structured output B
  ↓
  Step D: User gate — OPTIONAL
  ↓
  Step E: Delegate to Agent 3 (e.g., review / verify) — OPTIONAL
  ↓
  Skill renders final result
```

### Rules

1. **The parent skill orchestrates. Agents don't know about each other.** Each agent is invoked independently with a self-contained brief. The parent passes Agent 1's output as input to Agent 2 — Agent 2 doesn't fetch it directly.

2. **User gates between agents are valuable.** Don't run the whole chain headless. Insert "show the user, ask for direction" between steps where editorial judgment matters. Examples: "which candidates to keep?" "is this the angle?" "ship it or iterate?"

3. **Each agent's return contract is fixed and parsed by the parent.** When Agent 1 returns a 10-item ranked list, the parent knows the shape (because Agent 1's spec defines it). The parent can render the list, let the user pick a subset, and pass only the subset to Agent 2.

4. **Confidence flows through the chain.** If Agent 1 returns Low confidence, the parent should consider whether to:
   - Pause and ask the user for context (re-run Agent 1 with a richer brief)
   - Skip Agent 2 and report the gap directly
   - Proceed with the chain but flag the limitation in the final output

   See "Confidence-aware delegation" pattern in any consumer skill (e.g., `bizdev-outreach`, `lead-brief`, `weekly-outreach`).

5. **Don't re-invoke an agent for the same brief twice.** Cache outputs in the conversation context; reference them by structure rather than re-querying. Agents are expensive — re-running them in a chain step that already has the data is waste.

---

## Examples in the marketplace

### `news-curator/ai-roundup` — three-step chain

```
/ai-roundup
  ↓
  Step 2: news-curator agent → top 10 candidates with scores + themes
  ↓
  Step 3: USER GATE — show candidates, user picks 5-7
  ↓
  Step 4: post-assembler agent → drafted post + first-comment + alternate hook
  ↓
  Step 5: USER GATE — show draft, user iterates ("ship it" / "shorter" / "redo with X")
  ↓
  Step 6: Skill writes final to runs/[date].md
```

**Why this shape:** scanning is expensive (web fetches), drafting is voice-sensitive. The user gate after scanning lets editorial judgment shape the input to drafting. Without the gate, the post would draft from whatever the agent ranked highest — which is fine, but loses voice control.

### `weekly-outreach` — fan-out chain

```
/weekly-outreach
  ↓
  Step 1: contact-researcher agent (per upcoming external call) → call prep
  ↓
  Step 2: pipeline-analyst agent → ranked outreach queue
  ↓
  Step 3 (optional): Apollo enrichment for net-new prospects
  ↓
  Step 4 (optional): contact-researcher agent (per top 3-5 in queue) → deeper dossier
  ↓
  Skill drafts messages from queue + dossiers
  ↓
  USER GATE — show full plan, user reviews
  ↓
  Step 7-8: Create CRM tasks + calendar placeholders after approval
```

**Why this shape:** pipeline analysis is the spine, contact research deepens individual contacts. The fan-out (research the top N) is parallelizable in principle; sequential in practice because token budgets limit how many agent calls we want per skill.

### `bizdev-outreach` — single agent, parent-driven analysis

```
Skill (bizdev-outreach SKILL)
  ↓
  Phase 1: contact-researcher agent → dossier
  ↓
  Phase 2: Skill analyzes dossier (parent-context — no agent)
  ↓
  Phase 3: Skill drafts message (parent-context — no agent)
```

**Why this shape:** drafting is voice-faithful and context-heavy; doing it in the parent context (which knows the user's voice rules from `~/Documents/Claude/voice.md`) avoids the round-trip of passing voice rules to a drafter agent. Single-agent chain.

---

## Anti-patterns

### Don't: chain agents that need each other's tools

If Agent 1 needs HubSpot and Agent 2 needs HubSpot, two separate invocations is fine. But don't construct a chain where Agent 2 needs Agent 1's specific tool *output*; instead, have one agent that does both, or have Agent 1 return data structured enough that Agent 2 doesn't need the tool.

### Don't: deep chains (4+ agents)

Two-agent chains are clean. Three-agent chains are workable. Four-agent chains are over-engineered — the orchestration overhead exceeds the per-agent value. If you're considering a 4-agent chain, it's usually a sign that one of the agents should be split into two skills or merged into a single more capable agent.

### Don't: silent chains

Always render intermediate results to the user, even if briefly. "Step 1 returned 10 candidates, top 3 were X, Y, Z. Now drafting." gives the user a chance to interrupt if something's off. Silent chains feel like black boxes and erode trust.

### Don't: agents that call agents

Agents inherit parent tools. They don't (currently) invoke other agents. If you find yourself wanting Agent 1 to invoke Agent 2 mid-flight, restructure: have the parent skill orchestrate both, or merge them into one agent with broader scope.

---

## Confidence handling in chains

When chaining, confidence compounds. Three agents at Medium confidence each → final output is *less* than Medium because errors stack.

Pattern:

```
After each agent returns:
  - If Confidence >= Medium: continue
  - If Confidence == Low AND critical sections empty: pause, surface gap, ask user
  - If Confidence == Low AND critical sections OK: continue but note limitation in final output
```

The "critical sections" depend on the agent. For contact-researcher, critical = Contact Snapshot + Relationship History + at least one talking point. For pipeline-analyst, critical = the top-10 priority items (not the risks or pipeline-health snapshot, which can be sparse without breaking the queue). Each parent skill defines what's critical for its workflow.

---

## When NOT to chain — just use one agent

If a workflow can be done with a single agent invocation followed by inline drafting/synthesis in the parent context, do that. Chains add coordination cost. The cost is worth paying when:

- Different specialist tools are involved (research vs. drafting vs. review)
- User gates between steps materially shape the output (editorial judgment)
- The chain produces a meaningfully better result than a single big agent

Otherwise: one agent, one skill, done.
