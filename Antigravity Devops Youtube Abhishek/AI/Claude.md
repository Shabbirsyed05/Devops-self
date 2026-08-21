# 📚 AI Engineering Reference Guide

> **Source:** Video tutorials by *Abhishek Veeramalla*
> **Purpose:** Comprehensive notes for job interviews and daily AI/engineering use.

---

## 📋 Table of Contents

1. [Claude Code — Zero to Hero](#1-claude-code--zero-to-hero)
2. [Claude Code + OpenRouter + DeepSeek V4 Flash](#2-claude-code--openrouter--deepseek-v4-flash)
3. [DeepSeek Harness — Explained Simply](#3-deepseek-harness--explained-simply)
4. [RAG (Retrieval-Augmented Generation) — Zero to Hero](#4-rag-retrieval-augmented-generation--zero-to-hero)

---

<br>

# 1. Claude Code — Zero to Hero

> **Source:** "Claude Code Zero to Hero in 60 Minutes" by Abhishek Veeramalla
> **Purpose:** Simple, memorable notes for job interviews and daily engineering use.

---

### 📌 Table of Contents

- [1.1 AI Assistant vs. AI Agent](#11-ai-assistant-vs-ai-agent)
- [1.2 Installation & Prerequisites](#12-installation--prerequisites)
- [1.3 Essential CLI Shortcuts](#13-essential-cli-shortcuts)
- [1.4 Core Slash Commands](#14-core-slash-commands)
- [1.5 Permission Modes & Security](#15-permission-modes--security)
- [1.6 Context Management — CLAUDE.md](#16-context-management--claudemd)
- [1.7 Event Hooks](#17-event-hooks)
- [1.8 Model Context Protocol (MCP)](#18-model-context-protocol-mcp)
- [1.9 Skills & Plugins](#19-skills--plugins)
- [1.10 Quick Interview Cheat Sheet](#110-quick-interview-cheat-sheet)

---

### 1.1 AI Assistant vs. AI Agent

| Feature | AI Assistant (e.g., ChatGPT) | AI Agent (e.g., Claude Code) |
|---|---|---|
| **What it does** | Answers questions, suggests steps | Executes tasks end-to-end autonomously |
| **Human involvement** | Human must copy-paste & run code | Agent reads, edits, runs, and deploys |
| **Example action** | "Here's how to write a test" | Reads your repo → writes test → runs it → fixes errors |

> **🧠 Simple memory tip:**
> *Assistant = advisor. Agent = doer.*
> An assistant tells you what to cook; an agent goes to the kitchen and cooks it.

---

### 1.2 Installation & Prerequisites

- **Supported OS:** macOS, Linux, or Windows WSL
- **Requires:** Active Claude subscription (Pro or Max plan)
- **Install:** Single `curl` script command in terminal

```bash
# One-liner install (conceptually)
curl -fsSL https://claude.ai/install.sh | sh
```

> **Interview tip:** Claude Code is a **terminal-native** agent — it lives in your CLI, not a browser.

---

### 1.3 Essential CLI Shortcuts

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

### 1.4 Core Slash Commands

Slash commands are typed directly inside the Claude Code terminal.

#### `/model` — Switch AI Models

Choose the right model for the task complexity:

| Model | Best For |
|---|---|
| **Haiku** | Fast, cheap, simple tasks |
| **Sonnet** | Balanced — everyday coding |
| **Opus** | Complex reasoning, architecture |

```
/model sonnet
```

---

#### `/usage` — Monitor Token Consumption

- Shows real-time token usage and plan limits.
- **Why it matters:** Each plan has a token cap — exceeding it slows or stops Claude.

```
/usage
```

---

#### `/init` — Generate CLAUDE.md Memory File

- Scans your entire repository and creates a `CLAUDE.md` file.
- This file teaches Claude about your project structure automatically.

```
/init
```

---

#### `/clear` & `/compact` — Manage Context Window

| Command | What It Does |
|---|---|
| `/clear` | Wipes active conversation memory (fresh start) |
| `/compact` | Summarizes and compresses memory to free up tokens |

> **🧠 Why this matters:** Claude has a limited "memory window." Long sessions fill it up, causing degraded responses. These commands keep it clean.

---

### 1.5 Permission Modes & Security

#### Execution Modes *(cycle with `Shift + Tab`)*

| Mode | Behavior | Best For |
|---|---|---|
| **Manual Mode** | Asks permission for EVERY file edit | Beginners, sensitive codebases |
| **Accept Edits Mode** | Auto-edits files, asks before running commands | Most developers (daily use) |
| **Plan Mode** | Read-only — no changes, only analysis | Architecture reviews, planning |
| **Auto Mode** | Fully autonomous — no prompts | Advanced users, CI/CD pipelines |

---

#### Custom Permissions (`/permissions`)

Define explicit **allow** or **deny** rules for what Claude can do.

```json
// Example: Block dangerous commands and hide secrets
{
  "deny": ["rm -rf", "read .env"]
}
```

---

#### Config Scopes *(Where Settings Are Saved)*

| File | Scope | Git Tracked? |
|---|---|---|
| `.claude/settings.local.json` | Your local machine only | ❌ No (git-ignored) |
| `.claude/settings.json` | Whole project/team | ✅ Yes |
| User-level config | Global across all projects | ✅ Yes |

> **Interview tip:** This is like `.gitignore` logic — local secrets stay local, team rules go in the shared file.

---

### 1.6 Context Management — CLAUDE.md

#### What is CLAUDE.md?

A special Markdown file that acts as **Claude's memory for your project.**
Think of it as an onboarding doc you'd give a new developer — but for the AI.

#### What to Put in CLAUDE.md

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

#### Benefits

- ✅ Reduces token overhead (no need to re-explain project every session)
- ✅ Keeps AI aligned with team standards
- ✅ Works like an enterprise "system prompt" for your codebase

> **🧠 Simple memory tip:** `CLAUDE.md` = the README that Claude actually reads and remembers.

---

### 1.7 Event Hooks

Hooks let you automate actions **before or after** Claude does something.
Like lifecycle hooks in frameworks (React's `useEffect`, K8s admission controllers).

#### Hook Types

| Hook | When It Fires | Common Use Case |
|---|---|---|
| `pre-tool-use` | Before Claude runs any tool | Block dangerous commands, strip API keys |
| `user-prompt-submit` | Before your message is sent to the model | Remove sensitive credentials from prompts |
| `post-tool-use` | After Claude edits a file | Auto-run Prettier/linter to format code |
| `notification` | When Claude finishes a task | Send Slack/email alert |
| `stop` | When Claude stops completely | Log completion, trigger downstream pipelines |

#### Example Flow

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

### 1.8 Model Context Protocol (MCP)

#### What is MCP?

A **standardized protocol** for connecting Claude to external tools and services.

> **Analogy:** MCP is like USB — a universal connector. Instead of each tool needing its own special integration, MCP gives one standard way to plug anything in.

#### What You Can Connect

| Tool | Use Case |
|---|---|
| **GitHub** | Read PRs, create issues, review code |
| **Jira** | Create/update tickets from terminal |
| **Kubernetes** | Deploy, scale, inspect clusters |
| **PostgreSQL** | Query databases directly |

#### Key Commands

```bash
# Add a new MCP tool
claude mcp add github

# List all connected tools
claude mcp list
```

> **Interview tip:** MCP is Anthropic's answer to fragmented tool integrations. Knowing MCP shows you understand AI agent interoperability.

---

### 1.9 Skills & Plugins

#### Skills

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

#### Plugins

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

### 1.10 Quick Interview Cheat Sheet

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

### 🔑 Key Takeaways

1. **Claude Code is an agent, not a chatbot** — it takes action, not just advice.
2. **CLAUDE.md is your project's AI onboarding doc** — always create it with `/init`.
3. **Permission modes control risk** — use Manual for sensitive work, Auto for pipelines.
4. **Hooks = enterprise guardrails** — security, quality, and notifications automated.
5. **MCP = universal tool connector** — one protocol to rule all integrations.
6. **Manage your context window** — use `/compact` and `/clear` to keep Claude sharp.
7. **Skills & Plugins = reusable AI workflows** — codify and share team knowledge.

> *📅 Last updated: August 2026 | Based on: Claude Code Zero to Hero (60 min tutorial)*

---

<br>

# 2. Claude Code + OpenRouter + DeepSeek V4 Flash

> **Complete Setup Guide** | Based on tutorial by *Abhishek Veeramalla*

---

### 📌 What Is This About? (Simple Summary)

> **Think of it like this:**
> Claude Code is a smart AI coding assistant (CLI tool), but using it with premium models like Claude Opus/Sonnet is **very expensive**.
> This guide shows you how to use the **same Claude Code tool** but route it through **OpenRouter** to use **DeepSeek V4 Flash** — a cheap, capable open-source model.

---

### 🧩 Key Concepts (Plain English)

| Term | What It Means |
|---|---|
| **Claude Code** | A command-line AI coding agent made by Anthropic |
| **Frontier Models** | Top-tier expensive AI models (Claude Opus, GPT-4, etc.) |
| **OpenRouter** | A single API gateway that lets you access many AI models |
| **DeepSeek V4 Flash** | A fast, cheap, open-source model with great coding ability |
| **API Key** | A secret password to authenticate and use an AI service |
| **Environment Variable** | A system-level config value your terminal/app reads |

---

### 💡 The Problem & Solution

#### ❌ Problem

- Claude Code's default models (Opus/Sonnet) are **expensive**
- Self-hosting open models needs a **high-end GPU** (costly hardware)
- Heavy daily usage = **$100–$200/month** bill

#### ✅ Solution

- Use **OpenRouter** as a middleman
- Point Claude Code to OpenRouter's API instead of Anthropic's
- Use **DeepSeek V4 Flash** — a powerful open-source model served by OpenRouter

---

### 💰 Cost Comparison

| Model / Plan | Cost |
|---|---|
| Claude Sonnet/Opus (Anthropic) | ~$200/month (heavy usage) |
| **DeepSeek V4 Flash via OpenRouter** | **~$3–$4/month (~₹200–₹300 INR)** |
| DeepSeek Input tokens | $0.14 per 1M tokens |
| DeepSeek Output tokens | $0.28 per 1M tokens |

> 💸 **Savings = ~98%** compared to frontier model plans!

---

### 🌟 Why DeepSeek V4 Flash?

- ✅ **1 Million token context window** — handles huge codebases
- ✅ **Competitive coding performance** — great for daily dev tasks
- ✅ **Very low cost** — fraction of a cent per request
- ✅ Easily swappable via OpenRouter with models like:
  - `Qwen Coder`
  - `GLM`
  - Other open-source models

---

### 🔧 Step-by-Step Setup

#### Step 1 — Install Claude Code CLI

```bash
npm install -g @anthropic-ai/claude-code
```

> Installs Claude Code globally on your machine.

---

#### Step 2 — Set Up OpenRouter

1. Go to [openrouter.ai](https://openrouter.ai)
2. **Create an account**
3. Navigate to **API Keys** → Generate a new key
4. **Add credits** (start with ~$5 — lasts weeks/months!)

---

#### Step 3 — Redirect API Endpoint to OpenRouter

> Instead of hitting Anthropic's server, we point Claude Code to OpenRouter.

```bash
export ANTHROPIC_BASE_URL="https://openrouter.ai/api/v1"
export ANTHROPIC_API_KEY="your-openrouter-api-key-here"
```

> 💡 Add these to your `~/.bashrc` or `~/.zshrc` to make them permanent.

---

#### Step 4 — Map Model Aliases to DeepSeek V4 Flash

> Claude Code internally uses alias names like `haiku` or `sonnet`.
> We remap those aliases to point to DeepSeek V4 Flash.

```bash
export CLAUDE_MODEL_ALIAS_HAIKU="deepseek/deepseek-chat-v4-5"
export CLAUDE_MODEL_ALIAS_SONNET="deepseek/deepseek-chat-v4-5"
```

> ✅ Now whenever Claude Code tries to use Haiku or Sonnet, it uses DeepSeek instead.

---

#### Step 5 — Verify the Connection

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

### 🔁 Quick Recap (Memory Aid)

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

### 📝 Interview & Job Talking Points

1. **"I reduced AI coding assistant costs by ~98% by routing Claude Code through OpenRouter to use DeepSeek V4 Flash instead of expensive frontier models."**

2. **"OpenRouter acts as a unified API gateway — you can swap between open-source models without changing your workflow."**

3. **"Environment variables like `ANTHROPIC_BASE_URL` and model alias overrides allow redirecting API calls without modifying any source code."**

4. **"DeepSeek V4 Flash offers a 1M context window and strong coding performance at a fraction of the cost of closed models."**

5. **"This approach avoids the need for expensive local GPU hardware for self-hosting, while still using open-source models."**

---

### ⚡ TL;DR (30-Second Version)

> Claude Code is an AI coding CLI. By default it's expensive.
> Use **OpenRouter** as an API middleman + **DeepSeek V4 Flash** as the model.
> Cost drops from **~$200/month → ~$3–4/month**.
> Done with just 4 environment variable exports. No code changes needed.

---

### 🔗 Resources

- [Claude Code Docs](https://docs.anthropic.com/en/docs/claude-code)
- [OpenRouter](https://openrouter.ai)
- [DeepSeek Models on OpenRouter](https://openrouter.ai/deepseek)

> *📅 Last Updated: August 2026 | Source: YouTube – Abhishek Veeramalla*

---

<br>

# 3. DeepSeek Harness — Explained Simply

> **Complete Guide for Learning & Interviews** | Based on tutorial by *Abhishek Veeramalla*

---

### 📌 What Is This About? (One-Line Summary)

> **DeepSeek open-sourced the "engine room" behind their AI coding agent — the part that tells the LLM *how* to think, what to remember, and what tools to use. That hidden layer is called the Harness.**

---

### 🧩 Key Concepts (Plain English Glossary)

| Term | What It Means |
|---|---|
| **LLM** | The AI brain (e.g., DeepSeek, Claude, GPT) that generates responses |
| **Harness** | The wrapper around the LLM — manages prompts, memory, tools & context |
| **System Prompt** | Hidden instructions sent to the LLM before your message |
| **Agent** | An LLM + Harness working together to complete multi-step tasks |
| **Plugin** | A modular, swappable component that adds a feature to the harness |
| **Cordis Framework** | The plugin system DeepSeek Harness is built on |
| **Trajectory Tab** | A live log showing exactly what the agent is thinking and doing |
| **Open Source** | Publicly available code anyone can read, modify, and learn from |

---

### 💡 The Core Insight — What Is a Harness?

```
WITHOUT a Harness:
  You → LLM → Response

WITH a Harness (Agent):
  You
   ↓
[Harness Layer]
  ├── Builds the system prompt
  ├── Reads your codebase/repo context
  ├── Manages memory across turns
  ├── Handles tool calls (file read/write, search, etc.)
  └── Dispatches polished request
         ↓
       LLM
         ↓
   Smart Response
```

> 🔑 **Key Insight:** Two agents using the *same LLM* can perform very differently — the harness is what separates a great coding agent from a mediocre one.

---

### 🚨 The Problem with Existing Agents

| Agent | Harness Visibility |
|---|---|
| Claude Code (Anthropic) | ❌ Closed source — black box |
| GitHub Copilot | ❌ Closed source — black box |
| **DeepSeek Harness** | ✅ **Fully open source — inspect everything** |

> Most companies treat their harness as a **trade secret**. DeepSeek made theirs fully public.

---

### 🌟 Why DeepSeek Harness Went Viral

#### 1. 💯 100% Open Source

- You can see *exactly* how prompts are built
- Inspect how memory is stored and retrieved
- Understand how repo context is injected into LLM calls

#### 2. 🔭 Full Observability — The Trajectory Tab

- Every agent action has a **live trace log**
- Shows:
  - The injected system prompt
  - Workspace/repo context loaded
  - Tool calls made
  - Internal reasoning steps
- Makes debugging agent behavior straightforward

#### 3. 🔌 Modular Plugin Architecture

- Built on the **Cordis Framework**
- Contains **160+ plugins**
- Each feature = one plugin = independently swappable
- Examples of plugins:
  - Memory manager
  - File reader
  - Terminal executor
  - UI widget renderer
  - Token cost tracker

---

### 🔧 Installation & Setup

#### Step 1 — Install via npm/npx

```bash
npx dsh-web
```

Or install globally:

```bash
npm install -g dsh
dsh web
```

---

#### Step 2 — Launch the Web UI

```
Local URL: http://localhost:3080
```

> Opens a browser-based chat interface connected to the harness engine.

---

#### Step 3 — Configure Your Model

1. Go to ⚙️ **Settings** in the UI
2. Add your API key for:
   - DeepSeek
   - OpenAI
   - Or any supported provider
3. Switch between models freely — no code changes needed

---

### 🔌 Plugin Architecture Deep Dive

#### How Plugins Work

```
Cordis Framework (Core)
   ├── Plugin: memory-manager
   ├── Plugin: system-prompt-builder
   ├── Plugin: repo-context-reader
   ├── Plugin: tool-executor
   ├── Plugin: token-cost-tracker
   ├── Plugin: slack-notifier        ← custom (you can build this)
   ├── Plugin: email-dispatcher      ← custom (you can build this)
   └── Plugin: ui-widget             ← custom (you can build this)
```

#### Disabling or Tweaking Existing Plugins

Edit the CLI config file in your profile directory:

```yaml
# cordis.patch.yaml

plugins:
  memory-manager:
    enabled: false       # disable a plugin
  system-prompt-builder:
    maxTokens: 2000      # tweak a plugin's settings
```

---

### 🛠️ Building Your Own Custom Plugins

#### Switch to Creator Mode

1. Open DeepSeek Harness UI
2. Under **Agent Presets** → select **Creator Mode**
3. Now prompt the agent to build a plugin for you!

#### Example Prompts

```
"Add a Slack notification plugin that alerts me when a task completes."
```

```
"Build a token cost tracker that shows total spend per session."
```

```
"Add an email dispatcher plugin triggered after each agent run."
```

> 🤯 The agent **writes its own plugin code**, shows you a diff, and you **approve the change in real time** — no manual coding needed.

#### Plugin Build Workflow

```
You describe the feature
       ↓
Agent generates plugin code
       ↓
You review the diff in UI
       ↓
Approve → Plugin is live instantly
```

---

### 🔍 Trajectory Tab — Full Observability

> This is what makes DeepSeek Harness unique for **learning and debugging**:

For every agent action, you can see:

```
Trajectory Log Example:
─────────────────────────────
[SYSTEM PROMPT INJECTED]
  → "You are an expert software engineer..."
  → "Current workspace: /my-project"
  → "Available tools: read_file, write_file, run_command..."

[CONTEXT LOADED]
  → Repo structure scanned: 42 files
  → Relevant files injected: main.py, utils.py

[TOOL CALL]
  → read_file("src/api.py")

[LLM RESPONSE]
  → "I see the bug is on line 34..."

[TOOL CALL]
  → write_file("src/api.py", fixed_content)
─────────────────────────────
```

> 🎯 Perfect for **understanding how agents actually work** — not just using them as a black box.

---

### 📊 Quick Comparison Table

| Feature | Claude Code / Copilot | DeepSeek Harness |
|---|---|---|
| Open Source | ❌ | ✅ |
| View System Prompts | ❌ | ✅ |
| Trajectory Logging | ❌ | ✅ |
| Custom Plugins | ❌ | ✅ |
| Plugin Count | Unknown | 160+ |
| Self-hostable | ❌ | ✅ |
| Model-agnostic | Partial | ✅ |

---

### 📝 Interview & Job Talking Points

1. **"A Harness is the orchestration layer around an LLM — it handles system prompts, memory, repo context, and tool use. The LLM itself is just one component."**

2. **"DeepSeek Harness went viral because it's the first major open-source agent harness — you can inspect every prompt, tool call, and memory injection through the Trajectory Tab."**

3. **"It's built on the Cordis plugin framework with 160+ modular components. You can disable, tweak, or add plugins using YAML config or by prompting the agent itself in Creator Mode."**

4. **"The Trajectory Tab provides full agent observability — showing injected system prompts, loaded context, and tool call sequences — which makes debugging and auditing agent behavior straightforward."**

5. **"Two agents using the same LLM can behave very differently depending on their harness. This is why harness engineering is a critical emerging skill in AI/ML engineering roles."**

---

### ⚡ TL;DR (30-Second Version)

> An AI agent = **LLM + Harness**.
> The Harness is the hidden layer that builds prompts, manages memory, reads code context, and calls tools.
> Most agents keep this layer secret.
> **DeepSeek open-sourced theirs** — 160+ plugins, full trace logs, and you can build new plugins just by asking the agent.
> This is a **big deal** for transparency, learning, and customization.

---

### 🧭 Mental Model for Long-Term Memory

```
Think of an AI agent like a chef in a restaurant:

  LLM        = The Chef (cooking skill)
  Harness    = The Kitchen Setup
                ├── Recipe book (system prompt)
                ├── Pantry inventory (repo context)
                ├── Utensils (tools)
                └── Order tracker (memory)

DeepSeek = The only restaurant that lets you
           walk into the kitchen and see everything.
```

---

### 🔗 Resources

- [DeepSeek Harness GitHub](https://github.com/deepseek-ai/DeepSeek-Harness)
- [Cordis Framework](https://github.com/cordiverse/cordis)
- [DeepSeek Models](https://platform.deepseek.com)

> *📅 Last Updated: August 2026 | Source: YouTube – Abhishek Veeramalla*

---

<br>

# 4. RAG (Retrieval-Augmented Generation) — Zero to Hero

> **Complete Guide for Learning & Interviews** | Based on tutorial by *Abhishek Veeramalla*

---

### 📌 What Is This About? (One-Line Summary)

> **RAG lets an LLM answer questions using YOUR private documents — without retraining the model.
> It works by searching relevant document chunks and injecting them into the prompt before asking the LLM.**

---

### 🧩 Key Concepts (Plain English Glossary)

| Term | What It Means |
|---|---|
| **LLM** | AI model (e.g., GPT-4, DeepSeek) that generates text responses |
| **RAG** | A technique to give LLMs access to private/real-time data via retrieval |
| **Embedding** | A list of numbers (vector) that represents the *meaning* of a text |
| **Vector** | A point in multi-dimensional space; similar meanings = nearby points |
| **Cosine Similarity** | A score (0–1) measuring how semantically close two vectors are |
| **Chunking** | Breaking large documents into smaller pieces before embedding |
| **Vector Database** | A database that stores and searches embeddings (e.g., ChromaDB) |
| **Collection** | A group of stored embeddings in a vector DB (like a table in SQL) |
| **Augmented Prompt** | A prompt = retrieved context + system instructions + user query |
| **Hallucination** | When an LLM confidently makes up an incorrect answer |

---

### 💡 Why RAG Exists — The Problem

#### ❌ Problem 1: LLMs Have a Training Cutoff

> LLMs only know what they were trained on.
> They have no knowledge of events, policies, or documents created after their cutoff date.

#### ❌ Problem 2: LLMs Don't Know Your Private Data

> Your company's leave policies, runbooks, Confluence pages, SharePoint docs —
> an LLM has never seen any of it.

#### ❌ Problem 3: You Can't Just Paste Everything

> You *could* manually paste a document into a prompt — but what if your org has
> **10,000 internal documents**? That's impossible to manage and too expensive.

---

### ✅ The RAG Solution — 3 Simple Stages

```
User Query
    ↓
┌──────────────────────────────────────┐
│  R → RETRIEVE                        │
│      Search vector DB for relevant   │
│      document chunks                 │
├──────────────────────────────────────┤
│  A → AUGMENT                         │
│      Merge retrieved chunks +        │
│      system prompt + user query      │
├──────────────────────────────────────┤
│  G → GENERATE                        │
│      Send augmented prompt to LLM    │
│      to get a grounded answer        │
└──────────────────────────────────────┘
    ↓
Accurate, hallucination-free Answer
```

---

### 🔬 Core Concept 1 — Embeddings & Semantic Search

#### Why Keyword Search Fails

```
Document says:  "leave policy"
User asks:      "How many annual day offs do I get?"

❌ Keyword Search → No match (different words)
✅ Semantic Search → Match! (same meaning)
```

#### What an Embedding Looks Like

```python
# The word "leave policy" becomes a vector like:
[0.023, -0.145, 0.867, 0.334, ..., 0.091]
# ← 1536 numbers representing its meaning →
```

> Model used: `text-embedding-3-small` (OpenAI) — lightweight & fast

#### How Cosine Similarity Works

```
"leave policy"   →  Vector A  ────────┐
"annual day off" →  Vector B  ────────┤→ Cosine Score: 0.94 ✅ (very similar!)

"banana recipe"  →  Vector C  ────────┘→ Cosine Score: 0.12 ❌ (not related)
```

> Score **closer to 1.0** = semantically similar | Score **closer to 0.0** = unrelated

---

### 🔬 Core Concept 2 — Chunking

#### Why Chunking Is Necessary

> Embedding an entire 50-page document into ONE vector loses precision.
> The vector becomes a "blurry average" of everything.

#### How Chunking Works

```
Original Document (2000 tokens)
         ↓
┌────────────────────────────────────┐
│ Chunk 1: tokens 1–100             │ → Embedding 1
│ Chunk 2: tokens 101–200           │ → Embedding 2
│ Chunk 3: tokens 201–300           │ → Embedding 3
│ ...                               │ → ...
└────────────────────────────────────┘
```

> **Typical chunk size:** 100–500 tokens
> **Rule of thumb:** Smaller chunks = more precise retrieval

---

### 🔬 Core Concept 3 — Vector Databases (ChromaDB)

#### What a Vector Database Does

```
Relational DB (SQL)        Vector DB (ChromaDB)
────────────────────        ────────────────────
Table      →               Collection
Row        →               Document + its Embedding
WHERE      →               Cosine Similarity Search
```

#### ChromaDB at a Glance

- Runs **locally** (via Docker or embedded Python)
- Stores **text chunks + their embeddings** together
- Returns **top-N most similar chunks** for any query
- Free and open source ✅

---

### 🚀 Full RAG Pipeline — Step by Step

#### Step 1 — Prepare & Chunk Documents

```python
# Read your internal documents
documents = load_files(["leave_policy.txt", "runbook.txt"])

# Split into chunks
chunks = chunk_text(documents, chunk_size=200, overlap=20)
# Result: ["Employees get 12 days of...", "Sick leave policy states...", ...]
```

---

#### Step 2 — Embed & Store in ChromaDB

```python
# Generate embeddings for each chunk
embeddings = embedding_model.encode(chunks)
# Each chunk → [0.023, -0.145, ..., 0.091]

# Store in ChromaDB collection
collection = chromadb.create_collection("company_docs")
collection.add(documents=chunks, embeddings=embeddings)
```

---

#### Step 3 — Query Retrieval

```python
user_query = "How many sick leaves do I have?"

# Convert query to embedding
query_embedding = embedding_model.encode(user_query)

# Search ChromaDB for top 3 similar chunks
results = collection.query(
    query_embeddings=[query_embedding],
    n_results=3
)
# Returns: ["Sick leave policy states 10 days per year...", ...]
```

---

#### Step 4 — Build Augmented Prompt

```python
system_prompt = """
You are a helpful HR assistant.
Answer ONLY using the provided context.
Do NOT hallucinate or make up information.
"""

augmented_prompt = f"""
{system_prompt}

Context:
{results[0]}
{results[1]}
{results[2]}

User Question: {user_query}
"""
```

---

#### Step 5 — Generate the Answer

```python
response = openai.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": augmented_prompt}]
)

print(response.choices[0].message.content)
# → "You are entitled to 10 sick leaves per year as per the company policy."
```

---

### 🏗️ Full Architecture Diagram

```
┌──────────────────────────────────────────────────────────┐
│                    INDEXING PHASE (One-time)             │
│                                                          │
│  Raw Docs → Chunking → Embedding Model → ChromaDB        │
│  (PDFs, TXTs, Confluence pages...)                       │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│                    QUERY PHASE (Per request)             │
│                                                          │
│  User Query                                              │
│      ↓                                                   │
│  Embedding Model (query → vector)                        │
│      ↓                                                   │
│  ChromaDB (cosine similarity search → top chunks)        │
│      ↓                                                   │
│  Augmented Prompt (chunks + system prompt + query)       │
│      ↓                                                   │
│  LLM (gpt-4o-mini / DeepSeek)                           │
│      ↓                                                   │
│  ✅ Grounded, accurate answer                            │
└──────────────────────────────────────────────────────────┘
```

---

### 🛠️ Prerequisites & Setup

| Requirement | Purpose |
|---|---|
| **Python** | Run notebooks and pipeline scripts |
| **Docker** | Run ChromaDB or local Ollama |
| **OpenAI API Key** | Embedding model + completion model |
| **Jupyter Notebooks** | Interactive step-by-step execution |
| **ChromaDB** | Local vector database |

```bash
pip install chromadb openai langchain tiktoken
docker run -p 8000:8000 chromadb/chroma
```

---

### 📝 Interview & Job Talking Points

1. **"RAG solves two core LLM limitations: training cutoff dates and lack of access to private organizational data — without requiring model retraining."**

2. **"Keyword search fails for semantic queries. Embeddings convert text into high-dimensional vectors where meaning is encoded as position in space. Cosine similarity then finds the closest matching chunks."**

3. **"We chunk documents before embedding because a single vector for a large document becomes a diluted average of all its content, reducing retrieval precision."**

4. **"ChromaDB is a vector database that stores chunks with their embeddings in collections. At query time, the user's query is embedded and a cosine similarity search returns the top-N relevant chunks."**

5. **"The augmented prompt combines: (1) a system prompt preventing hallucination, (2) retrieved context chunks, and (3) the user's question — before sending to the LLM."**

6. **"RAG is more cost-effective than fine-tuning for knowledge updates. Fine-tuning retrains the model weights; RAG just updates the vector database."**

---

### 📊 RAG vs Alternatives

| Approach | Pros | Cons |
|---|---|---|
| **Paste doc in prompt** | Simple, fast | Fails at scale (1000s of docs) |
| **Fine-tuning** | Model "learns" the data | Expensive, slow, hard to update |
| **RAG** ✅ | Scalable, updatable, cheap | Requires vector DB setup |

---

### ⚡ TL;DR (30-Second Version)

> LLMs don't know your private data.
> RAG fixes this: **chunk** your docs → **embed** them → **store** in a vector DB.
> At query time: **embed** the question → **search** the DB → **inject** top results into the prompt → LLM gives a **grounded answer**.
> The magic glue is **embeddings** (meaning as numbers) + **cosine similarity** (finding close meanings).

---

### 🧭 Chef Analogy for Memory

```
Think of RAG like a chef preparing a dish:

  LLM          = The Chef (can cook, but only from memory)
  Your Docs    = The Recipe Book (private knowledge)
  Chunking     = Cutting the book into individual recipe cards
  Embeddings   = Tagging each card with flavor profile numbers
  Vector DB    = The organized recipe card cabinet
  Retrieval    = Finding the 3 most relevant recipe cards
  Augmentation = Handing those cards + instructions to the Chef
  Generation   = Chef cooks the perfect dish using the cards
```

---

### 🔗 Resources

- [ChromaDB Docs](https://docs.trychroma.com)
- [OpenAI Embeddings](https://platform.openai.com/docs/guides/embeddings)
- [LangChain RAG Guide](https://python.langchain.com/docs/tutorials/rag/)
- [Abhishek Veeramalla GitHub](https://github.com/iam-veeramalla)

> *📅 Last Updated: August 2026 | Source: YouTube – Abhishek Veeramalla*

---

<br>

*📅 Guide last compiled: August 2026*
