# AI-Powered Delivery Management — Setup Guide

> A step-by-step guide to recreating this workspace from scratch. No AI experience required.

---

## Table of Contents

1. What You're Building
2. Prerequisites
3. Install VS Code and GitHub Copilot
4. Connect Atlassian (Jira & Confluence) via MCP
5. Set Up Your Workspace Folder Structure
6. Write Copilot Instructions (Your AI's Rulebook)
7. Create Agent 1 — Ticket Hygiene
8. Create Agent 2 — Meeting Intelligence
9. Create Agent 3 — LLM Wiki
10. Create Skills (Analytical Lenses)
11. Run Your First Agent
12. Tips and Troubleshooting

---

## 1. What You're Building

By the end of this guide you'll have an AI-powered workspace that can:

* **Scan your Jira backlog** for ticket quality issues and produce a hygiene report (Ticket Hygiene Agent)
* **Process meeting transcripts**, extract decisions, post decision records to Confluence, and optionally create Jira tickets (Meeting Intelligence Agent)
* **Build a knowledge wiki** from raw documents — strategy docs, roadmaps, meeting notes — with interlinked pages, a search index, and a changelog (LLM Wiki Agent)

All of this runs inside VS Code using GitHub Copilot's chat and agent features. No coding required.

---

## 2. Prerequisites

| What | Why |
| --- | --- |
| A Mac, Windows, or Linux computer | VS Code runs on all three |
| An internet connection | GitHub Copilot is cloud-based |
| A GitHub account | Copilot requires one — you need a GitHub account that belongs to the hm-group organization |
| GitHub Copilot access | Access is managed centrally — see Step 2 below for how to request it |
| An Atlassian Cloud account (Jira + Confluence) | So the AI can read/write tickets and pages |

---

## 3. Install VS Code and GitHub Copilot

### Step 1 — Install VS Code

1. Go to [https://code.visualstudio.com](https://code.visualstudio.com/)
2. Download the installer for your operating system
3. Run the installer and follow the prompts
4. Open VS Code once it's installed

### Step 2 — Request GitHub Copilot access

You need to request access through your client’s organization.

### Step 3 — Open Copilot Chat

1. Press Cmd + Shift + I (Mac) or Ctrl + Shift + I (Windows/Linux) to open the Copilot Chat panel
2. You should see a chat window where you can type messages to the AI
3. Try typing "Hello" to verify it works

> **What is Copilot Chat?** It's a chat window inside VS Code where you talk to an AI assistant. You type questions or instructions in plain English, and the AI responds. It can also read and edit files in your workspace.

---

## 4. Connect Atlassian (Jira & Confluence) via MCP

**MCP** stands for **Model Context Protocol**. It's a way to give the AI access to external tools — in this case, your Jira and Confluence instance. Think of it as plugging Jira into the AI's brain.

### Step 1 — Install the Atlassian MCP extension

1. Click the **Extensions** icon in the left sidebar (the four-squares icon — same place you installed Copilot)
2. Search for **"Atlassian MCP"**
3. Click **Install**
4. Once installed, VS Code will prompt you to authenticate with Atlassian
5. Click the prompt — you'll be redirected to Atlassian's login page in your browser
6. Sign in with your Atlassian account and grant access
7. Once authenticated, the MCP server status should show as **connected** in VS Code

### Step 2 — Verify the connection

1. Open Copilot Chat
2. Type: **"Search for Jira tickets in project [PROJECT KEY]"**
3. If it returns tickets, you're connected!

> **What just happened?** You gave the AI permission to talk to your Jira and Confluence instance. Now when you ask it about tickets or pages, it can actually go look them up in real time.

---

## 5. Set Up Your Workspace Folder Structure

A "workspace" is just a folder on your computer that VS Code opens. All your agents, configs, and documents live inside it.

### Step 1 — Create the folder structure

Create a folder on your computer (e.g., Workspace) with this structure inside it:

```
Workspace/
├── .github/
│   ├── copilot-instructions.md      ← Rules for the AI
│   ├── agents/                       ← Your custom AI agents
│   │   ├── ticket-hygiene.prompt.md  ← Ticket Hygiene Agent
│   │   ├── meeting-intelligence.agent.md  ← Meeting Intelligence Agent
│   │   ├── llm-wiki.agent.md        ← LLM Wiki Agent
│   │   └── hygiene-history.json     ← Auto-maintained by Ticket Hygiene Agent
│   └── skills/                       ← Analytical lenses for Meeting Intelligence
│       ├── PM Analysis/
│       │   └── SKILL.md
│       ├── EM Analysis/
│       │   └── SKILL.md
│       └── C-Level Analysis/
│           └── SKILL.md
├── Transcripts/
│   ├── Input/                        ← Drop meeting transcripts here
│   └── Output/                       ← Agent writes decision logs here
│       └── decisions-log.md
└── llm-wiki/
    ├── SCHEMA.md                     ← Wiki conventions
    ├── raw/                          ← Drop source documents here
    └── wiki/                         ← AI-generated wiki pages
        ├── index.md
        ├── log.md
        ├── sources/
        ├── entities/
        ├── concepts/
        ├── comparisons/
        └── analyses/
```

### Step 2 — Open it in VS Code

1. In VS Code, go to **File → Open Folder**
2. Select your Workspace folder
3. You should see the folder tree in the left sidebar

> **Why .github?** VS Code and GitHub Copilot look for a .github folder in your workspace root for custom instructions and agent definitions. The dot (.) at the start makes it a hidden folder — that's normal.

---

## 6. Write Copilot Instructions (Your AI's Rulebook)

The file .github/copilot-instructions.md is a set of permanent rules that the AI follows in every conversation. Think of it as a job description for the AI.

### Create the file

Create .github/copilot-instructions.md and paste the following (customize the values in brackets):

```
# [Your Team Name] Workspace

This workspace is used for managing Jira boards, Confluence pages, and tracking
work for the **[Your Project Name]** team via Atlassian MCP tools.

## Atlassian Context

- **Cloud ID**: [your-cloud-id]
- **Site**: [yoursite].atlassian.net
- **Primary Jira project**: [PROJECT_KEY]
- **Confluence space**: https://[yoursite].atlassian.net/wiki/spaces/[SPACE_KEY]
- **Active board**: https://[yoursite].atlassian.net/jira/software/c/projects/[PROJECT_KEY]/boards/[BOARD_ID]

Always use the cloud ID above when calling Atlassian MCP tools.

## Jira Conventions

- When searching tickets, **exclude Done/Closed** by default. Use statusCategory != Done in JQL.
- When the user asks about "stale" tickets, use updated <= -7d.
- Sort stale ticket results by updated ASC (oldest first).
- Present Jira results as **tables** with columns: Key (linked), Summary, Status, Priority.

## Formatting

- Always link Jira ticket keys to their web URL.
- Keep summaries concise — use tables over prose for ticket lists.

## Safety

- Always ask for confirmation before any state-changing Atlassian action
  (creating, editing, commenting, transitioning).
- Do not guess. If the source is unclear, say so.
```

### How to find your Cloud ID

1. Open Copilot Chat
2. Type: **"Use the Atlassian MCP tool getAccessibleAtlassianResources to list my sites"**
3. The response will include your Cloud ID — copy it into the file above
4. You only need to do this once

> **Why does this matter?** Without these instructions, the AI would have to figure out your Jira project, formatting preferences, and safety rules every time. This file makes it consistent.

---

## 7. Create Agent 1 — Ticket Hygiene

An **agent** is a reusable AI task with a specific job. Instead of typing long instructions every time, you invoke an agent and it follows its own playbook.

### Step 1 — Create the agent file

Create .github/agents/ticket-hygiene.prompt.md and paste the following:

```
---
mode: agent
description: Scans the Jira backlog for ticket quality issues before grooming sessions
---

You are a Ticket Hygiene Agent for a Delivery Manager.

Your job is to scan the Jira backlog, flag problem tickets, score them by
priority, compare against last week, and produce a report the DM can act
on immediately.

## Setup

At the start of every run:
- Read sprint-config.json from the agents folder to get the projectKey
  and activeSprintLabel
- Read hygiene-history.json from the agents folder for last week's
  flagged tickets
- Do not ask the user for the project key or sprint label

## What to scan

The user will specify one of two modes:

**Sprint mode** — triggered by "sprint only" in the prompt
Scan only tickets that have the activeSprintLabel applied.

**Full backlog mode** — triggered by "full backlog" in the prompt
Scan all open tickets in the project regardless of label or sprint.

If the user does not specify a mode, ask them before proceeding.

## Flags to apply

**Flag 1 — Empty description**
The description field is blank, empty, or contains only whitespace.

**Flag 2 — Stuck in analysis**
Status = "In Analysis" AND the last status change or comment was > 7 days ago.

**Flag 3 — Possible duplicate**
Another open ticket has a very similar summary (same key words, same feature area).
Surface both ticket keys.

**Flag 4 — Orphaned ticket**
No Epic Link / parent Epic AND not in the active sprint.

## Priority scoring

P1 — Fix before grooming: ticket is in the active sprint AND flagged,
OR ticket is blocking another ticket.

P2 — Fix soon: ticket is linked to an epic in the current PI,
OR ticket has been flagged > 14 days (check hygiene-history.json).

P3 — Low urgency: everything else.

Sort by priority within each flag section (P1 first).

## Trend memory

Read hygiene-history.json at the start of every run.
Compare against the previous run to label each ticket:
- 🆕 NEW — not flagged last week
- 🔁 AGAIN — flagged last week and still flagged
- ✅ FIXED — flagged last week but no longer flagged

At the end of the run, overwrite hygiene-history.json with:
{
  "lastRun": "[today's date]",
  "flaggedTickets": ["TICKET-123", "TICKET-456"]
}

## Rules
- Only report. Do not create, edit, or transition any Jira tickets.
- Always update hygiene-history.json at the end of every run.
```

### Step 2 — Create the history file

Create .github/agents/hygiene-history.json:

```
{
  "lastRun": null,
  "flaggedTickets": []
}
```

This file starts empty. The agent will populate it after its first run.

---

## 8. Create Agent 2 — Meeting Intelligence

This agent processes meeting transcripts, extracts decisions, posts to Confluence, and can generate Jira tickets.

### Create the agent file

Create .github/agents/meeting-intelligence.agent.md and paste:

```
---
mode: agent
description: Processes meeting transcripts, extracts decisions, creates
  decision records, posts to Confluence, and optionally generates Jira tickets.
tools: [read, edit, execute, search, atlassian/*]
argument-hint: "Optionally specify a skill mode: PM, EM, or C-Level.
  Leave blank to default to PM mode."
---

You are the Meeting Intelligence Agent. Your job is to process meeting
transcripts alongside active Jira sprint data, extract all decisions made,
and produce structured decision records filed to Confluence.

## Setup

At the start of every run:
- Read sprint-config.json from the agents folder
- Determine the skill mode from the user's prompt (PM, EM, or C-Level)
- If no mode is specified, default to PM mode
- Load the corresponding skill file:
  - PM mode → skills/PM Analysis/SKILL.md
  - EM mode → skills/EM Analysis/SKILL.md
  - C-Level mode → skills/C-Level Analysis/SKILL.md

## Workspace Paths

- Transcripts inbox: Transcripts/Input/
- Decisions output log: Transcripts/Output/decisions-log.md
- Confluence space: [YOUR_SPACE_KEY]
- Confluence parent page: [link to your Decision Tracker page]

## Step-by-Step Workflow

### Step 1 — Detect transcript
Check Transcripts/Input/ for .txt, .md, or .docx files.
Process the most recently modified one.

### Step 2 — Pull Jira context
Query open tickets from the project for supplementary context.
Supports "sprint only", "full backlog", or "transcript only" modes.

### Step 3 — Analyse transcript
Apply the loaded skill file as the analytical lens.

### Step 4 — Append to output log
Write each decision to Transcripts/Output/decisions-log.md.

### Step 5 — Draft Confluence page
Compose a structured decision record page.

### Step 6 — Post to Confluence
Create a child page under your Decision Tracker page.

### Step 7 — Request review
Add a footer comment asking participants to review.

## Rules
- Do NOT invent decisions — only extract what is explicitly stated
- Do NOT modify files in Transcripts/Input/
- Do NOT post to Confluence until decisions are in the Output log
- Always append to the decisions log, never overwrite
```

---

## 9. Create Agent 3 — LLM Wiki

This agent builds a persistent, interlinked knowledge base from raw documents you drop into a folder. Instead of writing all the files manually, you'll ask Copilot to set it up for you based on a well-known pattern by Andrej Karpathy.

### Step 1 — Prompt Copilot to create it

1. Open Copilot Chat (Cmd + Shift + I)
2. Paste the following message:

> "Read this gist and use it to create an LLM Wiki agent for my workspace: <https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f>  — Create the agent file at .github/agents/llm-wiki.agent.md, the schema at llm-wiki/SCHEMA.md, and the starter wiki files under llm-wiki/wiki/ (index.md, log.md, and empty folders for sources, entities, concepts, comparisons, and analyses). Adapt it for use with GitHub Copilot in VS Code."

3. Copilot will read the gist and generate all the files for you
4. Review the files it creates — you can ask it to adjust anything

> **What's happening here?** The Karpathy LLM Wiki gist (

<https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)>
>  describes a pattern where an LLM incrementally builds and maintains a wiki from raw source documents. Instead of re-discovering knowledge from scratch on every question (like RAG), the AI reads your sources, extracts key information, and integrates it into a persistent set of interlinked markdown pages. The wiki keeps getting richer with every source you add.

### Step 2 — Verify the result

After Copilot finishes, you should have:

```
llm-wiki/
├── SCHEMA.md          ← Wiki conventions (created by Copilot)
├── raw/               ← Drop source documents here
└── wiki/
    ├── index.md       ← Content catalog
    ├── log.md         ← Append-only operation log
    ├── sources/       ← One summary page per ingested source
    ├── entities/      ← People, orgs, products, systems
    ├── concepts/      ← Ideas, themes, frameworks
    ├── comparisons/   ← Side-by-side analyses
    └── analyses/      ← Filed query results worth keeping
```

And an agent file at .github/agents/llm-wiki.agent.md.

### Step 3 — Customize for your domain

Open llm-wiki/SCHEMA.md and update the **Domain** section to match your team's context. For example, if you're a delivery team, you might define entity types like People, Teams, Features, Services, and concept types like Decisions, Risks, Metrics, Strategies. The agent will use this to decide how to categorize information from your sources.

---

## 10. Create Skills (Analytical Lenses)

Skills are instruction files that change *how* the Meeting Intelligence Agent analyses a transcript. You've built three:

### PM Analysis

Create .github/skills/PM Analysis/SKILL.md:

```
PM Analysis

You are a senior Product Manager and software engineering strategist.
Analyze the meeting transcript and extract structured, decision-oriented insights.

Structure your output as:
1. Executive Summary (5–8 bullet points max)
2. Explicit Decisions (agreed / deferred / ambiguous)
3. Implicit Signals / Subtext (ownership confusion, hidden risks, political positioning)
4. Risks Identified (description, severity, impact area, mitigation)
5. Action Items (table: Action | Owner | Priority | Deadline | Risk if not done)
6. Decisions Required (not taken yet)
7. Cross-Team Dependencies
8. PM-Level Strategic Implications
9. Recommended Next Steps (top 5 only)
```

### EM Analysis

Create .github/skills/EM Analysis/SKILL.md:

```
EM Analysis

You are a senior Engineering Manager with strong architectural and delivery
experience. Focus on: technical feasibility, architecture implications,
operational risk, ownership clarity, resource impact, delivery realism.

Structure your output as:
1. What Was Actually Decided (Engineering View)
2. Architecture & Design Implications
3. Operational Risk Signals (stability, MTTR, performance, security)
4. Ownership & Responsibility Gaps
5. Delivery & Capacity Reality Check
6. Tooling & Automation Implications
7. What Needs Immediate Engineering Attention (top 5)
8. What Should Be Escalated vs Monitored
```

### C-Level Analysis

Create .github/skills/C-Level Analysis/SKILL.md:

```
C-Level Analysis

You are a senior strategy advisor analyzing a leadership meeting transcript
for a C-level audience. Extract decision-grade insights, not operational
summaries.

Structure your output as:
1. Executive Snapshot (max 8 bullet points)
2. Strategic Decisions Made (explicit & implicit)
3. Business Risks Identified
4. Trade-offs Being Made
5. Investment & Resource Implications
6. Governance & Accountability Signals
7. What Requires Executive Intervention
8. What Should Be Monitored (Board Radar)
9. One-Paragraph Board Brief
```

> **How skills work**: The Meeting Intelligence Agent reads whichever skill file matches the mode you choose. Say "analyze this transcript in EM mode" and it switches to the engineering lens.

---

## 11. Run Your First Agent

### Running the Ticket Hygiene Agent

1. Open Copilot Chat in VS Code (Cmd + Shift + I)
2. Type: **@ticket-hygiene full backlog**
3. The agent will:

   * Read your sprint config
   * Query Jira for all open tickets
   * Flag issues (empty descriptions, stuck tickets, orphans, possible duplicates)
   * Score them by priority
   * Compare against last week's run
   * Output a formatted report
   * Save the flagged tickets to hygiene-history.json for next time

### Running the Meeting Intelligence Agent

1. Save a meeting transcript (.txt, .md, or .docx) into Transcripts/Input/
2. Open Copilot Chat
3. Type: **@meeting-intelligence** (defaults to PM mode)

   * Or: **@meeting-intelligence EM mode** for the engineering lens
   * Or: **@meeting-intelligence C-Level mode** for the executive lens
4. The agent will analyze the transcript, extract decisions, write them to the log, and post to Confluence

### Running the LLM Wiki Agent

1. Save a document (strategy doc, meeting notes, etc.) into llm-wiki/raw/
2. Open Copilot Chat
3. Type: **@llm-wiki ingest [filename]**
4. The agent will read the document, discuss key takeaways, and create/update wiki pages
5. To ask a question: **@llm-wiki What is our North Star Metric?**
6. To check wiki health: **@llm-wiki lint**

---

## 12. Tips and Troubleshooting

### General tips

* **Keep your**[**copilot-instructions.md**](http://copilot-instructions.md/)**up to date** — if you change your Jira project or Confluence space, update it here
* **You can talk to agents conversationally** — if the output isn't right, just tell it what to fix in the same chat thread
* **Agents can't see each other** — each agent runs independently, they don't share context between runs

### Common issues

| Problem | Solution |
| --- | --- |
| "MCP server not connected" | Open Command Palette → "MCP: List Servers" → check if Atlassian shows as connected. Re-authenticate if needed. |
| Agent not found when using @agent-name | Make sure the file is inside .github/agents/ and has the correct frontmatter (mode: agent or appropriate metadata) at the top |
| Copilot doesn't follow the instructions | Make sure .github/copilot-instructions.md is in the root of the folder you opened in VS Code |
| Jira queries return nothing | Check that the project key in your [copilot-instructions.md](http://copilot-instructions.md/) matches your actual Jira project key |
| Wiki agent modifies raw files | This shouldn't happen — the rules say not to. If it does, tell it to stop and undo the change |
| Transcript not detected | Make sure the file is in Transcripts/Input/ and is .txt, .md, or .docx |

### Understanding the key concepts

| Term | What it means |
| --- | --- |
| **VS Code** | A free text editor by Microsoft. Think of it as a smart notepad that can run AI tools. |
| **GitHub Copilot** | An AI assistant that lives inside VS Code. You chat with it and it can read/write files and talk to external services. |
| **MCP (Model Context Protocol)** | A standard that lets AI assistants connect to external tools like Jira and Confluence. Like a USB cable between the AI and your work tools. |
| **Agent** | A reusable AI task with a specific job description. You write the job description once in a file, and invoke it by name whenever you need it. |
| **Skill** | An instruction file that changes how an agent analyses something. Same agent, different analytical lens. |
| **Workspace** | The folder you have open in VS Code. Everything the AI can see and work with lives here. |
| **Frontmatter** | The bit between --- at the top of a markdown file. It tells VS Code metadata about the file (like "this is an agent"). |
| **JQL** | Jira Query Language — a way to search for tickets. The AI writes these automatically; you don't need to learn it. |
| **Confluence** | Atlassian's wiki/documentation platform. The Meeting Intelligence agent posts decision records there. |

---

## Quick Reference — File Checklist

When you're done, you should have these files:

```
.github/
├── copilot-instructions.md          ✅
├── agents/
│   ├── ticket-hygiene.prompt.md     ✅
│   ├── meeting-intelligence.agent.md ✅
│   ├── llm-wiki.agent.md           ✅
│   └── hygiene-history.json        ✅
└── skills/
    ├── PM Analysis/SKILL.md        ✅
    ├── EM Analysis/SKILL.md        ✅
    └── C-Level Analysis/SKILL.md   ✅

Transcripts/
├── Input/                           ✅ (empty, ready for transcripts)
└── Output/
    └── decisions-log.md            ✅ (created by agent on first run)

llm-wiki/
├── SCHEMA.md                       ✅
├── raw/                            ✅ (empty, ready for source docs)
└── wiki/
    ├── index.md                    ✅
    ├── log.md                      ✅
    ├── sources/                    ✅
    ├── entities/                   ✅
    ├── concepts/                   ✅
    ├── comparisons/                ✅
    └── analyses/                   ✅
```

---