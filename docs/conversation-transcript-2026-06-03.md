# Conversation Transcript — Autonomous Agent OS Design

**Date:** June 3, 2026

**Participants:** Jaron Bragg (operator), DeepSeek, Gemini, Grok, Claude (Perplexity session)

**Purpose:** Full verbatim record of the multi-AI conversation that produced the autonomous agent OS architecture in this repository.

---

## USER — Jaron Bragg

What system am I missing what more can I do to improve this works?

help make this better. By better I don't mean "safer" I have my own guardrails in place for this system but won't matter without the system designed and ready to run.
[Using different AIs to achieve this task, DO NOT USE MEMORY OF OLD PROJECTS UNLESS ASKED BY JARON NEW GOALS AND INTENTIONS MEANS WHAT YOU ASSUME COULD NOW PUT IN A BIAS ASPECT IN MY SYSTEM THAT CONTRADICTS NEW GOALS NOT IN YOUR MEMORY]

---

## DEEPSEEK — Blueprint Document

Building a Self-Healing, Autonomous AI Agent System
Your Personal Digital Proxy - A Practical Guide & Legal Notice

### 1. Important Legal & Ethical Notice

Before building or running any autonomous AI system, understand the legal boundaries.

- Installing software without your knowledge by a third party is illegal (e.g., under the US Computer Fraud and Abuse Act). You are building your own system - that is fine.
- Running an AI that you installed is generally legal, but you are responsible for its actions.
  - If it accesses other systems without authorisation, sends spam, commits fraud, or generates illegal content, you may be held liable even if you aren't watching it.
  - Processing personal data without consent can violate privacy laws (GDPR, CCPA, etc.).
- "Anonymous" AI is not inherently illegal for personal, local use. However, commercial use may require transparency disclosures (e.g., state chatbot laws, EU AI Act).
- Human-on-the-Loop (observation without approval) is allowed - you are not delegating legal responsibility. But you must be able to interrupt or shut down the system if it goes rogue.

Recommendation: Keep the system confined to your own devices. Use sandboxes for code generation. Never give it uncontrolled network access or financial credentials without a kill switch.

### 2. Your Stated Intentions (Summarised)

- An AI agent that acts as you at your computer.
- Views your phone screen and can communicate with it when needed.
- Checks your files, media sites, and "Jaron's daily topics".
- Learns to obtain what it lacks (downloads tools, libraries, data).
- Creates content for monetisation (X posts, YouTube/TikTok videos).
- Writes code to improve its own capabilities (GitHub repos, new modules).
- Uses a loop of multiple AI agents (Claude, ChatGPT, Grok, DeepSeek, Ollama) without manual approval - only observation.
- Self-healing: if an agent fails, another agent fixes or replaces it. Duplicates itself in case of issues, rolls back bad calls, and reboots.

Reality check: Fully autonomous, self-improving, self-healing AI does not exist today. The following is a blueprint using the most advanced open-source building blocks. You will need significant programming skills and patience.

### 3. Core Architectural Design

We use a Supervisor + Workers pattern with durable execution and rollback.

```
[ Orchestrator (Supervisor) ]  <- Human observation dashboard
  |-- Worker Agent (UI control)
  |-- Worker Agent (Phone control)
  |-- Worker Agent (File & topic scanning)
  |-- Worker Agent (Content creation)
  |-- Worker Agent (Code generation & download)
  |-- Worker Agent (Self-healing)
  `-- Checkpoint & Rollback Module
```

Key components:
- Durable execution - saves state after every action so the system can resume from a known good point.
- Self-healing controller - monitors health, restarts failed agents, rolls back to last good checkpoint.
- Human-on-the-Loop (HOTL) - you observe logs/video feed and can say commands or kill the system, but no approval pop-ups.

### 4. Recommended Tools & Frameworks (Best Knowledge as of 2026)

| Component | Recommended Tool | Why |
|---|---|---|
| Orchestrator (Supervisor) | LangGraph (Python) or CrewAI | Build stateful multi-agent loops with conditionals and error handling. |
| Durable execution | Vilano Runtime or Temporal | Persist workflow state; resume after crashes. |
| Rollback & checkpoints | ModelActionProtocol (MAP) + custom snapshot | Revert to last successful action. |
| Desktop screen + mouse control | Anthropic's Computer Use (Claude API) or OpenInterpreter with pyautogui | AI that sees the screen and moves mouse/keyboard. |
| Phone control | scrcpy (Android screen mirroring) + ADB commands | Let the AI interact with the phone over USB/WiFi. |
| Browser automation | browser-use (Python) or Playwright with AI prompts | AI navigates websites, extracts data, posts content. |
| Local AI models (no API keys) | Ollama (run Llama 3, Mistral, etc.) | For low-cost classification, summarisation, and fallback. |
| Cloud AI (high reasoning) | Anthropic Claude, GPT-4, Grok, DeepSeek (via API) | Use only when necessary; keep API keys secure. |
| Self-healing logic | Custom code using Supervisor pattern + Circuit Breaker + Retry with backoff | Not available out of the box - you must implement. |
| Code generation & execution | E2B (sandboxed cloud environment) or Docker (local) | AI can write, test, and run new code safely. |
| Content creation | MoviePy (video), Pillow (images), gTTS (voice) | AI orchestrates these libraries. |
| Human observation dashboard | Streamlit or Gradio + WebSocket | Show logs, screenshots, and accept text/voice commands. |

### 5. Building the Self-Healing Loop (Step-by-Step)

**5.1. Base Supervisor (LangGraph example)**

```python
from langgraph import StateGraph, Graph

def supervisor(state):
    # Decide which agent to call next based on current goal
    return next_agent

def handle_error(state, error):
    # Log error, rollback to last checkpoint, restart failed agent
    return recovered_state

builder = StateGraph(MyState)
builder.add_node("supervisor", supervisor)
builder.add_node("desktop_agent", desktop_agent_func)
builder.add_node("healing", handle_error)
# ... add edges with fallbacks
```

**5.2. Durable Checkpoints**

Use pickle or sqlite to save the full state (agent memory, file system changes, queue of tasks) after each completed step. Before any risky action (e.g., posting to X), save a checkpoint.

**5.3. Self-Healing Rules**

- Transient failure -> retry 3 times with exponential backoff.
- Agent crash -> restart that agent with the same input.
- Repeated failure -> roll back to previous checkpoint and skip the action.
- Unknown error -> pause and alert you via dashboard.

**5.4. "Good call" / "Bad call" detection**

Implement a Critic Agent (a small LLM) that evaluates each major action:
- Did it move closer to the goal?
- Does it cause unintended side effects?

If "bad", trigger rollback; if "good", allow system to proceed and optionally reboot the next agent in sequence (like a handshake).

**5.5. Agent Duplication & Recovery**

The Supervisor keeps a pool of identical worker processes. If one fails, the Supervisor spawns a new one using a pre-defined "child spec" (environment, code snapshot). For catastrophic failure, restore from the last known good state.

### 6. How to Avoid API Keys & Token Credits (Or Minimise Them)

- Local models via Ollama: Llama 3.1 8B/70B, Mistral, Qwen. They can handle file scanning, summarising topics, and simple decisions.
- Use the local model for the Critic Agent - it doesn't need top-tier reasoning.
- Only call cloud APIs for: Complex code generation, high-quality video script writing, rare/difficult reasoning tasks.
- Batch and cache - don't call APIs for every little thing.

### 7. Making It Learn to Obtain What It Lacks

You need a Knowledge Gap Detector that runs periodically:

1. When a task fails because a library or tool is missing, capture the error.
2. Pass the error to a Code Generation Agent with a prompt: "Write a Python script to download and install the missing dependency, then verify it works."
3. Run the script in a sandbox (Docker/E2B).
4. If successful, add the new capability to the system.
5. If it fails, try a different solution (e.g., ask the LLM to search the web via browser-use).
6. After three failures, notify the human and ask for guidance.

### 8. Creating a Loop of Different AIs

This is called model routing. The Supervisor decides which model to use for which subtask based on cost, speed, and capability.

| Subtask | Preferred Model | Fallback |
|---|---|---|
| Screen reading / mouse control | Anthropic Claude (Computer Use) | GPT-4V |
| Short content (X posts) | Grok or Claude Haiku | Local Llama 3 |
| Long video script | GPT-4 or DeepSeek | Claude Opus |
| Simple classification | Ollama (Mistral) | None |

To avoid endless loops:
- Set a maximum number of steps (e.g., 10).
- Use a LoopGuard that detects if the system is repeating the same action.

### 9. Phone Communication & Screen Viewing

**Viewing phone screen (Android):** Use scrcpy (open-source) to mirror the phone screen to a window on your laptop. The AI (via Computer Use) can see that window.

**Communicating with the phone:**
- adb shell input tap x y - simulate taps
- adb shell input text "hello" - type
- adb shell screencap - capture screen for the AI

### 10. Minimal Viable System (MVS) Roadmap

**Phase 0 - Environment Setup:** Install Python 3.11+, Docker, Ollama, scrcpy, ADB. Get API key for Anthropic.

**Phase 1 - Static Automation:** Write a script that uses pyautogui to open a browser and log into X. Add a local LLM that reads "Jaron's daily topics" from a text file.

**Phase 2 - Supervisor & Checkpoints:** Implement a simple LangGraph with two agents. Add checkpointing (save state to JSON after each step).

**Phase 3 - Self-Healing (Basic):** Wrap each agent in a try/except. On error, restart the agent. Add a Critic Agent.

**Phase 4 - Phone Integration:** Use scrcpy + ADB to let the AI see and tap your phone.

**Phase 5 - Code Generation & Downloads:** Create a "Downloader Agent" that installs missing libraries in Docker.

**Phase 6 - Full Loop with Multiple AIs:** Implement model routing. Add the "reboot the next system" logic.

---

## GEMINI — n8n + Claude MCP Architecture

### What is n8n?

n8n is an open-source, node-based workflow automation tool. Think of it like a highly customizable, self-hostable alternative to Zapier or Make. It allows you to connect different apps, databases, and APIs together to automate repetitive tasks.

| Feature | n8n | Zapier |
|---|---|---|
| Hosting | Can be self-hosted or cloud | Cloud-only |
| Pricing Model | Based on active workflows/users | Heavily gated by tasks executed per month |
| Data Privacy | Excellent for sensitive data (self-hosted) | Data passes through Zapier's servers |
| Complexity | Highly flexible, custom JavaScript/Python | Easier for beginners, harder to customize |

### The Co-Working Architecture

- **The Local Execution Layer (Claude + MCP):** MCP server running locally on Windows. Claude uses MCP tools to type, click, open apps, or read files.
- **The Orchestration Layer (n8n):** Sits outside the local desktop, acts as project manager. Handles timing and system-to-system triggers.

**The Loop:**
1. n8n Triggers the Cycle (time limit or external trigger).
2. n8n calls Claude with the goal via Advanced AI node.
3. Claude takes control via MCP and executes desktop steps.
4. Claude packages success/failure summary and sends back to n8n.
5. n8n closes the loop, waits for next scheduled trigger, spins up same request.

### The Catch-22 Problem

Claude cannot trigger itself on a timer or watch apps for external changes - it only reacts when prompted. n8n connects to AI using API Keys (developer access), not the Claude Max subscription.

**Jaron's Solution:** Use Google Drive as memory, Gmail as trigger buzzer.

**Approach 1 - Subscription Loop (Free):**
1. Start by pasting initial prompt into Claude Desktop App.
2. Claude executes tasks via Max subscription and Windows MCP.
3. Instruct Claude: "When completely finished, send email titled 'LOOP_COMPLETE' and upload state.txt to Google Drive."
4. n8n Gmail Trigger Node watches for LOOP_COMPLETE email.
5. n8n reads state.txt from Drive, types next prompt into Claude app, hits Enter.

**Approach 2 - Hybrid Trigger Loop:**

| Step | Component | Action | Cost |
|---|---|---|---|
| 1. The Brainstorm | n8n + Claude API | Pull state.txt, generate next prompt | Fraction of a cent |
| 2. The Handoff | n8n | Push prompt to local automation | Free |
| 3. The Muscle | Claude Desktop / Cowork | Execute full desktop task via MCP | Free (Max subscription) |
| 4. The Finish | Google Drive & Gmail | Log status to state.txt, send email | Free |
| 5. Loop Restarts | n8n | Gmail triggers n8n to start at Step 1 | Free |

### Full System Architecture (Gemini's Best Design)

**Core Components:**
- Brain: Claude Desktop App (Max subscription) with local MCP servers.
- Memory: current_state.json on Google Drive.
- Coordinator: n8n (running locally on Windows via Docker).
- Local Intermediary: AutoHotkey or Python script that n8n triggers.

**Python Paster snippet (n8n calls this to inject prompts):**
```python
import pyautogui
import time
pyautogui.hotkey('win', 's')
time.sleep(0.5)
pyautogui.write('Claude')
pyautogui.press('enter')
time.sleep(2)
pyautogui.write("Here is your next task based on the state file...")
pyautogui.press('enter')
```

**n8n Connectors used:**
- Google Drive Node (Download & Upload): pull and push tracking state.
- Execute Command Node: run native Windows PowerShell or Python scripts.

**Loop Counter Safety:** In current_state.json include "run_count": 1. If run_count hits 20, skip Claude and send alert text/email to phone.

---

## GROK — Integration Role

**Jaron:** Where can you fit in? You can have tasks set up which Claude can do and with that you can be apart of the system too. What else can you do? You are connected to my Google Drive, Gmail, and GitHub.

**Grok's role in this system:** Grok has proven read+write access to both Google Drive and GitHub from unattended scheduled tasks. In the four-hop relay test: Claude wrote a marker to Drive, Grok read it, Grok wrote it to the repo, Claude verified from the live repo - data intact.

This makes Grok the primary unattended writer in the system - the worker proven to deposit results somewhere durable without needing an attended session.

---

## CLAUDE (Perplexity session) — Architecture Gap Analysis

### Main Gap Identified

The current design is closest to a prompt-driven automation loop with state files and triggers, which is workable but brittle because the same loop is trying to be scheduler, memory, router, actor, and recovery layer at once.

Real missing layers:
- Durable workflow runtime
- Task ledger with typed state
- Capability registry
- Evaluator/critic lane
- Replay/rollback semantics
- Multi-model routing policy

### Better Architecture: 6-Layer Stack

| Layer | Best role | Suggested tool |
|---|---|---|
| Control plane | Long-running orchestration, retries, resumability | Temporal |
| Event bus | Triggering from Gmail, Drive, GitHub, webhooks | n8n |
| Tool exposure | Make workflows callable by AI clients | n8n MCP server |
| Browser/web actor | Websites, posting, scraping, DOM-heavy flows | browser-use or Playwright |
| Desktop actor | Native Windows apps, screen-based actions | Claude Desktop / computer-use |
| Local skill workers | Cheap classification, summarization, watchdogs | Ollama/local models |

### Systems Currently Missing

**1. Task Ledger** - canonical task object, not just state.txt:
- task_id, goal, subgoal, input_refs, assigned_agent, required_capabilities
- status, attempt_count, checkpoint_ref, output_refs, critic_score, next_action
- created_at, updated_at

**2. Capability Registry** - machine-readable registry of:
- Available models, available MCP tools, browser/desktop/phone skills
- Installable missing dependencies, cost/latency/reliability scores

Example entry:
- capability_id: github_commit_and_push
- executor: claude_desktop, mode: desktop
- preconditions: github_desktop_logged_in OR git_cli_available
- risk_level: medium, timeout_sec: 900
- success_evidence: commit_hash && remote_push_confirmed

**3. Evidence System** - agents attach artifacts:
- Screenshot before/after, DOM snapshot, file hash
- Commit hash, email message ID, Drive file ID, terminal output summary

**4. Critic Lane** - own dedicated lane, receives:
- Goal, claimed action, evidence bundle, policy checks, output quality rubric
- Returns: accept / retry_same_agent / reroute_new_agent / rollback_checkpoint / decompose_task

**5. Planner/Executor Split:**
- Planner model for decomposition
- Executor model for action
- Critic model for verification
- Recovery model for repair

**6. Skill Acquisition Pipeline:**
1. Failure classified as missing capability
2. Search registry for substitute
3. If none exists, create acquisition task
4. Sandbox build/test worker tries to install or implement
5. Run validation suite
6. Register capability only if validation passes
7. Requeue original blocked task

### Best Role for Each AI

| Role | Best fit |
|---|---|
| Planner/orchestrator prompts | Claude or GPT-class reasoning model |
| Desktop UI execution | Claude Desktop / computer-use worker |
| Browser-first workflows | browser-use / Playwright worker |
| Fast classification and critic triage | local Ollama model |
| Code repair and workflow generation | coding agent connected to n8n MCP |
| Integration glue | n8n workflows + MCP-exposed actions |

### Priority Upgrades

1. Add Temporal as the durable control plane.
2. Keep n8n as the integration/event layer and MCP tool surface.
3. Create a typed task ledger instead of relying on freeform state files.
4. Separate planner, executor, and critic into different roles.
5. Add evidence-based completion checks for every important action.

### Improved Full Blueprint

1. Ingress: Gmail, Drive changes, GitHub events, cron, webhook enter through n8n.
2. Workflow start: n8n opens or updates a Temporal workflow with structured task data.
3. Planner: planner agent decomposes goal into typed subtasks.
4. Router: selects browser worker, desktop worker, phone worker, or code worker from capability registry.
5. Execution: selected worker performs action and emits artifacts.
6. Critic: evaluator checks artifacts against goal.
7. Commit: accepted result updates ledger and advances workflow.
8. Recovery: if rejected, workflow retries, reroutes, or rolls back from checkpoint.
9. Learning: if capability missing, create acquisition workflow rather than improvising in-place.

---

*Transcript recorded: June 3, 2026*  
*Repository: github.com/JaronKBragg7337/autonomous-agent-os*
