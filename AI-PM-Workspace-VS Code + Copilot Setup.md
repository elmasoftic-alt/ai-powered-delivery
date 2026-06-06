# AI-Augmented Product Management Workspace — Onboarding Guide

A reproducible setup for senior product managers who run **discovery, planning, documentation, and stakeholder communication** through GitHub Copilot inside Visual Studio Code.

This guide is intentionally generic. Replace placeholder values (project keys, space names, board IDs, team rosters, sprint labels) with whatever your organization uses. Nothing in here assumes a specific product, market, or domain.

---

## 1. Overview

This workspace is a **knowledge-management and delivery surface**, not a code repository. It treats AI as an embedded teammate that ingests raw inputs, produces structured artefacts, and accumulates institutional memory over time.

The setup is built on three principles:

1. **Inputs are immutable.** Meeting transcripts, source documents, exports, and screenshots live in a read-only inbox. The AI reads them but never edits them.
2. **Outputs are AI-owned.** Analyses, decision records, status updates, planning artefacts, and retrospectives live in an AI-maintained outputs folder governed by a strict naming and indexing convention.
3. **Knowledge compounds.** Persistent strategic context — current quarter, annual goals, glossary, dependency map, planning preferences — lives in a memory layer that prompts and agents load automatically.

A product manager uses this setup, day to day, for:

- Ingesting meeting transcripts and producing PM, Engineering Manager, or executive-lens analyses.
- Extracting decisions, dependencies, and ticket candidates from meetings.
- Running sprint and backlog hygiene scans against the ticketing system.
- Drafting weekly stakeholder status updates, escalation briefs, and steering content.
- Quarterly backlog diagnosis, theme clustering, and next-quarter recommendations.
- Cross-cycle retrospective synthesis (delivery health signals over time).
- Drafting and posting decision records to the documentation site (always behind an explicit confirmation gate).

Every workflow is deterministic, file-based, and reviewable. Nothing is implicit.

---

## 2. Prerequisites and Installation

### 2.1 Required tools and accounts

| Item | Purpose |
|---|---|
| Visual Studio Code (current stable) | Editor and AI surface |
| GitHub account with Copilot + Copilot Chat | Core AI capabilities (chat, prompts, agents, custom instructions) |
| Python 3.10+ with `venv` | Required by the document-conversion skill (handles `.docx`, `.pdf`, `.pptx`, `.xlsx`, `.csv`, `.html`) |
| Node.js LTS with `npx` | Required by Node-based MCP servers (e.g., a whiteboard or design-tool MCP) |
| Account on the team's ticketing system | All ticket queries, status updates, and creation flows route through it |
| Account on the team's documentation/wiki site | Decision records and status updates are published here |
| Git | Versioning the workspace and sharing the configuration with teammates |

### 2.2 Recommended VS Code extensions

Add a `.vscode/extensions.json` to your repository so teammates get the right tooling on first open:

- **GitHub Copilot** and **GitHub Copilot Chat** — the core AI surface.
- **Markdown All in One** (or equivalent) — wiki authoring, table editing, TOC.
- **YAML** — needed for frontmatter on `*.instructions.md`, `*.prompt.md`, and `*.agent.md` files.
- **Mermaid Preview** — for diagram blocks in steering content and architecture decisions.
- A note-taking front-end (optional) — the workspace is compatible with vault-style note tools that read plain Markdown.

### 2.3 One-time setup

```bash
# 1. Clone the workspace
git clone <repo-url> ai-pm-workspace
cd ai-pm-workspace

# 2. Create the Python virtual environment for the document-conversion skill
python3 -m venv .venv
source .venv/bin/activate
pip install python-docx pdfplumber python-pptx openpyxl markdownify beautifulsoup4

# 3. Open in VS Code, sign in to Copilot, install recommended extensions
code .
```

### 2.4 Connect AI to your tools (MCP servers)

The workspace assumes Copilot Chat can reach your **ticketing system** and your **documentation site** through Model Context Protocol (MCP) servers. Two layers are involved:

- **User-level MCP servers** (configured in your VS Code Copilot settings) — typically the ticketing-system MCP and a code-host MCP. These give Copilot the ability to search tickets, read pages, post comments, and create draft pages.
- **Workspace-local MCP servers** (configured in `.vscode/mcp.json`) — used for tools where the integration is repo-scoped or experimental, such as a whiteboard/design MCP. Tokens go in this file; never commit real secrets.

A minimal `.vscode/mcp.json` looks like this:

```json
{
  "servers": {
    "<tool-name>": {
      "command": "npx",
      "args": ["-y", "<mcp-package-name>"],
      "env": {
        "<TOOL_API_TOKEN>": "<paste-your-token-here>"
      }
    }
  }
}
```

Record your organization's identifiers (cloud ID, project key, space key, active board, decision-log root page) in the workspace-wide instructions file (see Section 4) so every prompt and agent picks them up automatically.

### 2.5 Basic Git expectations for product managers

You do not need to write code, but you should be comfortable with:

- Cloning the repository and pulling the latest changes before starting work.
- Creating a branch for non-trivial customizations (new prompts, new instruction files).
- Committing AI-generated outputs alongside meaningful messages and pushing to a shared remote.
- Reviewing diffs in Markdown files — most of your changes will be Markdown.

---

## 3. Extension and Agent Setup

### 3.1 Copilot Chat surfaces in active use

| Surface | Why it matters here |
|---|---|
| `@workspace` chat | Default participant for any question about the repo, its prompts, or its outputs. |
| Slash-invoked prompt files (`/<prompt-name>`) | The primary way you trigger the standardized workflows. |
| `*.agent.md` participants (`@<agent-name>`) | Long-running, multi-step workflows you want to invoke by name and pass an argument to. |
| Built-in subagents (e.g., a read-only "Explore" subagent) | Used internally when a workflow needs to fan out into search/read tasks without polluting the main thread. |
| Ticketing-system MCP tools | Every ticket query, transition, comment, and creation. |
| Documentation-site MCP tools | Page search, fetch, draft, comment, and publish. |
| Code-host MCP tools | Optional — used when product work touches a code repository. |
| Image-viewing tool | Used by the visual-extraction skill for whiteboards, diagrams, and screenshots. |
| Terminal | Used by the document-conversion skill for `.docx`/`.pdf`/`.pptx`/`.xlsx` ingest. |

### 3.2 Custom agents (`.github/agents/`)

VS Code Copilot Chat recognizes `*.agent.md` files with YAML frontmatter (`name`, `description`, `tools`, `argument-hint`) as named chat participants. The workspace ships with one production agent and a small amount of supporting state.

| Agent | Purpose | Invocation |
|---|---|---|
| **Meeting Intelligence** | End-to-end pipeline that ingests a meeting transcript, optionally pulls ticketing context, extracts decisions and dependency signals, appends to the decisions log and dependency map, drafts a decision record on the documentation site, asks for confirmation before posting, and finally offers to generate ticket drafts. | `@<agent-name> [optional transcript filename]`. The agent detects the desired analytical lens (PM / EM / executive) and the ticketing scope (sprint-only / full backlog / transcript-only) from your prompt. |

Supporting state files live next to the agent:

- A **sprint configuration** JSON — single source of truth for the active project key and sprint label.
- A **hygiene history** JSON — overwritten on each ticket-hygiene run so trend labels (new, recurring, fixed) are accurate week over week.

Other multi-step workflows are implemented as **prompt files** (Section 4), which you trigger with `/`. If you want any of them to be invocable as `@name`, convert them to `*.agent.md` and add the appropriate frontmatter.

### 3.3 What "configured" means in practice

The agent and prompt files are how the team standardizes Copilot's behaviour without anyone re-typing long instructions. Once your VS Code is signed in, the rules in `.github/copilot-instructions.md` and the scoped instruction files apply automatically; prompts and agents can be invoked the moment you open the workspace.

---

## 4. Prompt Files and Custom Instructions

The workspace separates rules (what the AI must always do) from procedures (how to perform a specific task) from playbooks (multi-step pipelines). All three are plain Markdown with YAML frontmatter.

### 4.1 Workspace-wide instructions

| File | Scope | What it enforces |
|---|---|---|
| `.github/copilot-instructions.md` | All Copilot interactions in this workspace | The user's role and context, the workspace folder layout, default behaviours (read the wiki schema and outputs index first, ISO dates, append-only logs, naming conventions), skill auto-loading rules, ticketing/documentation safety, the team roster and key initiatives, and the quarterly-planning extension. |
| `.github/instructions/<topic>.instructions.md` | Scoped via `applyTo` glob (e.g., backlog-planning tasks) | Topic-specific rules. The shipped example forces use of the strategic memory and templates, requires synthesis over ticket lists, and prescribes a fixed set of action categories for next-quarter recommendations. |

Together these files act as an organization-level style guide for AI output. Every analysis comes back in the same shape, with the same date format, the same severity legend, and the same safety gates — regardless of which teammate ran it.

### 4.2 Prompt files (`.github/prompts/`)

Prompt files use VS Code's prompt-file frontmatter (`description`, `mode: agent`, optional `tools`). You invoke them with a slash (e.g., `/<prompt-name>`), or open the file and run it from the editor.

| Prompt | Intent | Typical output |
|---|---|---|
| **Backlog scan** | Diagnostic-only scan of the backlog: volumes by status, age distribution, blockers, orphaned items, possible duplicates. Run before deeper strategic analysis when the backlog is noisy. | Inline structured report. |
| **Theme clustering** | Collapses a long ticket list into a small number of strategic themes or initiative candidates with confidence flags and ticket-evidence links. | Inline themed table. |
| **Quarterly backlog strategist** | Strategic synthesis of the backlog against current sprint, current quarter, and annual goals. Returns next-quarter recommendations using a fixed action vocabulary: Commit, Discover first, Merge/reframe, Defer, Close, Escalate. | A structured analysis written to the outputs folder when asked to save. |
| **Next-quarter plan** | Produces a formal next-quarter recommendation set using the supplied template. Reads the strategic memory layer for context. | A plan file matching the template. |
| **Retro intelligence** | Cross-cycle synthesis. Reads every PM/EM analysis listed in the outputs index plus the decisions log; surfaces recurring risks, stale decisions, ownership hotspots, decayed actions, theme drift, and dependency health. | A "delivery health signal" file plus index/log updates. |
| **Weekly delivery chain** | End-to-end weekly pipeline: convert source → run Meeting Intelligence (transcript-only) → run ticket hygiene (sprint-only) → draft a stakeholder status update → write all outputs → request confirmation before publishing to the documentation site. Stop-on-failure with a strict pass-through contract between steps. | A status update plus updates to decisions log, dependency map, hygiene history, index, and operations log. |
| **Ticket hygiene** (lives in `.github/agents/`) | Standalone backlog or sprint hygiene scanner with four flags (empty description, stuck-in-analysis beyond a threshold, possible duplicates, orphaned), priority scoring (P1/P2/P3), and week-over-week trend memory. | Inline report; updates the hygiene-history JSON. |

### 4.3 Why prompts and instructions matter for a PM team

Prompts encode **how** to do recurring work. Instructions encode **what must always be true** about the output. Together they:

- Eliminate variance between teammates — two PMs running the same prompt get the same shape of output.
- Make AI output reviewable — readers learn the format once and can scan it quickly forever after.
- Move team-specific knowledge (action vocabulary, severity scale, naming convention, team roster) into version-controlled files instead of living in someone's head.

---

## 5. Skills, Tools, and Integrations

Skills are deterministic, step-by-step procedures the AI loads on demand. Each skill is a single `SKILL.md` file. They are stored in `.github/skills/<Skill Name>/SKILL.md` and are auto-loaded when the workspace-level instructions detect the matching trigger (file type, mode keyword, lens, etc.).

### 5.1 Analytical lens skills

| Skill | Lens | Auto-load trigger |
|---|---|---|
| **PM Analysis** | Senior product manager — explicit and implicit decisions, ownership gaps, risks with severity, structured action items, cross-team dependencies, strategic implications. Default lens when nothing is specified. | Meeting analyses without an explicit lens; PM mode in the Meeting Intelligence agent. |
| **EM Analysis** | Engineering manager — architectural implications, operational risk, ownership gaps, delivery realism, tooling concerns. | EM mode in the Meeting Intelligence agent. |
| **Executive (C-Level) Analysis** | Executive lens — strategic decisions, board-level risk, trade-offs, governance gaps, a one-paragraph executive brief. | Executive mode. |

Each lens skill produces sections in a consistent order so analyses can be diffed and compared across topics and weeks.

### 5.2 Input-handling skills

| Skill | Purpose | Trigger |
|---|---|---|
| **Format converter** | Converts `.docx`, `.pdf`, `.pptx`, `.xlsx`, `.csv`, `.html`, `.txt` to clean Markdown via the local Python environment. Preserves headings, lists, and table structure; writes a metadata header noting the original source. | Any non-Markdown file dropped into the inputs folder, or any "convert / extract / import" request. |
| **Visual extractor** | Structured extraction from images (whiteboards, architecture diagrams, board screenshots, roadmaps, slides, charts, tables). Confidence-rated output and optional Mermaid recreation of diagrams. | Any `.png` / `.jpg` / `.jpeg` / `.gif` / `.webp` source. |

### 5.3 Output skills

| Skill | Purpose | Trigger |
|---|---|---|
| **Stakeholder communications** | Three modes: a weekly RAG-based status update, a single-ask escalation brief, and steering-deck content (Markdown plus Mermaid-based timeline visualization). Each mode lists explicit quality gates and source-traceability rules. | Status / escalation / steering requests. |

### 5.4 Optional / reference skills

The workspace can also contain reference skills that document **how** to use external multi-agent plugins (for example, a portfolio-management plugin with its own subagents and workflows). These are documentation-only unless the underlying plugin is also installed; treat them as a forward-compatible dispatch reference.

### 5.5 Knowledge sources the AI is expected to use

Beyond skills, the AI relies on a small but high-leverage set of knowledge files that prompts reference by relative path:

- A **wiki schema** file at the repo root that defines page types, naming conventions, and the ingest/query/lint workflows.
- An **outputs index** that catalogs every AI-generated page so prior work is reused, not duplicated.
- An append-only **operations log** that records every ingest, query, and lint pass.
- An append-only **decisions log** that registers every decision the AI has extracted.
- A **memory** folder with persistent strategic context: current quarter, annual goals, planning preferences, glossary, dependency map, partner context.
- A **templates** folder with skeletons for next-quarter plans, retrospective syntheses, PRDs, decision records, and any other recurring artefact your team uses.

### 5.6 How to point Copilot at this knowledge

In any chat or prompt, you can pin context explicitly:

- `#file:<path>` to attach a specific file (memory, template, or analysis).
- `#folder:<path>` to attach a folder.
- Or just tell the AI: "Use the next-quarter-plan template and the current-quarter memory." The workspace instructions will resolve the paths.

---

## 6. Workspace Documentation and Product Wiki

### 6.1 Layered structure

```
README.md                      ← Project overview (optional)
WIKI.md                        ← Wiki schema: page types, naming, ingest/query/lint workflows
.github/
  copilot-instructions.md      ← Repository-wide Copilot rules
  instructions/                ← Scoped *.instructions.md files
  prompts/                     ← VS Code prompt files (slash-invocable)
  agents/                      ← Copilot Chat agents and their state
  skills/                      ← SKILL.md procedures
<Workstream>/
  Inputs/                      ← Read-only sources (transcripts, exports, screenshots)
  Outputs/                     ← AI-owned wiki
    index.md                   ← Catalog of every AI-generated page
    log.md                     ← Append-only operations log
    decisions-log.md           ← Append-only decisions registry
    *-PM-Analysis-YYYY-MM-DD.md
    *-EM-Analysis-YYYY-MM-DD.md
    *-Meeting-Summary-YYYY-MM-DD.md
    *-JIRA-Tickets-YYYY-MM-DD.md
    Status-Update-YYYY-MM-DD.md
    Escalation-*-YYYY-MM-DD.md
    Steering-Deck-*-YYYY-MM-DD.md
    Delivery-Health-Signal-*.md
  AI/
    Memory/                    ← Persistent strategic context
      current-quarter.md
      annual-goals.md
      planning-preferences.md
      domain-glossary.md
      dependency-map.md
      partner-context.md
    Templates/                 ← next-quarter-plan-template.md, retrospective-template.md, etc.
.vscode/
  mcp.json                     ← Workspace-local MCP servers
  extensions.json              ← Recommended extensions for teammates (recommended addition)
```

### 6.2 How docs are used as Copilot context

- **Wiki schema**: loaded explicitly at the start of any ingest, query, or lint operation.
- **Outputs index**: read before writing any new page so prior analyses can be cross-referenced and superseded rather than duplicated.
- **Memory files**: referenced by name in instruction files and in every relevant prompt — they carry the strategic context that turns generic AI output into team-specific output.
- **Templates folder**: drives the structure of generated planning and retrospective artefacts.

### 6.3 Recurring documented workflows

- **Source → analysis → decision record → published page**: handled by the Meeting Intelligence agent and an optional ticket-generation step.
- **Weekly delivery roll-up**: handled by the weekly-delivery-chain prompt — one input transcript yields decisions, dependency updates, sprint-hygiene flags, and a stakeholder status update in a single run.
- **Quarterly planning**: backlog scan → theme clustering → quarterly backlog strategist → next-quarter plan.
- **Retrospective**: retro-intelligence prompt run after enough analyses have accumulated.
- **Visual or document ingest**: format-converter or visual-extractor first, then any analytical lens skill.

---

## 7. Standard AI Workflows for Product Managers

For each workflow: the starting prompt, the agents and skills involved, and how to validate the output.

### 7.1 Discovery and research synthesis

- **Start with**: drop research notes, transcripts, or survey exports into the inputs folder. Then ask: *"Convert and analyse with the PM lens. Surface themes, jobs-to-be-done, and contradictions."*
- **Agents / skills**: the format converter (or visual extractor for whiteboards) → the PM Analysis lens.
- **Validate**: every theme should cite at least one source quote or section reference; flag low-confidence inferences explicitly; reconcile contradictions before declaring a finding.

### 7.2 Problem and opportunity framing

- **Start with**: *"Using #file:current-quarter.md and the latest research analysis, structure a problem statement, opportunity assessment, and impact-vs-effort table for <topic>."*
- **Agents / skills**: PM Analysis lens; optionally the theme-clustering prompt if the input is broad.
- **Validate**: ensure the opportunity statement names the user, the unmet need, the desired outcome, and the success metric; impact-vs-effort scoring should state assumptions, not just numbers.

### 7.3 PRD and specification drafting

- **Start with**: *"Use the PRD template in the templates folder as a base. Draft a v0 PRD from the attached opportunity assessment and the latest discovery findings."*
- **Agents / skills**: PRD template + PM Analysis lens; optionally the EM Analysis lens for a parallel engineering view on feasibility and architecture.
- **Validate**: cross-check assumptions, success metrics, and out-of-scope items against the source documents; iterate by asking targeted questions rather than re-generating the whole document.

### 7.4 Stakeholder communication

- **Start with**: invoke the stakeholder-communications skill with a clear mode keyword — `status-update`, `escalation-brief`, or `steering-deck` — plus the audience and any specific blockers to surface.
- **Agents / skills**: the stakeholder-comms skill, which pulls the current-quarter memory and the most recent analyses or delivery-health signal.
- **Validate**: walk the quality gates listed at the bottom of the skill — no invented facts, an owner per blocker, RAG status justified, the outputs index and operations log updated, the publish step gated by an explicit confirmation.

### 7.5 Meeting intelligence and decision capture

- **Start with**: `@<meeting-intelligence-agent> [optional filename]`. State the lens (PM / EM / executive) and the scope (`sprint only`, `transcript only`, or default full backlog).
- **Agents / skills**: the Meeting Intelligence agent → the chosen analytical lens skill → optional format conversion.
- **Validate**: confirm new rows in the outputs index and operations log; review the decisions-log append; review the documentation-site draft before confirming publish; review any ticket drafts before they are created.

### 7.6 Roadmap and quarterly planning

- **Start with**: `/backlog-scan` for diagnosis → `/theme-clustering` → `/quarterly-backlog-strategist` → `/next-quarter-plan`.
- **Agents / skills**: all four prompts; the strategist reads the memory layer and the next-quarter-plan template.
- **Validate**: every recommendation must use the fixed action vocabulary (Commit / Discover first / Merge–reframe / Defer / Close / Escalate); every "Commit" item must be checked against the dependency map for unresolved blockers; provisional recommendations must be flagged as such when strategic context is incomplete.

### 7.7 Cross-cycle retrospective

- **Start with**: `/retro-intelligence`.
- **Agents / skills**: reads the current-quarter memory, the decisions log, and every PM/EM analysis listed in the outputs index. Writes a single delivery-health-signal file.
- **Validate**: every recurring-risk claim should cite at least two source files; stale decisions should include an age in days; ownership hotspots should be cross-referenced with ticketing data on request.

### 7.8 Generic review pattern (applies to all workflows)

- Compare every claim back to a source file or ticket.
- Sanity-check commercial, legal, or regulatory implications manually.
- Confirm that the output respects the team's tone and severity conventions.
- Update the outputs index and append to the operations log before considering the task done.

---

## 8. Conventions and Best Practices

### 8.1 How to structure prompts

Use the **Context → Goal → Constraints → Output format** pattern:

1. **Context**: which files, memory, or analyses the AI should treat as ground truth (`#file:` references work well).
2. **Goal**: what you want produced, expressed as an outcome ("a v0 PRD", "a board-ready summary", "a next-quarter recommendation set").
3. **Constraints**: lens, audience, length cap, severity scale, things to avoid, sources of truth.
4. **Output format**: which template, which sections, which tables, where to save it, whether to update the index and log.

### 8.2 Writing style

- Direct, structured, senior-PM tone. Tables over prose for decisions, risks, and action items.
- ISO dates everywhere (`YYYY-MM-DD`).
- Severity legend used consistently (e.g., 🔴 high, 🟠 medium, 🟡 low, 🟢 healthy).
- No marketing language, no padding, no generic commentary.
- Internal communication is honest and diagnostic; external communication tightens the same content into outcomes and asks.

### 8.3 Decision and audit trail

- Append every decision to the decisions log — never overwrite.
- Append every operation (ingest, query, lint, dependency update, publish gate) to the operations log.
- Update the outputs index immediately after creating any new page so future runs find it.
- Add cross-references between related pages so a reader following one trail finds the others.

### 8.4 Safety rules (non-negotiable)

- Never modify files in the inputs folder.
- Never invent identifiers (ticket keys, page IDs) — always fetch them through the appropriate MCP tool.
- Never create, edit, or transition tickets, and never publish a documentation page, without an explicit "yes" from the user.
- Always write the artefact locally first; publishing is a separate, gated step.
- Treat tokens in `.vscode/mcp.json` as local-only secrets — never commit real values.

### 8.5 What product managers should always check manually

- **Numerical claims** — KPIs, percentages, counts. The AI is good at structure; you remain accountable for accuracy.
- **Commercial assumptions** — pricing, contractual implications, partner exposure.
- **Legal, regulatory, and compliance language** — these never get auto-published.
- **Owner attribution** — never let the AI assign work to a named person without confirmation.
- **Strategic framing** — the AI synthesizes; you decide what gets escalated.

### 8.6 How to avoid over-relying on AI

- Use AI to accelerate **the first 80%** of analysis, drafting, and structuring. Reserve the final pass for human judgment.
- Resist the temptation to publish AI output unmodified. The confirmation gate exists for a reason.
- When the AI asks for clarifying input, give it real input. Vague prompts produce confident but useless output.
- Periodically run the lint workflow on the wiki to catch contradictions and stale claims that the AI itself has accumulated.

---

## 9. How to Customize This Setup

The workspace is designed to be forked and adapted. Keep customizations consistent with the existing conventions so a community of product managers can share a common AI vocabulary across organizations.

### 9.1 Add or modify an instruction file

1. Create `.github/instructions/<topic>.instructions.md` with YAML frontmatter — `applyTo: "<glob>"` and a `description`.
2. Keep it rule-oriented: what the AI must do or refuse to do for that topic. Defer narrative procedures to skill files.
3. Update the workspace-wide `copilot-instructions.md` only if the rule is genuinely workspace-wide.

### 9.2 Add a new prompt

1. Add `.github/prompts/<name>.prompt.md` with frontmatter (`description`, `mode: agent`, optional `tools`).
2. Reference memory and templates by relative path so the prompt works regardless of the user's current folder.
3. Specify the output filename pattern explicitly, and include the index/log update step in the prompt's procedure.

### 9.3 Add a new agent

1. Add `.github/agents/<name>.agent.md` with frontmatter (`name`, `description`, `tools`, `argument-hint`).
2. Number the workflow steps (Step 0 → Step N), state the constraints, and include an explicit confirmation gate before any state-changing action.
3. Co-locate any state files the agent maintains (history JSONs, configuration JSONs) in the same folder.

### 9.4 Add a new skill

1. Create `.github/skills/<Skill Name>/SKILL.md`.
2. Make the procedure deterministic — numbered steps, code or queries where relevant, explicit output paths, and a "what this skill does NOT do" section.
3. Add an auto-load trigger in `.github/copilot-instructions.md` so the AI knows when to apply it.

### 9.5 Extend the persistent knowledge base

- Add or update files under `<Workstream>/AI/Memory/`. Common ones: current-quarter, annual goals, glossary, dependency map, partner context, planning preferences.
- Reference any new memory file explicitly from the prompts or instructions that should consume it. Files that aren't referenced won't be loaded automatically.

### 9.6 Add an MCP integration

- Edit `.vscode/mcp.json`. Use `npx` for Node-based MCP servers and Python entry points for Python servers.
- Document the integration in a new instruction file or in the workspace-wide instructions so other prompts and agents can use it.
- Never commit real tokens. Provide a `.env.example` or a placeholder pattern (`<paste-your-token-here>`).

### 9.7 Adapt the setup to a different product area or market

- Fork the memory layer (current-quarter, annual goals, glossary, dependency map) per product area; keep prompts and skills shared.
- If two product areas share a workspace, give each its own outputs and inputs folder with its own index and operations log; keep `.github/` shared.
- Keep the action vocabulary, severity scale, naming convention, and confirmation-gate behaviour identical across product areas — these are the conventions that make AI output portable across teams.

### 9.8 Recommended additions when standing up a new copy

- Add a `.vscode/extensions.json` so teammates install the same set of extensions on first open.
- Add a `README.md` at the repo root that points new teammates to this onboarding guide.
- Add a `CONTRIBUTING.md` describing how to propose new prompts, skills, or instruction files (PR review process, naming, frontmatter conventions).
- Decide and document a single audit/log retention policy — these files grow indefinitely if left untouched.

---

## Appendix: Quick reference

### File naming

`[Topic-in-kebab-case]-[PageType]-[YYYY-MM-DD].md`

Examples: `<topic>-PM-Analysis-YYYY-MM-DD.md`, `Status-Update-YYYY-MM-DD.md`, `Delivery-Health-Signal-<cycle>-YYYY-MM-DD.md`.

### Action vocabulary for backlog and quarterly planning

**Commit**, **Discover first**, **Merge / reframe**, **Defer**, **Close**, **Escalate**.

### Severity legend

🔴 High · 🟠 Medium · 🟡 Low · 🟢 Healthy.

### Confirmation gate

Any state-changing action — creating, editing, transitioning a ticket, or publishing a documentation page — must be preceded by an explicit "yes" from the user. The AI writes a draft locally first, surfaces it for review, then publishes only on confirmation.
