# Claude Code Zero to Hero in 60 Minutes

# 🤖 Claude Code — Zero to Hero: Interview & Job Reference Guide

> **Source:** "Claude Code Zero to Hero in 60 Minutes" by Abhishek Veeramalla  
> **Purpose:** Simple, memorable notes for job interviews and daily engineering use.

---

## 📌 Table of Contents

1. [AI Assistant vs. AI Agent](#1-ai-assistant-vs-ai-agent)
2. [Installation & Prerequisites](#2-installation--prerequisites)
3. [Essential CLI Shortcuts](#3-essential-cli-shortcuts)
4. [Core Slash Commands](#4-core-slash-commands)
5. [Permission Modes & Security](#5-permission-modes--security)
6. [Context Management — CLAUDE.md](#6-context-management--claudemd)
7. [Event Hooks](#7-event-hooks)
8. [Model Context Protocol (MCP)](#8-model-context-protocol-mcp)
9. [Skills & Plugins](#9-skills--plugins)
10. [Quick Interview Cheat Sheet](#quick-interview-cheat-sheet)

---

## 1. AI Assistant vs. AI Agent

| Feature | AI Assistant (e.g., ChatGPT) | AI Agent (e.g., Claude Code) |
|---|---|---|
| **What it does** | Answers questions, suggests steps | Executes tasks end-to-end autonomously |
| **Human involvement** | Human must copy-paste & run code | Agent reads, edits, runs, and deploys |
| **Example action** | "Here's how to write a test" | Reads your repo → writes test → runs it → fixes errors |

> **🧠 Simple memory tip:**  
> *Assistant = advisor. Agent = doer.*  
> An assistant tells you what to cook; an agent goes to the kitchen and cooks it.

---

## 2. Installation & Prerequisites

- **Supported OS:** macOS, Linux, or Windows WSL
- **Requires:** Active Claude subscription (Pro or Max plan)
- **Install:** Single `curl` script command in terminal

```bash
# One-liner install (conceptually)
curl -fsSL https://claude.ai/install.sh | sh
```

> **Interview tip:** Claude Code is a **terminal-native** agent — it lives in your CLI, not a browser.

---

## 3. Essential CLI Shortcuts

| Shortcut | What It Does | When to Use |
|---|---|---|
| `Esc` (once) | **Stop/interrupt** current action | Claude is doing something wrong mid-way |
| `Esc` (twice) | **Open history** — restore earlier code/context | Rolled too far, need to go back |
| `Ctrl + C` (twice) | **Exit** Claude Code session | Done working, close the session |
| `claude --resume` | **Reconnect** to a previous session | Accidentally closed terminal, pick up where you left off |
| `Shift + Tab` | **Cycle permission modes** | Switch between manual/auto execution modes |

> **🧠 Simple memory tip:**  
> *One Esc = pause. Two Esc = time machine. Two Ctrl+C = goodbye.*

---

## 4. Core Slash Commands

Slash commands are typed directly inside the Claude Code terminal.

### `/model` — Switch AI Models
- Choose the right model for the task complexity:

  | Model | Best For |
  |---|---|
  | **Haiku** | Fast, cheap, simple tasks |
  | **Sonnet** | Balanced — everyday coding |
  | **Opus** | Complex reasoning, architecture |

```
/model sonnet
```

---

### `/usage` — Monitor Token Consumption
- Shows real-time token usage and plan limits.
- **Why it matters:** Each plan has a token cap — exceeding it slows or stops Claude.

```
/usage
```

---

### `/init` — Generate CLAUDE.md Memory File
- Scans your entire repository and creates a `CLAUDE.md` file.
- This file teaches Claude about your project structure automatically.

```
/init
```

---

### `/clear` & `/compact` — Manage Context Window

| Command | What It Does |
|---|---|
| `/clear` | Wipes active conversation memory (fresh start) |
| `/compact` | Summarizes and compresses memory to free up tokens |

> **🧠 Why this matters:** Claude has a limited "memory window." Long sessions fill it up, causing degraded responses. These commands keep it clean.

---

## 5. Permission Modes & Security

### Execution Modes (cycle with `Shift + Tab`)

| Mode | Behavior | Best For |
|---|---|---|
| **Manual Mode** | Asks permission for EVERY file edit | Beginners, sensitive codebases |
| **Accept Edits Mode** | Auto-edits files, asks before running commands | Most developers (daily use) |
| **Plan Mode** | Read-only — no changes, only analysis | Architecture reviews, planning |
| **Auto Mode** | Fully autonomous — no prompts | Advanced users, CI/CD pipelines |

---

### Custom Permissions (`/permissions`)

Define explicit **allow** or **deny** rules for what Claude can do.

```json
// Example: Block dangerous commands and hide secrets
{
  "deny": ["rm -rf", "read .env"]
}
```

---

### Config Scopes (Where Settings Are Saved)

| File | Scope | Git Tracked? |
|---|---|---|
| `.claude/settings.local.json` | Your local machine only | ❌ No (git-ignored) |
| `.claude/settings.json` | Whole project/team | ✅ Yes |
| User-level config | Global across all projects | ✅ Yes |

> **Interview tip:** This is like `.gitignore` logic — local secrets stay local, team rules go in the shared file.

---

## 6. Context Management — CLAUDE.md

### What is CLAUDE.md?
A special Markdown file that acts as **Claude's memory for your project.**

Think of it as an onboarding doc you'd give a new developer — but for the AI.

### What to Put in CLAUDE.md

```markdown
# Project: My App

## Architecture
- Frontend: React (src/components/)
- Backend: Node.js (src/api/)
- DB: PostgreSQL

## Build & Test Commands
- `npm run dev` — Start dev server
- `npm test` — Run unit tests

## Coding Style
- Use camelCase for variables
- Always write JSDoc for public functions
- No console.log in production code
```

### Benefits

- ✅ Reduces token overhead (no need to re-explain project every session)
- ✅ Keeps AI aligned with team standards
- ✅ Works like an enterprise "system prompt" for your codebase

> **🧠 Simple memory tip:** `CLAUDE.md` = the README that Claude actually reads and remembers.

---

## 7. Event Hooks

Hooks let you automate actions **before or after** Claude does something. Like lifecycle hooks in frameworks (React's `useEffect`, K8s admission controllers).

### Hook Types

| Hook | When It Fires | Common Use Case |
|---|---|---|
| `pre-tool-use` | Before Claude runs any tool | Block dangerous commands, strip API keys |
| `user-prompt-submit` | Before your message is sent to the model | Remove sensitive credentials from prompts |
| `post-tool-use` | After Claude edits a file | Auto-run Prettier/linter to format code |
| `notification` | When Claude finishes a task | Send Slack/email alert |
| `stop` | When Claude stops completely | Log completion, trigger downstream pipelines |

### Example Flow

```
You type a prompt
     ↓
[user-prompt-submit hook] → Strip secrets from prompt
     ↓
Claude runs and edits files
     ↓
[post-tool-use hook] → Run Prettier on changed files
     ↓
Claude finishes
     ↓
[notification hook] → Send Slack message "Task done!"
```

> **Interview tip:** Hooks make Claude Code **enterprise-ready** — you can enforce security policies and quality gates without changing developer workflows.

---

## 8. Model Context Protocol (MCP)

### What is MCP?
A **standardized protocol** for connecting Claude to external tools and services.

> **Analogy:** MCP is like USB — a universal connector. Instead of each tool needing its own special integration, MCP gives one standard way to plug anything in.

### What You Can Connect

| Tool | Use Case |
|---|---|
| **GitHub** | Read PRs, create issues, review code |
| **Jira** | Create/update tickets from terminal |
| **Kubernetes** | Deploy, scale, inspect clusters |
| **PostgreSQL** | Query databases directly |

### Key Commands

```bash
# Add a new MCP tool
claude mcp add github

# List all connected tools
claude mcp list
```

> **Interview tip:** MCP is Anthropic's answer to fragmented tool integrations. Knowing MCP shows you understand AI agent interoperability.

---

## 9. Skills & Plugins

### Skills
- Markdown files (`skills.md`) stored in `.claude/skills/`
- Teach Claude **how to perform complex, repeatable tasks**
- Like a recipe card — Claude reads it and follows instructions

```
.claude/
  skills/
    deploy-kubernetes.md   ← "Here's how we deploy to our K8s cluster"
    run-migrations.md      ← "Here's our DB migration process"
```

**Example skill instruction:**
```markdown
# Kubernetes Deployment Skill

1. Always run `kubectl get nodes` first to verify cluster health
2. Use `helm upgrade --install` for deployments
3. After deploy, run smoke tests from `tests/smoke/`
```

---

### Plugins
- A **bundled package** that combines:
  - MCP servers
  - Hooks
  - Skills
  - Sub-agents
- Installed with `/plugin` command
- Like an npm package — one install, everything works

```
/plugin install my-company-devops-plugin
```

> **🧠 Simple memory tip:**  
> *Skills = instructions for one task. Plugins = a full toolkit bundled together.*

---

## ⚡ Quick Interview Cheat Sheet

| Question | Key Answer |
|---|---|
| **What is Claude Code?** | Terminal-native AI agent by Anthropic that autonomously reads, edits, runs, and deploys code |
| **Agent vs. Assistant?** | Agent = executes end-to-end. Assistant = advises, human executes |
| **What is CLAUDE.md?** | Project memory file — stores architecture, commands, coding standards for the AI |
| **What are permission modes?** | Manual, Accept Edits, Plan, Auto — control how autonomously Claude operates |
| **What are Hooks?** | Lifecycle triggers (pre/post) to enforce security, formatting, and notifications |
| **What is MCP?** | Standard protocol to connect Claude to tools like GitHub, Jira, K8s, databases |
| **Skills vs. Plugins?** | Skills = task instructions in Markdown. Plugins = full bundled integrations |
| **How to manage token limits?** | Use `/compact` to compress memory, `/clear` to reset, `/usage` to monitor |
| **How to secure Claude Code?** | Custom `/permissions` rules + `pre-tool-use` hooks to block secrets/dangerous commands |
| **Config scope priority?** | Local JSON (personal) → Project JSON (team) → User-level (global) |

---

## 🔑 Key Takeaways (The 20% That Gets You 80%)

1. **Claude Code is an agent, not a chatbot** — it takes action, not just advice.
2. **CLAUDE.md is your project's AI onboarding doc** — always create it with `/init`.
3. **Permission modes control risk** — use Manual for sensitive work, Auto for pipelines.
4. **Hooks = enterprise guardrails** — security, quality, and notifications automated.
5. **MCP = universal tool connector** — one protocol to rule all integrations.
6. **Manage your context window** — use `/compact` and `/clear` to keep Claude sharp.
7. **Skills & Plugins = reusable AI workflows** — codify and share team knowledge.

---

*📅 Last updated: August 2026 | Based on: Claude Code Zero to Hero (60 min tutorial)*

================================================================
