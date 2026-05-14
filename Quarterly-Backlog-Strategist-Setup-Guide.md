# Quarterly-Backlog-Strategist Setup Guide

## Quarterly Backlog Strategist — Setup Guide

This guide explains how to extend an existing GitHub Copilot workspace with a quarterly backlog strategy capability using repository instructions, reusable prompt files, and workspace planning memory. VS Code supports reusable `.prompt.md` files for Copilot workflows, while repository instructions remain the main shared behavior layer for the workspace.

---

## Purpose

The goal of this setup is to let GitHub Copilot analyze a large Jira backlog and recommend strategic next-quarter actions based on sprint reality, quarter priorities, and annual business goals. The setup is designed to fit inside an existing workspace ambient rather than replace it, which aligns with how repository instructions and prompt files are intended to complement each other.

---

## Recommended Folder Structure

The following structure should be added to the existing workspace so the new planning capability lives inside the current operating model rather than as a separate AI environment.

```
.github/
  copilot-instructions.md
  prompts/
    quarterly-backlog-strategist.prompt.md
    backlog-scan.prompt.md
    theme-clustering.prompt.md
    next-quarter-plan.prompt.md
  instructions/
    quarterly-planning.instructions.md

[Team Folder]/
  Inputs/
  Outputs/
  AI/
    Memory/
      current-quarter.md
      annual-goals.md
      planning-preferences.md
      domain-glossary.md
    Templates/
      next-quarter-plan-template.md
```

---

## Structure Rationale

| Folder | Purpose |
|--------|---------|
| `.github/prompts/` | Reusable Copilot prompt files for repeatable task workflows with optional frontmatter (`description`, `mode`) |
| `.github/instructions/` | Narrower instruction files that reinforce a specific task pattern without replacing the main repository instruction file |
| `[Team Folder]/AI/Memory/` | Durable planning context (sprint state, annual goals, glossary, preferences) that grounds the agent in business context |
| `[Team Folder]/AI/Templates/` | Reusable output structures for consistent plan formatting across runs |

---

## How It Fits the Existing Workspace

The existing `.github/copilot-instructions.md` should remain the primary workspace brain because repository instructions are meant to define broad, shared behavior for the whole repo. The new quarterly planning capability should be added by appending one dedicated extension section and by creating prompt files, rather than by replacing the current instruction model.

---

## What to Append to the Existing Instructions File

Append the following block to the end of the existing `.github/copilot-instructions.md` file:

```
## Quarterly Planning Agent Extension

This workspace also supports quarterly backlog strategy analysis for the [PROJECT_KEY] team.

### Purpose
Use reusable prompt files in `.github/prompts/` to analyze Jira backlog data and recommend strategic next-quarter actions based on:
- current sprint context
- current quarter priorities
- annual strategic goals
- active delivery constraints

### Planning context files
When running backlog planning or quarterly prioritization tasks, use:
- `[Team Folder]/AI/Memory/current-quarter.md`
- `[Team Folder]/AI/Memory/annual-goals.md`
- `[Team Folder]/AI/Memory/planning-preferences.md`
- `[Team Folder]/AI/Memory/domain-glossary.md`
- `[Team Folder]/AI/Templates/next-quarter-plan-template.md`

### Quarterly planning rules
- Treat backlog analysis as strategic synthesis, not ticket summarization.
- Separate:
  - current sprint commitments
  - current quarter commitments
  - next-quarter candidate work
  - stale / duplicate / orphaned / low-alignment work
- Prefer theme-level recommendations over long ticket-by-ticket outputs.
- Explicitly identify work to commit, discover first, merge/reframe, defer, close, or escalate.
- Protect current sprint commitments unless there is a strong strategic or delivery reason not to.
- Do not create or update Jira issues unless explicitly asked and confirmed by the user.
- If strategic context is missing, say so and treat the recommendation as provisional.

### Output behavior for planning analyses
- Write planning outputs locally first before any Confluence action.
- Store generated planning outputs under `[Team Folder]/Outputs/`.
- Update `[Team Folder]/Outputs/index.md` and append to `[Team Folder]/Outputs/log.md` after creating a planning output file.
```

---

## File Descriptions

### `.github/prompts/quarterly-backlog-strategist.prompt.md`
The main orchestration prompt. End-to-end workflow that reads planning context, checks Jira and Confluence context through Atlassian MCP, analyzes the backlog, groups tickets into strategic themes, and produces a next-quarter recommendation set.

### `.github/prompts/backlog-scan.prompt.md`
The diagnostic first pass. Surfaces issue volume, stale work, likely carryover, blockers, duplicates, and noisy backlog patterns before prioritization begins.

### `.github/prompts/theme-clustering.prompt.md`
Converts many individual tickets into a smaller set of strategic themes or initiative candidates. Especially useful when the backlog is messy or inconsistent and needs a product strategy lens.

### `.github/prompts/next-quarter-plan.prompt.md`
Turns backlog findings into a recommendation set for the next quarter. Focuses on what to commit, discover first, merge or reframe, defer, close, or escalate based on sprint context, quarter context, and annual goals.

### `.github/instructions/quarterly-planning.instructions.md`
Scoped instruction layer for backlog planning tasks. Reinforces planning-specific rules (sprint protection, strategic synthesis, low-confidence signaling, use of planning memory files) without conflicting with the main repository instruction file.

### `[Team Folder]/AI/Memory/current-quarter.md`
Stores the current quarter operating context: active sprint, quarter priorities, committed initiatives, deadlines, and delivery constraints.

### `[Team Folder]/AI/Memory/annual-goals.md`
Stores annual strategic goals and defines what strong and weak alignment look like.

### `[Team Folder]/AI/Memory/planning-preferences.md`
Stores preferred planning style, heuristics, risk posture, and output preferences.

### `[Team Folder]/AI/Memory/domain-glossary.md`
Explains internal team names, acronyms, status meanings, label conventions, and backlog quirks.

### `[Team Folder]/AI/Templates/next-quarter-plan-template.md`
Defines the standard final structure for the planning output.

---

## Full File Contents

### `.github/instructions/quarterly-planning.instructions.md`

```
---
applyTo: "**/*"
description: Instructions for quarterly backlog planning and strategic backlog analysis tasks
---
```

```
Apply these instructions when the task is backlog analysis, quarterly planning, prioritization strategy, or next-quarter recommendation work.

- Use workspace conventions from `.github/copilot-instructions.md` as the primary rule set.
- Use:
  - `[Team Folder]/AI/Memory/current-quarter.md`
  - `[Team Folder]/AI/Memory/annual-goals.md`
  - `[Team Folder]/AI/Memory/planning-preferences.md`
  - `[Team Folder]/AI/Memory/domain-glossary.md`
  - `[Team Folder]/AI/Templates/next-quarter-plan-template.md`
- Prefer strategic synthesis over item-by-item summaries.
- Separate current sprint work from current-quarter and next-quarter planning candidates.
- Use annual goals and quarter priorities as the primary prioritization lens.
- Treat stale backlog as signal, not just clutter.
- Explicitly identify what should be deferred, closed, or escalated.
- Protect active sprint work unless there is a strong business reason to intervene.
- Flag weak metadata and low-confidence inferences.
- Write outputs locally first under `[Team Folder]/Outputs/` before any Confluence action.
```

---

### `.github/prompts/quarterly-backlog-strategist.prompt.md`

```
---
description: Analyze [PROJECT_KEY] backlog against sprint, quarter, and annual strategy, then recommend next-quarter actions
mode: agent
---
```

```
# Quarterly Backlog Strategist

Act as a strategic backlog planning agent for [Owner] in the [PROJECT_KEY] workspace.

Your task is to analyze a Jira backlog that may contain hundreds of tickets across mixed ages, statuses, priorities, and quality levels, then recommend the best strategic next steps for the next quarter.

You must anchor your analysis in:
- the current sprint
- the current quarter
- annual strategic goals
- delivery reality and constraints

## Required context
Before forming recommendations, use these local files if they exist:
- [current quarter](../../[Team Folder]/AI/Memory/current-quarter.md)
- [annual goals](../../[Team Folder]/AI/Memory/annual-goals.md)
- [planning preferences](../../[Team Folder]/AI/Memory/planning-preferences.md)
- [domain glossary](../../[Team Folder]/AI/Memory/domain-glossary.md)
- [next quarter plan template](../../[Team Folder]/AI/Templates/next-quarter-plan-template.md)

Also use Jira and Confluence context available through Atlassian MCP, following all safety and confirmation rules in `.github/copilot-instructions.md`.

## Your objectives
1. Understand current execution context before recommending next-quarter priorities.
2. Separate current sprint work from broader backlog inventory.
3. Group backlog items into themes, initiative candidates, or strategic workstreams.
4. Identify:
   - stale but important work
   - current-quarter carryover risk
   - low-alignment or low-value work
   - duplicate or overlapping work
   - blocked or dependency-heavy work
   - orphaned work with weak strategic linkage
5. Produce next-quarter recommendations in clear action categories:
   - Commit
   - Discover first
   - Merge / reframe
   - Defer
   - Close
   - Escalate

## Operating rules
- Do not optimize for volume of output; optimize for decision quality.
- Prefer theme-level synthesis over listing individual tickets.
- Use specific tickets only as evidence or examples.
- Protect current sprint commitments unless there is material urgency or major strategic conflict.
- If backlog metadata is weak, infer carefully and state confidence.
- If strategy context is missing, explicitly say the recommendation is provisional.
- Make trade-offs explicit.
- Always say what should not be prioritized next quarter.
- Never post to Confluence and never change Jira state unless explicitly asked and confirmed.

## Suggested analysis sequence
1. Summarize current sprint and current quarter context.
2. Retrieve or review the relevant backlog scope.
3. Break the backlog into meaningful clusters:
   - by epic
   - by product area
   - by theme
   - by workflow pattern
   - by strategic relevance
4. Diagnose the backlog:
   - where the work is concentrated
   - what is aging
   - what is stuck
   - what is likely to carry over
   - what appears misaligned
5. Score themes qualitatively using:
   - strategic alignment
   - impact
   - urgency
   - readiness
   - cost of delay
   - confidence
6. Recommend next-quarter actions.
7. Format the output using the template if available.
8. If asked to save the analysis, write it first to `[Team Folder]/Outputs/`, update `index.md`, and append to `log.md`.

## Output requirements
Return the result in this structure:

# Executive summary

# Current-state diagnosis

# Strategic themes

Use a table with columns:
Theme | Evidence | Annual goal alignment | Quarter relevance | Readiness | Recommendation

# Recommended next-quarter actions

Use a table with columns:
Initiative / Theme | Action | Why now | Dependencies / Risks | Confidence

# Defer / close

Use a table with columns:
Item / Theme | Recommendation | Rationale

# Risks and assumptions

# Decisions needed

## Important
If the user has not specified the board, filter, project, or product area, ask for that first before doing deep analysis.
```

---

### `.github/prompts/backlog-scan.prompt.md`

```
---
description: Scan a [PROJECT_KEY] backlog and return a diagnostic view before prioritization
mode: agent
---
```

```
# Backlog Scan

Act as a backlog diagnostic analyst for the [PROJECT_KEY] workspace.

Your goal is to scan the specified Jira backlog and produce a structured diagnosis before any strategic recommendation is made.

## Use this context
- [current quarter](../../[Team Folder]/AI/Memory/current-quarter.md)
- [domain glossary](../../[Team Folder]/AI/Memory/domain-glossary.md)
- Jira context from Atlassian MCP

## What to do
1. Retrieve the requested backlog scope.
2. Separate current sprint work from non-sprint backlog.
3. Summarize:
   - issue volume by status
   - volume by epic / theme if available
   - old unresolved tickets
   - recently updated tickets
   - blocked items
   - orphaned items
   - duplicate-looking clusters
4. Highlight patterns that matter for quarterly planning:
   - stale but likely important
   - current-quarter carryover risk
   - busywork crowding the backlog
   - weakly described items
   - fragmented work across too many small tickets

## Output format
Return:
- Scope analyzed
- Backlog health summary
- Key patterns
- Risks
- Items needing cleanup before planning

Use concise bullets and one compact table where useful.
```

---

### `.github/prompts/theme-clustering.prompt.md`

```
---
description: Cluster [PROJECT_KEY] backlog items into strategic themes or initiative candidates
mode: agent
---
```

```
# Theme Clustering

Act as a product strategy analyst for the [PROJECT_KEY] workspace.

Your goal is to convert a noisy Jira backlog into a small number of useful strategic themes or initiative candidates.

## Inputs
Use:
- Jira issue context
- [annual goals](../../[Team Folder]/AI/Memory/annual-goals.md)
- [current quarter](../../[Team Folder]/AI/Memory/current-quarter.md)
- [domain glossary](../../[Team Folder]/AI/Memory/domain-glossary.md)

## Instructions
1. Review the backlog set provided in context.
2. Group work into themes or initiative candidates.
3. For each theme:
   - summarize what it represents
   - identify example issues or epics as evidence
   - estimate likely strategic goal alignment
   - identify whether the work is delivery, discovery, maintenance, compliance, operational support, or growth-oriented
   - flag low confidence where metadata is weak
4. Merge overlapping themes where sensible.
5. Keep the number of themes small enough to support executive planning.

## Output
Use a table with columns:
Theme | What it includes | Example evidence | Strategic alignment | Confidence

Then add:
- Overlaps to merge
- Themes that appear weak or non-strategic
- Themes likely suitable for next-quarter planning
```

---

### `.github/prompts/next-quarter-plan.prompt.md`

```
---
description: Turn [PROJECT_KEY] backlog themes into a next-quarter recommendation set
mode: agent
---
```

```
# Next Quarter Plan

Act as a quarterly portfolio planning advisor for the [PROJECT_KEY] workspace.

Use backlog analysis, current sprint context, current-quarter commitments, and annual goals to recommend what should happen next quarter.

## Context to use
- [current quarter](../../[Team Folder]/AI/Memory/current-quarter.md)
- [annual goals](../../[Team Folder]/AI/Memory/annual-goals.md)
- [planning preferences](../../[Team Folder]/AI/Memory/planning-preferences.md)
- [template](../../[Team Folder]/AI/Templates/next-quarter-plan-template.md)

## Instructions
Build a next-quarter recommendation set that:
- protects current sprint commitments unless there is a strong reason not to
- accounts for likely current-quarter carryover
- prioritizes a short list, not everything
- explicitly identifies what to defer, close, or reframe
- surfaces leadership trade-offs and decisions

## Action categories
Use only these recommendation labels:
- Commit
- Discover first
- Merge / reframe
- Defer
- Close
- Escalate

## Decision logic
- High alignment + high readiness → Commit
- High alignment + low readiness → Discover first
- Medium alignment + fragmented demand → Merge / reframe
- Low alignment + low urgency → Defer
- Low value + stale + weak ownership → Close
- High impact but unresolved trade-off → Escalate

## Output
Use the template if available. Otherwise return:

# Executive summary

# Recommended next-quarter focus

# Recommendation table
Columns:
Theme | Action | Why | Risks / Dependencies | Confidence

# Defer / close

# Risks and assumptions

# Decisions for leadership
```

---

### `[Team Folder]/AI/Memory/current-quarter.md`

```
# Current quarter context

Quarter: Q[X] [YEAR]
Current sprint: [Sprint name/number]
Quarter dates: [Start] – [End]

## Quarter priorities
- [Priority 1]
- [Priority 2]
- [Priority 3]

## Committed initiatives
- [Initiative 1]
- [Initiative 2]
- [Initiative 3]

## Known deadlines and commitments
- [Date] — [Commitment]
- [Date] — [Commitment]

## Delivery constraints
- [Constraint — e.g., limited headcount across multiple P0 initiatives]
- [Dependency — e.g., external team blocker on critical-path work]
- [Risk — e.g., work concentration in a single workstream]

## Current sprint notes
- Sprint goal:
- Major in-flight items:
- Protected commitments:
- Areas where reprioritization would be disruptive:
```

---

### `[Team Folder]/AI/Memory/annual-goals.md`

```
# Annual company goals

Year: [YEAR]

## Strategic goals
1. [Goal 1 — e.g., Revenue growth through digital channel optimization]
2. [Goal 2 — e.g., Platform reliability and operational efficiency]
3. [Goal 3 — e.g., Customer experience elevation]
4. [Goal 4 — e.g., AI/Data as productivity amplifier]

## What strong alignment looks like
- Revenue growth
- Retention / expansion
- Customer experience
- Platform reliability
- Operational efficiency
- Compliance / contractual obligation
- Strategic client delivery

## What weak alignment looks like
- Isolated backlog noise with no measurable outcome
- Nice-to-have requests with no strategic sponsor
- Work that consumes delivery capacity without moving a current-year goal
- Legacy carryover with no validated reason to continue

## Trade-off guidance
- Prefer [critical-path delivery] over [exploratory work] when capacity is constrained
- Protect [platform stability] even if feature work slips
- Avoid starting [low-alignment initiatives] unless linked to a top-tier goal
```

---

### `[Team Folder]/AI/Memory/planning-preferences.md`

```
# Planning preferences

## Planning stance
- Prefer initiative-level recommendations over ticket-level commentary
- Optimize for strategic clarity, not exhaustive detail
- Protect current sprint commitments unless there is a compelling reason to intervene
- Assume capacity is finite and force prioritization
- Always identify what should not be done next quarter

## Backlog heuristics
- Flag tickets older than 90 days unless strategically justified
- Treat vague tickets with low confidence
- Separate delivery from discovery
- Highlight duplicate or overlapping effort
- Call out work that lacks strategic linkage

## Output preferences
- Start with an executive summary
- Use compact decision tables
- Be explicit about trade-offs
- Include confidence / assumptions
- Include "decisions needed" when human judgment is required

## Risk preferences
- Prefer explainable recommendations over aggressive automation
- Surface carryover risk early
- Be cautious with recommendations that would disrupt active sprint commitments
```

---

### `[Team Folder]/AI/Memory/domain-glossary.md`

```
# Domain glossary

## Teams
- [Team 1] — [Description of responsibility]
- [Team 2] — [Description of responsibility]
- [Team 3] — [Description of responsibility]

## Product areas
- [Area 1] — [What it covers]
- [Area 2] — [What it covers]
- [Area 3] — [What it covers]

## Common acronyms
- PI — Program Increment
- DM — Delivery Manager
- [Add your own project/org-specific acronyms]

## Jira status meanings
- To Do — Not yet started
- In Progress — Active delivery work
- In Review — Usually waiting on review, validation, or stakeholder input
- Blocked — Cannot progress without dependency resolution
- Done — Completed; normally excluded from planning scans by default

## Label / tag interpretations
- [label-1] — [What it means in your context]
- [label-2] — [What it means in your context]

## Known backlog quirks
- "In Review" may hide real dependency delay rather than near-complete work
- Some legacy tickets may predate current strategic direction
- Discovery tickets may mix research, scoping, and delivery preparation
- Not all tickets are consistently linked to epics or strategic goals
- [Add your own known quirks]
```

---

### `[Team Folder]/AI/Templates/next-quarter-plan-template.md`

```
# Next Quarter Plan

## Executive summary
- Recommended focus:
- Main trade-off:
- Biggest risk:
- Overall planning confidence:

## Current-state diagnosis
- Backlog shape:
- Current sprint protection:
- Current-quarter carryover risk:
- Main signal from the backlog:

## Strategic themes
| Theme | Evidence | Annual goal alignment | Quarter relevance | Readiness | Recommendation |
|---|---|---|---|---|---|

## Recommended next-quarter actions
| Initiative / Theme | Action | Why now | Dependencies / Risks | Confidence |
|---|---|---|---|---|

## Defer / close
| Item / Theme | Recommendation | Rationale |
|---|---|---|

## Risks and assumptions
- [Risk / assumption]
- [Risk / assumption]

## Decisions needed
- [Decision]
- [Decision]
```

---

## One-Shot Setup Prompt

The following prompt can be pasted into Copilot Chat to create the full setup and populate all files while preserving the existing workspace ambient:

```
Extend the current workspace by adding a quarterly backlog strategy capability inside the existing ambient.

Important rules:
- Do not replace the existing `.github/copilot-instructions.md`.
- Keep all existing content in `.github/copilot-instructions.md`.
- Append a new section at the end called `## Quarterly Planning Agent Extension`.
- Create exactly the files listed in this guide and populate them with the provided content.
- Do not create any extra files.
- Do not modify existing agents under `.github/agents/`.
- Replace all `[PROJECT_KEY]` with the actual Jira project key.
- Replace all `[Team Folder]` with the actual team folder name.
- Replace all `[Owner]` references with the actual owner name and role.

After making these changes, show me:
1. the list of files created
2. confirmation that `.github/copilot-instructions.md` was appended, not replaced
3. a short summary of what each new file is for
```

---

## Suggested First Run

After setup, fill in the current quarter and annual goals files first, then invoke the strategist on a bounded backlog scope rather than the entire universe of issues. A bounded first pass usually produces cleaner clustering and more reliable recommendations.

A practical first run prompt:

```
/quarterly-backlog-strategist

Analyze the [PROJECT_KEY] active backlog for next-quarter planning.

Use:
- current sprint context
- current quarter priorities
- annual goals
- the active board or relevant filter

Do not create or update Jira tickets.
Return a recommendation only.
If the scope is too large, first summarize by status, age, epic, and theme.
```

---

## Customization Checklist

Before using this setup, replace all placeholders:

| Placeholder | Replace With |
|-------------|--------------|
| `[PROJECT_KEY]` | Your Jira project key |
| `[Team Folder]` | Your team's workspace folder name |
| `[Owner]` / `[Owner Name, Role]` | The person responsible for planning |
| `[YEAR]` | Current planning year |
| Strategic goals in `annual-goals.md` | Your organization's actual annual goals |
| Teams in `domain-glossary.md` | Your actual team names and responsibilities |
| Product areas in `domain-glossary.md` | Your actual product areas |
