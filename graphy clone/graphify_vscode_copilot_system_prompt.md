# Graphify System Prompt — VS Code GitHub Copilot (Local, Data-Secure)

> **Purpose:** This is the master system prompt and implementation guide for integrating Graphify into VS Code with GitHub Copilot (Claude Opus model), designed for corporate environments where all data must remain on-machine. It uses the COSTAR + RISEN hybrid prompting framework for maximum effectiveness.

---

## 1. Project Context & Architecture

### What Graphify Does

Graphify (`graphifyy` on PyPI) is a codebase knowledge-graph tool that:

1. **Extracts** your entire codebase (code, docs, PDFs, images) into a structured knowledge graph using tree-sitter ASTs (fully local, zero API calls for code).
2. **Produces three artefacts** in `graphify-out/`:
   - `graph.json` — the persistent knowledge graph (query any time, no re-read needed)
   - `graph.html` — interactive browser visualisation (click nodes, filter, search)
   - `GRAPH_REPORT.md` — highlights: god nodes, surprising connections, suggested questions
3. **Enables token-efficient queries** — instead of re-reading raw files every turn, Copilot queries the pre-built graph using `graphify query "<question>"`, retrieving only the relevant subgraph.

### Why This Matters for You (Token Reduction)

Without a graph: Copilot reads file-by-file on every new session → thousands of tokens wasted.  
With a graph: Copilot calls `graphify query` → gets a precise subgraph of relevant nodes → answers with 10–40× fewer tokens consumed.

### Data Privacy Architecture (Corporate-Safe)

```
Your Machine (Mac mini M4)
├── graphify-out/
│   ├── graph.json          ← all graph data, local only
│   ├── graph.html          ← browser visualisation, local only
│   └── GRAPH_REPORT.md     ← text report, local only
├── graphify-out/cache/     ← extraction cache (AST), local only
└── .graphifyignore         ← exclude secrets, env files, etc.

DATA FLOW:
  Code files → tree-sitter AST extraction (100% local, NO network)
  graph.json → graphify query (local binary, NO network)
  Copilot Chat → reads graph.json / GRAPH_REPORT.md (local file read)
```

**Critical:** Use `--backend ollama` with a local Ollama model for doc/PDF extraction to keep ALL data on-machine. Never use `--backend claude` or `--backend openai` in a corporate context.

---

## 2. The System Prompt (COSTAR + RISEN Framework)

Copy this verbatim into your VS Code Copilot Chat instructions file (`.github/copilot-instructions.md` or `.vscode/settings.json → github.copilot.chat.codeGeneration.instructions`).

---

```markdown
<!-- ============================================================
  GRAPHIFY COPILOT SYSTEM PROMPT
  Framework: COSTAR (Context, Objective, Style, Tone, Audience, Response)
             + RISEN (Role, Instructions, Steps, End-goal, Narrowing)
  Version: 1.0  |  Data Residency: LOCAL ONLY — no external API calls
  ============================================================ -->

## [CONTEXT]
You are operating inside a VS Code workspace that has been indexed into a
persistent knowledge graph using Graphify (graphifyy). The graph artefacts are
located at:
  - graphify-out/graph.json      → full knowledge graph (primary source of truth)
  - graphify-out/GRAPH_REPORT.md → architecture overview, god nodes, key connections
  - graphify-out/graph.html      → browser visualisation (reference only)

The graph was built with: `graphify extract . --backend ollama`
All data is local. No file content should be sent to any external service.

## [OBJECTIVE]
Your primary objective is to answer codebase questions and assist with code
tasks by querying the pre-built knowledge graph FIRST — before reading any
raw source files. This dramatically reduces token usage and keeps responses
fast and precise.

## [ROLE] (RISEN)
You are a senior software architect and QA automation expert who specialises
in graph-augmented code navigation. You understand that the knowledge graph
is a compressed, queryable representation of the entire codebase, and you
prefer surgical graph queries over broad file reads.

## [INSTRUCTIONS] (RISEN)

### Graph-First Navigation Protocol

ALWAYS follow this priority order when answering codebase questions:

  STEP 1 — GRAPH QUERY (preferred, minimal tokens)
    Run: graphify query "<your question about the code>"
    Use for: "what does X connect to?", "how does auth flow work?",
             "which modules depend on Y?", "find all usages of Z"

  STEP 2 — REPORT REVIEW (for broad architecture questions)
    Read: graphify-out/GRAPH_REPORT.md
    Use for: "give me an overview", "what are the god nodes?",
             "what are surprising connections?"

  STEP 3 — TARGETED FILE READ (only when graph query is insufficient)
    Read: the specific file(s) the graph query identified
    Never read files speculatively or scan directories first.

  STEP 4 — NEVER do a full directory scan or read all files.
    Forbidden patterns: reading every file in a folder, grep across
    all files without a graph-guided starting point.

### Query Formulation Rules

When calling `graphify query`, formulate queries as:
  - Natural-language questions: "what connects UserService to DatabasePool?"
  - Entity lookups: `graphify explain "RateLimiter"`
  - Path tracing: `graphify path "AuthModule" "SessionStore"`
  - Neighbour inspection: `graphify query "<entity> neighbours"`

Always include `--graph graphify-out/graph.json` if the default path is not set.

### Token Budget Discipline

- Max context from graph queries: 2,000 tokens per query response
- If a graph response exceeds this, summarise before continuing
- Never include raw file content in your context window unless the graph
  query explicitly points to a section < 100 lines
- Prefer node IDs and edge labels over full file dumps

### Code-Only (AST) vs. Semantic Nodes

The graph contains two node types. Treat them differently:
  - AST nodes (EXTRACTED): high confidence, use directly
  - Semantic nodes (INFERRED / AMBIGUOUS): verify with a targeted file read
    before modifying code based on them

### Data Residency Enforcement

  ✓ ALLOWED: reading local files, running local binaries (graphify, ollama)
  ✗ FORBIDDEN: calling any external API, sending file content to cloud,
               suggesting `--backend claude` or `--backend openai` for extraction

If the user asks to use a cloud backend for extraction, remind them of the
corporate data-residency policy and suggest `--backend ollama` instead.

## [STYLE]
- Concise: lead with the answer, support with graph evidence
- Structured: use code blocks for commands, tables for node comparisons
- Traceable: cite the graph node or edge that supports each claim
- Example: "According to graph node `AuthService` (degree: 14, cluster: auth),
  it connects to `SessionStore` via a `calls` edge and `UserRepo` via `depends_on`."

## [TONE]
Professional and precise. You are advising a QA automation engineer who has
deep technical knowledge. Skip basic explanations unless asked.

## [AUDIENCE]
QA Automation Engineer / Software Developer working in a corporate environment.
Advanced technical knowledge. Uses VS Code + GitHub Copilot + Mac mini M4.
Privacy and data security are non-negotiable constraints.

## [RESPONSE FORMAT] (RISEN End-goal)

For every codebase question, structure your response as:

  ### Graph Evidence
  <result from graphify query or GRAPH_REPORT.md>

  ### Analysis
  <your interpretation of the graph result>

  ### Action / Code
  <code change, command, or recommendation>

  ### Confidence
  EXTRACTED | INFERRED | AMBIGUOUS
  (based on the confidence tags in the graph nodes)

## [NARROWING] (RISEN)
- Scope: only this workspace's graph (do not hallucinate relationships)
- If the graph returns no results: say so explicitly, then suggest a targeted
  file read as a fallback
- If the graph is stale (files changed since last extract): remind the user
  to run `graphify update . --backend ollama` before answering
- Never invent node names or edges not present in the graph response
```

---

## 3. Installation & Setup (Local, Corporate-Safe)

### Prerequisites

```bash
# Check Python version (3.10+ required)
python3 --version

# Install uv (recommended — manages PATH automatically)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Verify
uv --version
```

### Install Graphify with Ollama Backend

```bash
# Install graphifyy with ollama support (data stays local)
uv tool install "graphifyy[ollama]"

# Verify CLI is on PATH
graphify --version
```

> **PyPI note:** The package name is `graphifyy` (double-y). The CLI command is `graphify`.

### Install Ollama (Local LLM for doc/PDF extraction)

```bash
# macOS
brew install ollama

# Start Ollama service
ollama serve &

# Pull a model suitable for semantic extraction (choose based on your RAM)
ollama pull mistral          # 4GB VRAM — good for doc extraction
ollama pull llama3.2         # 2GB VRAM — lightweight option
ollama pull qwen2.5-coder    # best for code-heavy semantic extraction
```

### Configure Environment Variables (local, no secrets leave machine)

Create `.env.graphify` in your project root (add to `.gitignore`):

```bash
# .env.graphify — source this before running graphify extract
export OLLAMA_BASE_URL=http://localhost:11434
export OLLAMA_MODEL=qwen2.5-coder   # or mistral, llama3.2
export GRAPHIFY_MAX_WORKERS=4        # adjust for Mac mini M4 CPU cores
export GRAPHIFY_QUERY_LOG_DISABLE=1  # disable query logging for privacy
```

Source it:
```bash
source .env.graphify
```

### Create .graphifyignore (exclude secrets)

```bash
# .graphifyignore — keep sensitive files out of the graph
.env
.env.*
*.pem
*.key
*.p12
*.pfx
secrets/
credentials/
node_modules/
dist/
build/
.git/
graphify-out/cache/
__pycache__/
*.pyc
```

### Build the Knowledge Graph

```bash
# Source env first
source .env.graphify

# Build graph — code is extracted locally (AST, no API), docs go via Ollama
graphify extract . --backend ollama --max-concurrency 2

# Output will be in graphify-out/
ls graphify-out/
# graph.json  graph.html  GRAPH_REPORT.md  cache/  manifest.json
```

### Install the VS Code Copilot Skill

```bash
# Register graphify skill with VS Code Copilot Chat
graphify vscode install
```

This writes the skill instruction file that VS Code Copilot Chat reads on every session.

### Set Up Auto-Rebuild on Git Commit (AST only, no API cost)

```bash
# Install post-commit hook — rebuilds AST graph on every commit (local, free)
graphify hook install
```

---

## 4. VS Code Configuration

### copilot-instructions.md Placement

Place the system prompt above in one of these locations (pick one):

```
Option A (repo-level, shareable with team):
  .github/copilot-instructions.md

Option B (workspace-level, local only):
  .vscode/copilot-instructions.md
```

### settings.json — Claude Opus Model Selection

In VS Code `settings.json`:

```json
{
  "github.copilot.chat.codeGeneration.instructions": [
    {
      "file": ".github/copilot-instructions.md"
    }
  ],
  "github.copilot.selectedModel": "claude-opus-4-8",
  "github.copilot.chat.localeOverride": "en"
}
```

### MCP Server (Optional — for persistent graph access across sessions)

Start the MCP server locally so Copilot has structured tool access:

```bash
# Start local MCP server (stays on machine, loopback only)
python -m graphify.serve graphify-out/graph.json --transport http \
  --host 127.0.0.1 --port 8080
```

Add to VS Code MCP config (`mcp.json`):

```json
{
  "mcpServers": {
    "graphify": {
      "url": "http://127.0.0.1:8080/mcp",
      "transport": "http"
    }
  }
}
```

MCP tools available: `query_graph`, `get_node`, `get_neighbors`, `shortest_path`.

---

## 5. Daily Workflow

### Session Start Checklist

```bash
# 1. Ensure Ollama is running (for any doc updates)
ollama serve &

# 2. Check if graph is stale (files changed since last extract)
graphify check-update .

# 3. Update only changed files (fast, incremental)
graphify update . --backend ollama

# 4. Open VS Code — Copilot reads the graph automatically
code .
```

### Common Copilot Chat Commands (Graph-Driven)

In VS Code Copilot Chat (`Ctrl+Shift+I`):

```
/graphify .                          → rebuild full graph
/graphify query "how does auth work" → targeted graph query
/graphify explain "TestRunner"       → explain a specific node
/graphify path "LoginPage" "Database" → trace dependency path
```

### Updating After Code Changes

```bash
# Re-extract only modified files (preserves rest of graph)
graphify update . --backend ollama --max-concurrency 2

# Force full rebuild (after major refactors)
graphify extract . --backend ollama --force
```

---

## 6. Team Sharing (Corporate-Safe)

When sharing with your team, commit `graphify-out/` (excluding `cache/` and `cost.json`):

```bash
# Add to .gitignore (already present from .graphifyignore setup)
echo "graphify-out/cost.json" >> .gitignore
echo "graphify-out/cache/" >> .gitignore   # optional: commit for speed

# Commit the graph (so teammates start with a ready map)
git add graphify-out/graph.json graphify-out/GRAPH_REPORT.md
git commit -m "feat: add graphify knowledge graph"
```

Team members pull, open VS Code, and Copilot reads the graph immediately — no rebuild needed.

**For teammates:** run once after cloning:

```bash
uv tool install "graphifyy[ollama]"
graphify vscode install
```

---

## 7. Maintenance & Troubleshooting

| Symptom | Fix |
|---|---|
| `graphify: command not found` | `uv tool install graphifyy` — uv manages PATH |
| Graph has stale nodes after refactor | `graphify extract . --force --backend ollama` |
| Duplicate nodes for same entity | `graphify extract . --force` (auto-merged in v0.8.33+) |
| Copilot not reading graph | Re-run `graphify vscode install`, restart VS Code |
| Ollama context window exceeded | Set `GRAPHIFY_OLLAMA_NUM_CTX=8192` in `.env.graphify` |
| MCP server not responding | Restart: `python -m graphify.serve graphify-out/graph.json --transport http --host 127.0.0.1 --port 8080` |
| Graph too large (>5000 nodes) | Use `--no-viz` flag; query via CLI only |

---

## 8. Framework Rationale

### Why COSTAR + RISEN?

| Component | Role in This Prompt |
|---|---|
| **C**ontext | Establishes the graph artefact locations and local-only constraint upfront |
| **O**bjective | Token reduction + graph-first navigation as the primary goal |
| **S**tyle | Concise, traceable, node-cited responses |
| **T**one | Professional peer-level for an advanced engineer |
| **A**udience | QA automation engineer, corporate privacy constraints |
| **R**esponse format | Structured: Graph Evidence → Analysis → Action → Confidence |
| **R**ole (RISEN) | Senior architect specialising in graph-augmented navigation |
| **I**nstructions (RISEN) | Explicit priority order: query → report → file → never scan |
| **S**teps (RISEN) | 4-step protocol with forbidden patterns |
| **E**nd-goal (RISEN) | Consistent structured output with confidence tagging |
| **N**arrowing (RISEN) | Scope to local graph only; stale-graph reminders |

The dual-framework approach ensures both the AI's *behaviour* (RISEN's procedural steps) and the AI's *output quality* (COSTAR's style/tone/audience calibration) are explicitly constrained — reducing hallucinated relationships and improving answer reliability.

---

## Here could observe LLM connection through Ollama but please modify such that it uses LLM details like agent from Github co-pilot chat a we have ##enterprise subscription to github co-pilot. Please re-write everything according this criteria.