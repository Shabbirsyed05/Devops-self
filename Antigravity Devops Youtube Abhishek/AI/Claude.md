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

 # Claude Code + OpenRouter + DeepSeek V4 Flash


# 🤖 Claude Code + OpenRouter + DeepSeek V4 Flash
> **Complete Setup Guide** | Based on tutorial by *Abhishek Veeramalla*

---

## 📌 What Is This About? (Simple Summary)

> **Think of it like this:**
> Claude Code is a smart AI coding assistant (CLI tool), but using it with premium models like Claude Opus/Sonnet is **very expensive**.
> This guide shows you how to use the **same Claude Code tool** but route it through **OpenRouter** to use **DeepSeek V4 Flash** — a cheap, capable open-source model.

---

## 🧩 Key Concepts (Plain English)

| Term | What It Means |
|---|---|
| **Claude Code** | A command-line AI coding agent made by Anthropic |
| **Frontier Models** | Top-tier expensive AI models (Claude Opus, GPT-4, etc.) |
| **OpenRouter** | A single API gateway that lets you access many AI models |
| **DeepSeek V4 Flash** | A fast, cheap, open-source model with great coding ability |
| **API Key** | A secret password to authenticate and use an AI service |
| **Environment Variable** | A system-level config value your terminal/app reads |

---

## 💡 The Problem & Solution

### ❌ Problem
- Claude Code's default models (Opus/Sonnet) are **expensive**
- Self-hosting open models needs a **high-end GPU** (costly hardware)
- Heavy daily usage = **$100–$200/month** bill

### ✅ Solution
- Use **OpenRouter** as a middleman
- Point Claude Code to OpenRouter's API instead of Anthropic's
- Use **DeepSeek V4 Flash** — a powerful open-source model served by OpenRouter

---

## 💰 Cost Comparison

| Model / Plan | Cost |
|---|---|
| Claude Sonnet/Opus (Anthropic) | ~$200/month (heavy usage) |
| **DeepSeek V4 Flash via OpenRouter** | **~$3–$4/month (~₹200–₹300 INR)** |
| DeepSeek Input tokens | $0.14 per 1M tokens |
| DeepSeek Output tokens | $0.28 per 1M tokens |

> 💸 **Savings = ~98%** compared to frontier model plans!

---

## 🌟 Why DeepSeek V4 Flash?

- ✅ **1 Million token context window** — handles huge codebases
- ✅ **Competitive coding performance** — great for daily dev tasks
- ✅ **Very low cost** — fraction of a cent per request
- ✅ Easily swappable via OpenRouter with models like:
  - `Qwen Coder`
  - `GLM`
  - Other open-source models

---

## 🔧 Step-by-Step Setup

### Step 1 — Install Claude Code CLI

```bash
npm install -g @anthropic-ai/claude-code
```

> Installs Claude Code globally on your machine.

---

### Step 2 — Set Up OpenRouter

1. Go to [openrouter.ai](https://openrouter.ai)
2. **Create an account**
3. Navigate to **API Keys** → Generate a new key
4. **Add credits** (start with ~$5 — lasts weeks/months!)

---

### Step 3 — Redirect API Endpoint to OpenRouter

> Instead of hitting Anthropic's server, we point Claude Code to OpenRouter.

```bash
export ANTHROPIC_BASE_URL="https://openrouter.ai/api/v1"
export ANTHROPIC_API_KEY="your-openrouter-api-key-here"
```

> 💡 Add these to your `~/.bashrc` or `~/.zshrc` to make them permanent.

---

### Step 4 — Map Model Aliases to DeepSeek V4 Flash

> Claude Code internally uses alias names like `haiku` or `sonnet`.
> We remap those aliases to point to DeepSeek V4 Flash.

```bash
export CLAUDE_MODEL_ALIAS_HAIKU="deepseek/deepseek-chat-v4-5"
export CLAUDE_MODEL_ALIAS_SONNET="deepseek/deepseek-chat-v4-5"
```

> ✅ Now whenever Claude Code tries to use Haiku or Sonnet, it uses DeepSeek instead.

---

### Step 5 — Verify the Connection

```bash
claude
```

Inside Claude Code CLI:

```
/model
```

- Select the mapped DeepSeek model
- Ask it: *"Which model are you running on?"*
- It should confirm: **DeepSeek V4 Flash** ✅

---

## 🔁 Quick Recap (Memory Aid)

```
Claude Code CLI
     ↓
(Instead of Anthropic API)
     ↓
OpenRouter API Gateway
     ↓
DeepSeek V4 Flash (cheap + powerful)
```

---

## 📝 Interview & Job Talking Points

> Use these to explain the concept clearly in interviews or discussions:

1. **"I reduced AI coding assistant costs by ~98% by routing Claude Code through OpenRouter to use DeepSeek V4 Flash instead of expensive frontier models."**

2. **"OpenRouter acts as a unified API gateway — you can swap between open-source models without changing your workflow."**

3. **"Environment variables like `ANTHROPIC_BASE_URL` and model alias overrides allow redirecting API calls without modifying any source code."**

4. **"DeepSeek V4 Flash offers a 1M context window and strong coding performance at a fraction of the cost of closed models."**

5. **"This approach avoids the need for expensive local GPU hardware for self-hosting, while still using open-source models."**

---

## ⚡ TL;DR (30-Second Version)

> Claude Code is an AI coding CLI. By default it's expensive.
> Use **OpenRouter** as an API middleman + **DeepSeek V4 Flash** as the model.
> Cost drops from **~$200/month → ~$3–4/month**.
> Done with just 4 environment variable exports. No code changes needed.

---

## 🔗 Resources

- [Claude Code Docs](https://docs.anthropic.com/en/docs/claude-code)
- [OpenRouter](https://openrouter.ai)
- [DeepSeek Models on OpenRouter](https://openrouter.ai/deepseek)

---

*📅 Last Updated: August 2026 | Source: YouTube – Abhishek Veeramalla*

