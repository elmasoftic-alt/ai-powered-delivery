# PMO-Team-Skill Setup Guide

## What This Is

The `ring-pmo-team` plugin provides **6 specialized PMO subagents** for portfolio-level management inside a GitHub Copilot / Claude Code workspace. Each subagent is a dedicated AI instance with its own context window, specialized system prompt, and restricted tool access.

This guide covers:
- What the 6 subagents do
- What the 9 bundled workflows do
- How to dispatch them
- When to use PMO vs single-project tools
- Governance guardrails and blocker criteria

---

## How Subagents Work

```
You: "I need portfolio health assessment. Dispatch portfolio-manager."
  ↓
Claude Code spawns: ring:portfolio-manager (isolated context, specialized prompt)
  ↓
Agent analyzes: multi-project coordination, strategic alignment, capacity
  ↓
Returns: Markdown report with findings, risks, and recommendations
```

1. You dispatch a subagent by name
2. A new AI instance spawns with its own ~200k token context and specialized system prompt
3. The subagent works independently using its domain expertise
4. Results return as markdown analysis, structured reports, or recommendations
5. You can ask follow-up questions or dispatch additional agents in parallel

---

## 6 PMO Specialist Subagents

| Agent | Focus | Use When |
|-------|-------|----------|
| `portfolio-manager` | Multi-project coordination, strategic alignment, portfolio health scoring, prioritization frameworks | You need to assess how projects work together, identify misalignments, plan portfolio priorities |
| `resource-planner` | Capacity planning, skills matrix, allocation optimization, conflict resolution, team load balancing | You need to plan team assignments, resolve resource conflicts, assess capacity for new work |
| `governance-specialist` | Gate reviews, compliance assessment, process adherence, audit readiness, policy enforcement | You need to assess gate readiness, ensure process compliance, audit project governance |
| `risk-analyst` | RAID log aggregation, portfolio risk correlation, mitigation strategy, dependency-based risk analysis | You need portfolio-level risk assessment, cross-project risk correlation, mitigation strategies |
| `executive-reporter` | Portfolio status dashboards, executive summaries, board-ready findings, KPI dashboards | You need executive summaries, board packages, portfolio dashboards, strategic briefings |
| `delivery-reporter` | Git analysis, delivery metrics, squad showcases, visual HTML presentations, release summaries | You need delivery reports, squad showcases, release summaries, performance metrics |

---

## 9 Supporting Workflows

Workflows bundle agents together or provide guided analysis. Use them when you want a pre-defined process instead of custom dispatch.

| Workflow | Agents Used | Output |
|----------|-------------|--------|
| `portfolio-review` | portfolio-manager + resource-planner + risk-analyst + governance-specialist (parallel) | Comprehensive report: strategic alignment, capacity, risks, compliance |
| `executive-summary` | executive-reporter | Board-ready summary, KPI dashboards, strategic briefing |
| `portfolio-planning` | portfolio-manager | Strategic plan, capacity forecast, optimization recommendations |
| `resource-allocation` | resource-planner | Capacity models, allocation matrices, conflict resolution options |
| `risk-management` | risk-analyst | RAID log, risk correlation, mitigation strategies |
| `project-health-check` | portfolio-manager + risk-analyst | Health scorecard: strategic fit, resource status, risks, blockers |
| `dependency-analysis` | portfolio-manager | Dependency map, critical paths, integration risks |
| `pmo-retrospective` | portfolio-manager + executive-reporter | Lessons learned, process improvements, next-cycle recommendations |
| `using-pmo-team` | (this guide — no agent) | Reference documentation |

**Total: 6 subagents + 9 workflows = 15 entry points**

---

## When to Use What

| I want to... | Use this | Type |
|---|---|---|
| Assess portfolio health (all dimensions) | `ring:portfolio-review` | Workflow (4 agents) |
| Plan resources across projects | `ring:resource-allocation` | Workflow |
| Create an executive report | `ring:executive-summary` | Workflow |
| Manage portfolio-level risks | `ring:risk-management` | Workflow |
| Check one project's health | `ring:project-health-check` | Workflow |
| Map cross-project dependencies | `ring:dependency-analysis` | Workflow |
| Run portfolio retrospective | `ring:pmo-retrospective` | Workflow |
| Do something custom | Direct dispatch (e.g., `ring:portfolio-manager`) | Agent |

---

## PMO vs PM vs Dev — Scope Boundaries

| Team | Focus | Scope |
|------|-------|-------|
| **ring-pmo-team** | Portfolio governance | Multi-project coordination, resources, executive reporting |
| **ring-pm-team** | Single feature planning | PRD, TRD, task breakdown for ONE feature |
| **ring-dev-team** | Implementation | Code, architecture, DevOps, QA |

**Use PMO when:**
- Managing multiple projects simultaneously
- Planning resources across projects
- Reporting to executives on portfolio status
- Assessing portfolio-level risks
- Conducting governance reviews

**Use PM when:**
- Planning a single feature
- Creating PRD/TRD for one initiative
- Breaking down one feature into tasks

**Teams work together:** PMO provides portfolio context → PM plans features → Dev implements code.

---

## Dispatch Syntax

### Single agent

```
Dispatch ring:portfolio-manager to assess our 5 active projects for Q2 capacity
and identify any strategic misalignments.
```

### Multiple agents (parallel — preferred)

```
CORRECT (parallel):
Task #1: ring:portfolio-manager
Task #2: ring:risk-analyst
(Both run simultaneously)

WRONG (sequential = 2x slower):
Task #1: ring:portfolio-manager → wait → Task #2: ring:risk-analyst
```

### With urgency flag

```
Dispatch ring:portfolio-manager
Prompt: "URGENT: Board meeting tomorrow. Assess portfolio status across
all active projects. Focus on blockers and resource conflicts."
```

---

## Output Formats by Agent

| Agent | Returns |
|-------|---------|
| `portfolio-manager` | Strategic findings, capacity summaries, prioritization recommendations |
| `resource-planner` | Capacity matrices, allocation options, conflict resolutions |
| `governance-specialist` | Gate assessment, compliance findings, process gaps |
| `risk-analyst` | RAID aggregation, mitigation strategies, portfolio risk summary |
| `executive-reporter` | Dashboard markdown, status summary, board-ready findings |
| `delivery-reporter` | Git analysis, squad showcases, HTML visual presentations |

---

## Timing Expectations

| Complexity | Time |
|------------|------|
| Simple request | 1–2 minutes |
| Complex analysis (multi-project, deep dive) | 3–5 minutes |
| Multi-agent parallel | Results stagger by complexity |
| With follow-ups | 5–10 minutes per iteration |

---

## Blocker Criteria — STOP and Report

The PMO skill enforces mandatory escalation for decisions that require human authority:

| Decision Type | Examples | Required Action |
|---------------|----------|-----------------|
| Portfolio Prioritization | Which project gets resources first | STOP → Report trade-offs → Wait for executive decision |
| Resource Conflict | Same person needed on multiple projects | STOP → Document conflict → Wait for management decision |
| Strategic Alignment | Project doesn't fit current strategy | STOP → Escalate with analysis → Wait for guidance |
| Budget Reallocation | Moving funds between projects | STOP → Prepare options → Wait for financial approval |
| Project Termination | Recommend stopping a project | STOP → Document rationale → Wait for sponsor decision |

The agent cannot make strategic or resource decisions autonomously. It will always escalate these.

---

## Non-Negotiable Requirements

| Requirement | Rationale |
|-------------|-----------|
| Dispatch to specialist agent | Specialists have PMO frameworks loaded; general agents don't |
| Evidence-based reporting | Opinions are not PMO outputs — data is required |
| Gate compliance | Gates prevent project failures; skipping creates risk |
| Risk documentation | Undocumented risks cannot be managed |
| Stakeholder communication | Silent PMO = failed PMO |

These cannot be overridden by user pressure, urgency, or authority.

---

## Pressure Resistance (Built-In)

The skill includes explicit resistance to process bypass:

| Pressure | Agent Response |
|----------|----------------|
| "Skip the analysis, just give me the answer" | "Executive urgency increases need for accuracy. Expediting but not skipping validation." |
| "Just approve this project" | "Approval without analysis creates downstream problems. Completing analysis now." |
| "Don't include that risk" | "Accurate risk reporting is non-negotiable. Reporting with appropriate context and mitigation." |
| "Resources are fine, trust the leads" | "Trust and verify. Confirming with utilization data." |
| "Skip governance, we're agile" | "Agile requires MORE governance discipline, not less. Applying lightweight gates." |

---

## The ORCHESTRATOR Principle

The core operating principle: dispatch to specialists, don't analyze directly.

### Correct (Orchestrator)
> "I need portfolio status. Let me dispatch `ring:portfolio-manager` to analyze."

### Incorrect (Operator)
> "I'll manually review each project and create a summary myself."

**Why this matters:**
- Specialists have domain-specific frameworks pre-loaded
- Isolated context windows prevent contamination between analyses
- Parallel dispatch is faster than sequential manual work
- Consistent quality regardless of who triggers the analysis

---

## Setup Requirements

### Prerequisites
- VS Code with GitHub Copilot Chat
- Claude Code or equivalent agent runtime supporting subagent dispatch
- The `ring` plugin installed or skill files present in `.github/skills/`

### File Placement

```
.github/
  skills/
    ringusing-pmo-team/
      SKILL.md          ← The full skill definition (dispatch guide)
```

### Integration with Existing Workspace

The PMO skill works alongside existing skills (PM Analysis, EM Analysis, etc.) — it adds portfolio-level oversight, following the same `Outputs/` folder conventions.

---

## Quick Reference Card

**Subagents (6 — dispatch these directly):**

1. `ring:portfolio-manager` — Multi-project coordination, strategic alignment, portfolio health
2. `ring:resource-planner` — Capacity planning, skills matrix, allocation optimization
3. `ring:governance-specialist` — Gate reviews, compliance, process adherence
4. `ring:risk-analyst` — RAID logs, risk aggregation, mitigation planning
5. `ring:executive-reporter` — Portfolio status dashboards, board packages
6. `ring:delivery-reporter` — Git analysis, squad delivery showcases, visual HTML reports

**Supporting Workflows (9 — use as entry points):**

1. `ring:portfolio-planning` — Strategic portfolio planning & capacity assessment
2. `ring:portfolio-review` — Comprehensive portfolio review (dispatches 4 agents in parallel)
3. `ring:resource-allocation` — Resource capacity planning across projects
4. `ring:risk-management` — Portfolio-level risk identification & mitigation
5. `ring:project-health-check` — Individual project health assessment
6. `ring:dependency-analysis` — Cross-project dependency mapping
7. `ring:executive-summary` — Executive dashboards and status summaries
8. `ring:pmo-retrospective` — Portfolio retrospectives & lessons learned
9. `ring:using-pmo-team` — This skill (dispatch guide)

**Total: 6 subagents + 9 supporting workflows = 15 entry points**

| I want to... | Use this | Type |
|---|---|---|
| Assess portfolio health | `ring:portfolio-review` | Workflow (4 agents in parallel) |
| Plan resources | `ring:resource-allocation` | Workflow (resource-planner) |
| Create an executive report | `ring:executive-summary` | Workflow (executive-reporter) |
| Manage risks | `ring:risk-management` | Workflow (risk-analyst) |
| Do something custom | `ring:portfolio-manager` (or other agent) | Direct dispatch |
| Learn how to use the plugin | `ring:using-pmo-team` | This skill |

**Portfolio Review (comprehensive):**

```
Entry Point: ring:portfolio-review
Dispatches: portfolio-manager + resource-planner + risk-analyst + governance-specialist (in parallel)
Output: Comprehensive assessment with all dimensions
```
