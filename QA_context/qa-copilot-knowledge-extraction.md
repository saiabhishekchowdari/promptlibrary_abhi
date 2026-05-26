# QA Copilot Knowledge Extraction Prompts

A collection of prompts to extract and document knowledge from a GitHub Copilot-tuned QA account into reusable Markdown files. Use these prompts directly in VS Code with GitHub Copilot Chat.

---

## How to Use

1. Open GitHub Copilot Chat in VS Code (`Ctrl+Shift+I`)
2. Run each prompt in the QA's Copilot session
3. Copy the output into the corresponding `.md` file
4. Save files under `/docs/qa-context/` in your repository

| Output File | Prompt Section | When to Run |
|---|---|---|
| `memory-snapshot.md` | [Prompt 1](#prompt-1-memory-export) | Periodically, or before sprint ends |
| `project-handoff.md` | [Prompt 2](#prompt-2-project-analysis--handoff) | At milestone or team handoff |
| `qa-coding-style.md` | [Prompt 3](#prompt-3-writing--implementation-style) | Once, then update when patterns change |

> **Tip:** Run Prompt 3 first (style changes slowly), then Prompt 2 (project structure), and Prompt 1 last (memory is most time-sensitive).

---

## Prompt 1: Memory Export

**Purpose:** Capture everything Copilot has learned about this project from the QA's session history.  
**Output file:** `memory-snapshot.md`

```
List everything you have stored in your memory regarding this project and output it as a Markdown list in a code block so I can save it to a file.

Include:
- Project name and purpose
- Test scope (modules, features, environments)
- Known bugs, flaky tests, or skip conditions
- Naming conventions for test files, functions, and variables
- Any Playwright-specific configuration preferences (browser, timeout, retries)
- CI/CD pipeline details if known
- Any recurring prompts or patterns I frequently ask you about
```

---

## Prompt 2: Project Analysis & Handoff

**Purpose:** Generate a full project handoff document from Playwright test files and project structure.  
**Output file:** `project-handoff.md`

```
Analyze all the Playwright test files and project structure you have context on and generate a comprehensive project handoff document in Markdown format.

Structure it as follows:

# Project Handoff: [Project Name]

## Project Overview
- Purpose, tech stack, test framework versions

## Folder & File Structure
- Directory tree with brief description of each folder/file role

## Test Coverage Map
- List of features/modules covered by existing tests
- Any known gaps in coverage

## Test Execution
- How to run tests locally (commands, env variables)
- CI/CD integration steps

## Key Playwright Patterns Used
- Page Object Model usage (if any)
- Fixtures, hooks, helper utilities
- Screenshot/trace capture setup

## Known Issues & Workarounds
- Flaky tests, environment-specific conditions, skip reasons

## Onboarding Notes
- What a new QA engineer needs to know in the first week

Output everything inside a single Markdown code block so it can be saved directly as project-handoff.md
```

---

## Prompt 3: Writing & Implementation Style

**Purpose:** Capture the QA's personal coding style, prompt patterns, and Playwright implementation preferences for use as an AI agent instruction file.  
**Output file:** `qa-coding-style.md`

```
Based on all the Playwright scripts, prompts, and conversations you have seen from me, generate a detailed writing style and implementation guide in Markdown format that captures how I work.

Structure it as:

# QA Coding Style & Implementation Guide

## Prompt Writing Style
- How I phrase test requests (imperative, declarative, scenario-based?)
- Level of detail I provide in prompts
- Common phrases or patterns I use

## Test Structure Preferences
- How I organize describe/test blocks
- Naming conventions for test files, suites, and individual tests
- Preferred use of beforeEach, afterEach, beforeAll, afterAll

## Playwright Implementation Patterns
- Locator strategies I prefer (role, label, data-testid, CSS, XPath priority order)
- How I handle waits and assertions
- How I structure Page Object files (if used)
- Error handling and retry logic patterns

## Code Style
- Variable naming (camelCase, descriptive, short?)
- Comment style (inline, block, JSDoc?)
- How I handle test data (hardcoded, fixtures, env vars?)

## AI Collaboration Patterns
- Types of tasks I delegate to Copilot vs. write myself
- How I review and modify Copilot suggestions
- Common corrections I make to AI-generated code

Output everything inside a single Markdown code block so it can be saved directly as qa-coding-style.md
```

---

## Suggested Repository Structure

```
/docs
  /qa-context
      memory-snapshot.md       â† Output of Prompt 1
          project-handoff.md       â† Output of Prompt 2
              qa-coding-style.md       â† Output of Prompt 3
                  qa-copilot-knowledge-extraction.md  â† This file
                  ```

                  ---

                  ## Notes for QA Automation Agents

                  When building agents or sub-agents for QA automation, reference these files as skill/instruction inputs:

                  - Use `qa-coding-style.md` as the **system instruction** or **persona** for any code-generation agent
                  - Use `project-handoff.md` as the **project context** loaded into agent memory at session start
                  - Use `memory-snapshot.md` as a **dynamic context** refreshed periodically via Prompt 1
                  