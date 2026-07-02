# 🔍 RCA Analysis Expert Agent — VS Code + GitHub Copilot

A ready-to-use GitHub Copilot agent for **Root Cause Analysis (RCA)** of production
and test environment issues across the **Mastercraft** (COBOL/JCL/DB2/Mainframe/Q++)
and **Talon** (Java/Spring Boot/React) stacks.

---

## 📁 Folder Structure

```
.github/
  copilot-instructions.md         ← Agent persona, behavior rules, output format
  prompts/
    rca-intake.prompt.md          ← Triage prompt: collects incident details
    rca-diagram.prompt.md         ← Generates Mermaid root cause map
    rca-fix-options.prompt.md     ← Produces ranked Quick/Proper/Preventive fixes
    rca-jira-ticket.prompt.md     ← Drafts a Jira bug ticket from completed RCA
  skills/
    rca-expert/
      skill.md                    ← Skill descriptor + trigger keywords
      rca-checklist.md            ← Step-by-step investigation checklist
      abend-codes.md              ← IBM Mainframe ABEND code reference
      db2-sqlcodes.md             ← DB2 SQLCODE reference
      stack-patterns.md           ← Known error patterns with fix templates
.vscode/
  mcp.json                        ← MCP server connections (GitLab, Jira, Figma)
  settings.json                   ← VS Code Copilot settings
README.md                         ← This file
```

---

## 🚀 Setup Instructions

### Step 1 — Copy files into your GitLab repo

Copy the `.github/` and `.vscode/` folders into the **root of your workspace repo**
(either `mastercraft` or `talon` group — or a shared support repo).

If using a shared support workspace, create a new GitLab repo (e.g., `support/rca-agent`)
and copy both folders there. Then open that folder in VS Code.

### Step 2 — Set environment variables

Add these to your system environment or VS Code `.env` file:

```bash
export GITLAB_PAT=your_gitlab_personal_access_token
export FIGMA_PAT=your_figma_personal_access_token
```

> **GitLab PAT Scopes needed:** `read_api`, `read_repository`
> **Figma PAT:** Generate from Figma → Account Settings → Personal Access Tokens

### Step 3 — Connect the Atlassian MCP (Jira)

1. Open VS Code → Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`)
2. Search: **MCP: Add Server**
3. Search in the MCP Gallery for **Atlassian MCP Server**
4. Click Install → Authenticate via OAuth (your Jira account credentials)

### Step 4 — Enable Agent Mode in VS Code

1. Open VS Code Settings (`Ctrl+,`)
2. Search: `github.copilot.chat.agentMode.enabled`
3. Set to `true`

Alternatively — the `.vscode/settings.json` in this package already sets this automatically.

### Step 5 — Open Copilot Chat in Agent Mode

1. Click the Copilot Chat icon in VS Code sidebar
2. In the chat dropdown, select **Agent Mode**
3. You're ready. The agent will auto-load all instructions and skills.

---

## 💬 How to Use — Workflow

### Option A: Use prompt files (Recommended)

In Copilot Chat (Agent Mode), type `/` to see available prompts:

| Prompt | When to Use |
|--------|------------|
| `/rca-intake` | A new incident just came in — start here |
| `/rca-diagram` | You have enough info — generate the root cause map |
| `/rca-fix-options` | Root cause confirmed — get fix recommendations |
| `/rca-jira-ticket` | RCA complete — draft the Jira bug ticket |

### Option B: Freeform conversation

Just describe the problem in plain English. The agent will ask clarifying questions
automatically if critical details are missing.

**Example:**
```
"We're seeing an S0C7 ABEND in production on the PAYROLL batch job after yesterday's deployment."
```

The agent will respond with questions about which module, the exact ABEND location,
DB2 SQLCODEs if any, and recent changes — before starting analysis.

---

## 🗺️ RCA Output Format

Every completed analysis will produce:

1. **Incident Summary** — 2-3 line description
2. **Investigation Path** — COSTAR + 5-Whys reasoning
3. **Probable Root Causes Table** — ranked with confidence %
4. **Mermaid Root Cause Map** — visual flowchart (renders in VS Code Markdown Preview)
5. **Fix Options Table** — Quick / Proper / Preventive with effort and risk
6. **Jira Ticket Draft** — ready to paste into Jira

---

## 🔌 MCP Integrations

| MCP | Purpose | Auth |
|-----|---------|------|
| **GitLab MCP** | Fetch source files, commits, MR history from Mastercraft/Talon repos | PAT token |
| **Atlassian MCP** | Search past Jira tickets, create new bug tickets | OAuth |
| **Figma MCP** | Compare UI spec vs actual for visual discrepancies | PAT token |

> The agent will only call Figma MCP if a UI/visual issue is part of the incident.

---

## 🧠 Auto-Trigger Keywords

The RCA Expert Skill auto-loads when you use any of these words in chat:

`production error` · `ABEND` · `SQLCODE` · `exception` · `root cause` · `incident`
`investigation` · `stack trace` · `fix` · `hotfix` · `bug` · `crash` · `analysis`

---

## 📚 Reference Files (Inside the Skill)

| File | Contents |
|------|---------|
| `abend-codes.md` | S0C7, S0C4, S806, S222, S878 and more — with common fix areas |
| `db2-sqlcodes.md` | -803, -911, -805, -100, -302 and more — with fix strategies |
| `rca-checklist.md` | 5-phase checklist: Triage → Evidence → Hypothesis → Code → Output |
| `stack-patterns.md` | Pre-built patterns for S0C7, -803, LazyInitializationException, React API mismatch, HikariCP exhaustion |

---

## 🤝 Team Sharing

Since this entire setup lives inside `.github/` and `.vscode/`, anyone on your team who:
1. Clones or pulls this repo
2. Has GitHub Copilot with Agent Mode enabled in VS Code
3. Connects the MCPs once (one-time per developer)

...will **automatically** get the full agent with all instructions, prompts, and skills.
No individual configuration needed beyond MCP authentication.

---

## 🔄 Updating the Agent

To add new known error patterns: edit `.github/skills/rca-expert/stack-patterns.md`  
To add new prompt workflows: add a new `*.prompt.md` file in `.github/prompts/`  
To change the agent's behavior: edit `.github/copilot-instructions.md`  

Commit and push — all team members get the update on next `git pull`.

---

## 📋 Requirements

- VS Code 1.95 or later
- GitHub Copilot subscription (Team or Enterprise recommended for MCP support)
- GitLab Personal Access Token with `read_api` + `read_repository` scopes
- Atlassian account with Jira access (for Atlassian MCP OAuth)
- Figma account (optional — only for UI-related incidents)
