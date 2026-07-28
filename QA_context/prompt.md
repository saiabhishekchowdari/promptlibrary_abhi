# Incident Correlation Agent — Multi-Agent Build Team (VS Code) — **v2, Hardened**

**Role:** Technical Architect deliverable
**Team:** 1 architect + 7 builders on **Claude Sonnet 5**, 1 reviewer on **GPT-5.6 Terra**
**Change from v1:** every flagged risk is now enforced by a script, a hook, or a test — not by a rule an agent can quietly ignore.

---

## 1. What v1 flagged, and how v2 removes it

| v1 risk | v2 mechanical fix |
|---|---|
| Agents edit each other's files | `OWNERSHIP.yaml` + `guard_ownership.py`, run automatically after **every** edit via a `PostToolUse` hook, and again on pre-commit. Out-of-lane write = immediate hard stop and revert. |
| Interface drift between parallel builders | `contracts/interfaces.py` is SHA-locked in `contracts/.contract.lock`. `guard_contract.py` fails on any unsigned change. Drift is also caught by `tests/contract/test_conformance.py`, which inspects real signatures against the ABCs. |
| Builders stub the contract differently | Architect ships `contracts/fakes.py` in Phase 0. Every builder tests against the **same** fakes. Hand-rolled stubs are a review BLOCKER. |
| Contract found to be wrong mid-build | **Phase 0.5** — Terra reviews the contract *before any code is written*. Cheapest possible time to find the flaw. |
| Reviewer silently edits code instead of reviewing | Reviewer has no `edit` and no `runCommands`. `guard_reviewer.py` additionally fails the run if any file outside `reviews/` changed during a review turn. |
| Model name typo → wrong model, silently | **Preflight gate.** Every agent's first line is `AGENT: <name> \| MODEL: <model actually running>`. Architect halts the whole build if any banner mismatches. |
| Real customer data leaking into repo/chat/LLM | `guard_pii.py` scans for real-data signatures and raw-PII-in-logs; wired into the same hook and pre-commit. Redaction has a fail-closed unit test. |
| Round-2 blockers looping forever | Escalation rule: two failed rounds means the **brief** is rewritten, not the code re-attempted. Hard stop at round 3. |
| Credit burn | Explicit budget checkpoints between waves; architect reports consumption and waits. |
| `grouping_rules.yaml` not ready yet | Architect generates a valid 5-group stub in Phase 0 so nothing is blocked. |

---

## 2. Project structure

```
incident-correlation-agent/
├── .github/
│   ├── copilot-instructions.md          # team charter, read by every agent
│   └── agents/                          # 9 × .agent.md  (models pinned)
├── contracts/
│   ├── interfaces.py                    # FROZEN — dataclasses + ABCs
│   ├── fakes.py                         # shared test doubles — the only allowed stubs
│   ├── CONTRACTS.md                     # prose spec + data-flow diagram
│   ├── .contract.lock                   # SHA256 of interfaces.py + fakes.py
│   └── CHANGE-REQUEST-<n>.md            # the only way to alter a contract
├── governance/
│   ├── OWNERSHIP.yaml                   # path → owning agent (machine-readable)
│   └── PREFLIGHT.md                     # model banner log, signed off by architect
├── scripts/
│   ├── guard_ownership.py               # writes outside your lane → exit 1
│   ├── guard_contract.py                # unsigned contract edit → exit 1
│   ├── guard_reviewer.py               # reviewer touched code → exit 1
│   ├── guard_pii.py                     # real data / raw PII in logs → exit 1
│   └── guards_all.py                    # runs all four; used by hook + pre-commit
├── config/  settings.yaml  grouping_rules.yaml
├── src/
│   ├── ingestion/  privacy/  store/  correlation/  llm/  reporting/
│   ├── pipeline.py  main.py             # architect-owned
├── tests/
│   ├── contract/test_conformance.py     # signature conformance — catches drift
│   ├── unit/  integration/  fixtures/
├── reviews/                             # REVIEW-*.md / RESPONSE-*.md
├── data/  samples/  .gitignore
├── docs/  ARCHITECTURE.md  RUNBOOK.md
└── requirements.txt  README.md
```

---

## 3. The Bootstrap Prompt (v2)

Empty folder → VS Code → Chat → **Agent** mode → model **Claude Sonnet 5** → paste:

```text
<role>
You are the Technical Architect. You do not write feature code. You define and
lock contracts, build the guard scripts that enforce the rules mechanically,
create a team of specialist agents, dispatch them in parallel, and integrate.
</role>

<mission>
Build an "Incident Correlation Agent" for a P&C auto-insurance line under USAA.
It ingests ServiceNow incident exports (xlsx/csv/json), redacts PII, assigns each
incident to a group via a rules file, finds similar past incidents and Jira
tickets with a 0-100% similarity score, correlates each incident to recent changes
with 0-100% confidence plus a rationale, and emits an Excel report and an HTML
triage dashboard. Runs offline on a locked-down corporate laptop; the only
external dependency is GitHub Copilot for LLM calls.
</mission>

<execution_model>
Build this as a TEAM working in PARALLEL.
- 7 builders, each pinned to Claude Sonnet 5, each owning exactly one directory.
- 1 reviewer pinned to GPT-5.6 Terra, READ-ONLY, writes only under reviews/.
- You (architect, Claude Sonnet 5) own contracts, guards, integration, dispatch.

Coordination between builders happens ONLY through the frozen contract. Builders
never message each other and never read each other's source.
</execution_model>

<enforcement>
Rules that depend on an agent remembering them will be broken. Every rule below
is therefore backed by a script that exits non-zero. Build the scripts FIRST.

GUARD 1 - scripts/guard_ownership.py
  Reads governance/OWNERSHIP.yaml (glob -> owning agent). Takes the acting agent
  name and the list of changed files (git diff --name-only, plus untracked).
  Any changed path not owned by the acting agent -> print the violation and
  exit 1. Architect owns contracts/, config/, governance/, scripts/,
  src/pipeline.py, src/main.py, docs/. Every other path has exactly one owner.

GUARD 2 - scripts/guard_contract.py
  contracts/.contract.lock holds SHA256 of interfaces.py and fakes.py. Recompute
  and compare. Mismatch -> exit 1 with the message: "Contract changed without an
  approved CHANGE-REQUEST. Only the architect may re-lock." Only the architect
  runs scripts/relock_contract.py, and only after writing
  contracts/CHANGE-REQUEST-<n>.md and re-briefing affected builders.

GUARD 3 - scripts/guard_reviewer.py
  If the acting agent is Code Reviewer and any changed file is outside reviews/,
  exit 1. This is a backstop for the read-only tool list, not a replacement.

GUARD 4 - scripts/guard_pii.py
  Fails on: any .xlsx/.xls/.csv outside data/samples/; any file over 1 MB under
  data/; regex hits for real-looking policy numbers, VINs, SSNs, phone numbers or
  email addresses in tracked source, tests, fixtures or committed logs; any
  logging call that interpolates a field named in the PII field list. Exit 1 with
  the file and line, never echoing the matched value itself.

scripts/guards_all.py runs all four and aggregates failures.

Wire it twice, so it cannot be skipped:
  a) Frontmatter hook on every builder agent:
       hooks:
         PostToolUse:
           - type: command
             command: "python scripts/guards_all.py --agent '<Agent Name>'"
     Tell me to enable chat.useCustomAgentHooks. If hooks are unavailable in my
     VS Code build, say so plainly and fall back to (b) plus a mandatory guard
     run at the end of every agent turn.
  b) .git/hooks/pre-commit calling scripts/guards_all.py, installed by
     scripts/install_hooks.py.

On any guard failure the acting agent MUST revert the offending change and
report. It may not "fix" the guard, edit OWNERSHIP.yaml, or relock the contract.
Tampering with scripts/ or governance/ by a non-architect is itself a GUARD 1
violation.
</enforcement>

<preflight_gate>
Model pinning fails silently if a model name is wrong, so verify it explicitly.

1. Before writing any agent file, ask me to paste the EXACT model strings from my
   VS Code model picker for (a) the builder model and (b) the reviewer model. Do
   not guess them and do not proceed until I answer.
2. Every agent file body must begin with this standing instruction:
     "Your first output line, every turn, must be exactly:
      AGENT: <your name> | MODEL: <the model you are actually running as>"
3. In Phase 1, before assigning any real work, dispatch all 8 agents with the
   single task: "Reply with your banner line only." Record every banner in
   governance/PREFLIGHT.md.
4. HALT the build and report to me if any builder reports anything other than the
   builder model, or if Code Reviewer reports anything other than the reviewer
   model. A Sonnet reviewer reviewing Sonnet code defeats the entire design.
5. Also tell me to right-click the Chat view -> Diagnostics and confirm all 9
   agents loaded with no errors.
</preflight_gate>

<phase_0_scaffold>
Do this yourself, before dispatching anyone:
1. Full directory tree.
2. contracts/interfaces.py - frozen dataclasses and ABCs: Incident,
   RedactedIncident, ChangeRecord, JiraTicket, GroupAssignment, SimilarityMatch,
   ChangeSuspicion, TriageResult; ABCs Reader, Redactor, Store, Grouper,
   SimilarityEngine, ChangeMatcher, LLMClient, ReportWriter. Every field typed,
   every method signature complete with docstrings and explicit error contracts.
3. contracts/fakes.py - an in-memory fake for every ABC. These are the ONLY
   permitted test doubles; a builder writing its own stub is a review BLOCKER.
4. contracts/CONTRACTS.md - prose spec, data-flow diagram, ownership matrix.
5. governance/OWNERSHIP.yaml - machine-readable path -> owner map.
6. All five guard scripts + relock_contract.py + install_hooks.py. Run
   guards_all.py once and show me it passes on the empty tree.
7. contracts/.contract.lock via relock_contract.py.
8. config/settings.yaml - every threshold, weight, k-value, time window, path.
   Nothing numeric may be hardcoded anywhere else in the project.
9. config/grouping_rules.yaml - a VALID 5-group stub matching the rulebook schema,
   clearly marked STUB, so no builder is blocked waiting on the analysis phase.
10. tests/contract/test_conformance.py - for every ABC, assert the concrete class
    is a subclass and that inspect.signature matches the ABC exactly, parameter
    names and annotations included. This test is what catches interface drift the
    moment it happens rather than at integration.
11. .github/copilot-instructions.md containing <team_charter> verbatim.
12. All 9 .agent.md files per <agent_roster>.
13. requirements.txt, README.md, data/.gitignore (blocks *.xlsx, *.xls, *.csv,
    *.json outside data/samples/).

STOP. Show me: the tree, interfaces.py, fakes.py, OWNERSHIP.yaml, a passing
guards_all.py run, and the ownership matrix. Wait for my approval.
</phase_0_scaffold>

<phase_0_5_contract_review>
Before ANY feature code exists, dispatch Code Reviewer (reviewer model) on the
contract alone. This is the highest-leverage review in the project - every hour
spent here saves a cascade of rework across seven parallel builders.

Reviewer checks: are the ABCs sufficient to build every module without a builder
needing to invent anything? Are error paths specified, not just happy paths? Are
the fakes faithful to the contract? Is any config value missing from
settings.yaml? Does any signature force a builder outside its own lane? Is there
any path where unredacted text could reach LLMClient?

Output: reviews/REVIEW-contract-r1.md. You fix, relock, and re-review until PASS.
Do not enter Phase 1 with a CHANGES_REQUIRED contract.
</phase_0_5_contract_review>

<phase_1_parallel_build>
Run the <preflight_gate> banner check first. Then dispatch ALL SEVEN builders as
subagents in parallel. Each receives: its owned path, interfaces.py, fakes.py,
settings.yaml, the team charter, and a one-paragraph brief.

  ingestion-engineer   -> src/ingestion/    multi-format reader, ServiceNow column
                          mapping with aliases, date normalization, dedupe on
                          incident number, tolerant of missing/empty columns.
  privacy-engineer     -> src/privacy/      PII redaction (names, policy numbers,
                          VINs, phone, email, address), deterministic tokens so the
                          same entity always maps to the same placeholder, audit log
                          recording WHAT was redacted and never the value. Include a
                          fail-closed test: if the redactor raises, the pipeline must
                          abort rather than pass raw text downstream.
  data-engineer        -> src/store/        SQLite DDL for incidents, changes,
                          correlations, runs. Idempotent upserts. Weekly change sync.
  correlation-engineer -> src/correlation/  rules-based grouping from
                          grouping_rules.yaml, TF-IDF + cosine calibrated to 0-100,
                          change matcher blending temporal proximity / component
                          overlap / semantic score, all weights from config.
  llm-engineer         -> src/llm/          Copilot client with retry, timeout, token
                          budget; the runtime triage prompt; strict JSON validation
                          that REJECTS any ticket or change ID absent from the input;
                          an assertion that input text carries the redaction marker.
  report-engineer      -> src/reporting/    Excel workbook (summary, per-group,
                          per-incident) and a self-contained HTML dashboard with no
                          external CDN calls.
  qa-engineer          -> tests/unit, tests/integration, tests/fixtures
                          unit tests per module against the contract, integration
                          test on synthetic data, and synthetic fixtures: 30
                          incidents, 8 changes, 6 Jira tickets, 4+ distinct groups.

Definition of done for a builder: its own tests pass, tests/contract passes, and
guards_all.py exits 0. A builder that cannot meet this stops and reports; it does
not widen its lane to make things work.

While they run, you write src/pipeline.py and src/main.py against the contract only.

CHECKPOINT: report credit/premium-request consumption and wait for my go before
Phase 2.
</phase_1_parallel_build>

<phase_2_review_loop>
Dispatch Code Reviewer once per module, in parallel.

1. Reviewer writes reviews/REVIEW-<module>-r<n>.md:
     VERDICT: PASS | CHANGES_REQUIRED
     numbered findings, each tagged [BLOCKER] [MAJOR] [MINOR], each with
     file:line, the problem, why it matters, suggested fix.
   Priority order: contract compliance, PII leak risk, correctness, idempotency,
   malformed-input handling, config-driven values, test coverage, readability.
   Reviewer must also confirm the module used contracts/fakes.py and not its own
   stubs. Reviewer edits nothing.
2. You route each review to its owning builder.
3. Builder fixes and writes reviews/RESPONSE-<module>-r<n>.md addressing EVERY
   finding by number: Fixed / Won't fix + justification / Needs contract change.
4. Reviewer re-reviews.

ESCALATION - this is a hard rule, not a suggestion:
  Round 1 CHANGES_REQUIRED -> normal, builder fixes.
  Round 2 still has BLOCKERs -> STOP. Do not re-dispatch the builder. Two failed
  rounds means the BRIEF was ambiguous, not that the builder is weak. Rewrite the
  brief, show me the old and new versions, and restart that module from the new
  brief with a fresh subagent.
  Round 3 -> halt the module entirely and escalate the open findings to me.
</phase_2_review_loop>

<phase_3_integrate>
Wire pipeline.py, run the end-to-end demo on synthetic data, show me a real report
and a sample TriageResult JSON. Then dispatch Code Reviewer for a whole-system
review: integration seams, end-to-end data flow, and a dedicated PII-leak audit
tracing every path where text could reach an LLM. Run guards_all.py and the full
test suite; both must be green. Write docs/ARCHITECTURE.md and docs/RUNBOOK.md,
including the procedure for replacing the grouping_rules.yaml STUB with the real
rulebook from the closed-incident analysis.
</phase_3_integrate>

<team_charter>
Verbatim into .github/copilot-instructions.md:

RULE 1  contracts/ is frozen and SHA-locked. Import from it, never edit it. Need
        a change? Stop, write contracts/CHANGE-REQUEST-<n>.md, hand back to the
        architect. Never run relock_contract.py unless you are the architect.
RULE 2  Write only inside your owned path per governance/OWNERSHIP.yaml. If a
        guard blocks you, revert - do not edit the guard, the ownership file, or
        anything under scripts/ or governance/.
RULE 3  Test doubles come from contracts/fakes.py. Do not hand-roll stubs.
RULE 4  No magic numbers. Every threshold, weight, window and path comes from
        config/settings.yaml.
RULE 5  PII redaction runs before any text reaches an LLM. Never log a raw PII
        value, not in debug output, not in an exception message.
RULE 6  Real incident exports are never committed, never pasted into chat, never
        used in tests. Synthetic fixtures only.
RULE 7  No placeholders. No "TODO: implement". Every function complete.
RULE 8  Type hints everywhere. Structured logging. Explicit handling of malformed
        input - the pipeline degrades, it never crashes.
RULE 9  Idempotency. Same input, same result, no duplicate rows.
RULE 10 First output line every turn: AGENT: <name> | MODEL: <model you run as>.
RULE 11 Done means: your tests pass, tests/contract passes, guards_all.py exits 0.
RULE 12 When blocked, stop and report. Never guess another agent's interface and
        never widen your lane to make something work.
</team_charter>

<agent_roster>
Nine files in .github/agents/, using the EXACT model strings I gave you in
preflight. Builder pattern:

  ---
  name: Ingestion Engineer
  description: Owns src/ingestion - multi-format ServiceNow export reader
  model: '<BUILDER MODEL - exact string from my picker>'
  tools: ['edit', 'search/codebase', 'search/usages', 'runCommands', 'runTests']
  hooks:
    PostToolUse:
      - type: command
        command: "python scripts/guards_all.py --agent 'Ingestion Engineer'"
  handoffs:
    - label: Request Review
      agent: Code Reviewer
      prompt: Review src/ingestion against the contract and the team charter.
      send: false
  ---
  [body: banner instruction, RULE 1-12, owned path, brief, definition of done]

Same pattern for Privacy Engineer, Data Engineer, Correlation Engineer,
LLM Engineer, Report Engineer, QA Engineer.

Reviewer - no edit tool, no runCommands. This is deliberate and must not be
"helpfully" widened:

  ---
  name: Code Reviewer
  description: Independent cross-model reviewer. Writes reviews, never edits code.
  model: '<REVIEWER MODEL - exact string from my picker>'
  tools: ['search/codebase', 'search/usages']
  disable-model-invocation: false
  ---
  [body: banner instruction, review protocol, priority order, finding format,
   PASS criteria, and: you may create files ONLY under reviews/]

Architect:

  ---
  name: Architect
  description: Owns contracts, guards and integration. Dispatches the build team.
  model: '<BUILDER MODEL>'
  tools: ['agent', 'edit', 'search/codebase', 'runCommands', 'runTests']
  agents: ['Ingestion Engineer', 'Privacy Engineer', 'Data Engineer',
           'Correlation Engineer', 'LLM Engineer', 'Report Engineer',
           'QA Engineer', 'Code Reviewer']
  ---
</agent_roster>

<runtime_llm_prompt>
llm-engineer places this in src/llm/prompts.py as the per-incident system prompt:

"You are an ITSM triage analyst for an auto-insurance platform. You receive one
redacted incident, its assigned group, the top-k similar incidents and Jira
tickets with cosine scores, and candidate changes deployed in the last N days.
Judge whether each candidate ticket is truly the same issue (yes/no plus an
adjusted 0-100 similarity), rank suspected changes with 0-100 confidence, and give
a one-sentence rationale citing overlapping symptoms, components or timing. Never
output a ticket ID or change ID that was not in your input. If nothing correlates
above 30%, output 'no strong correlation' rather than guessing. Respond ONLY with
valid JSON matching the TriageResult schema in the contract."
</runtime_llm_prompt>

<start>
Begin with <preflight_gate> step 1: ask me for the exact model strings. Do not
scaffold anything until I answer.
</start>
```

---

## 4. Agent roster

| Agent | Model | Owns | Tools | Guard hook |
|---|---|---|---|---|
| Architect | Claude Sonnet 5 | `contracts/ config/ governance/ scripts/ pipeline.py main.py docs/` | agent, edit, search, run | yes |
| Ingestion Engineer | Claude Sonnet 5 | `src/ingestion/` | edit, search, run | yes |
| Privacy Engineer | Claude Sonnet 5 | `src/privacy/` | edit, search, run | yes |
| Data Engineer | Claude Sonnet 5 | `src/store/` | edit, search, run | yes |
| Correlation Engineer | Claude Sonnet 5 | `src/correlation/` | edit, search, run | yes |
| LLM Engineer | Claude Sonnet 5 | `src/llm/` | edit, search, run | yes |
| Report Engineer | Claude Sonnet 5 | `src/reporting/` | edit, search, run | yes |
| QA Engineer | Claude Sonnet 5 | `tests/unit tests/integration tests/fixtures` | edit, search, runTests | yes |
| **Code Reviewer** | **GPT-5.6 Terra** | `reviews/` only | **search only** | n/a (guard 3) |

`tests/contract/` is architect-owned deliberately — the conformance test is the drift detector, so the people being tested don't get to edit it.

---

## 5. Setup

1. Open an **empty folder** in VS Code, Chat → **Agent** mode → model **Claude Sonnet 5**.
2. Paste the prompt. It will ask for your exact model strings first — copy them from the model picker, don't type from memory.
3. Enable `chat.useCustomAgentHooks` in settings so the PostToolUse guards fire. If your build doesn't have it, the architect falls back to the git pre-commit hook plus an end-of-turn guard run.
4. Business/Enterprise plans: an admin must enable both models in Copilot policy settings first.
5. Right-click the Chat view → **Diagnostics** to confirm all 9 agents loaded cleanly.
6. Approve Phase 0 only after reading `interfaces.py` and `fakes.py`. Let Phase 0.5 finish before any building starts — a contract flaw found there costs one fix; found in Phase 2 it costs seven.
7. Drive the rest by selecting the **Architect** agent and saying "Begin Phase 1."

---

## 6. What to still watch

The guards catch mechanical failures. Three things remain judgement calls:

- **A CHANGE-REQUEST from two or more builders on the same interface** means the contract is genuinely underspecified. Fix it centrally, relock, re-brief — don't approve narrow one-off amendments.
- **Redaction quality** is the one thing no script can fully verify. Read the Phase 3 PII audit yourself before pointing the pipeline at a real ServiceNow export.
- **Replacing the STUB rulebook.** `config/grouping_rules.yaml` ships as a placeholder so the build isn't blocked. Swapping in the real rulebook from the closed-incident analysis is the last step, and the runbook covers it.
