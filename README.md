# Autonomous Agent OS

**Self-healing, autonomous multi-AI agent operating system.**  
Personal digital proxy — Supervisor + Workers + Durable Execution + Critic Lane + Task Ledger.

---

## What This Is

This repo is the design home for a fully autonomous agent system that:

- Acts as Jaron at the computer — desktop control, browser control, phone integration.
- Runs a loop of multiple AI models (Claude, ChatGPT, Grok, DeepSeek, Ollama) without manual approval.
- Self-heals: if an agent fails, another fixes or replaces it. Rolls back bad calls, reboots workers.
- Creates content for monetisation (X posts, YouTube/TikTok).
- Writes code to improve its own capabilities and expand its tool registry.
- Uses Google Drive as memory and Gmail as event triggers.

---

## Architecture — 6-Layer Stack

| Layer | Role | Tooling |
|---|---|---|
| Control plane | Long-running orchestration, retries, resumability | Temporal |
| Event bus | Triggers from Gmail, Drive, GitHub, webhooks | n8n |
| Tool exposure | Make workflows callable by AI clients via MCP | n8n MCP server |
| Browser/web actor | Websites, posting, scraping | browser-use / Playwright |
| Desktop actor | Native Windows apps, screen-based actions | Claude Desktop / computer-use |
| Local skill workers | Cheap classification, summarization, watchdogs | Ollama (Llama 3, Mistral) |

---

## Core Missing Systems (Identified June 2026)

1. **Task Ledger** — typed task objects with task_id, goal, subgoal, assigned_agent, required_capabilities, status, attempt_count, checkpoint_ref, output_refs, critic_score, next_action.
2. **Capability Registry** — machine-readable list of available models, MCP tools, browser/desktop/phone skills, cost/latency/reliability scores.
3. **Evidence System** — agents attach artifacts (screenshot, file hash, commit hash, email message ID, terminal output) so the critic judges reality, not narration.
4. **Critic Lane** — dedicated evaluator that receives goal + claimed action + evidence bundle and returns accept / retry_same_agent / reroute_new_agent / rollback_checkpoint / decompose_task.
5. **Planner/Executor Split** — separate models for decomposition, action, verification, and repair.
6. **Skill Acquisition Pipeline** — when a capability is missing: search registry, create acquisition task, sandbox build/test, validate, register, requeue original task.

---

## Data Flow

```
Ingress (Gmail / Drive change / GitHub event / cron / webhook)
    -> n8n ingestion
    -> Temporal workflow open
    -> Planner: decompose goal into typed subtasks
    -> Router: select worker from capability registry
    -> Worker: act + emit artifacts
    -> Critic: check artifacts against goal
    -> Accept -> ledger update -> advance workflow
    -> Reject -> retry / reroute / rollback checkpoint
    -> Missing capability -> acquisition workflow (not improvised in-place)
```

---

## Worker Roles by Environment

| Environment | Worker | Tools |
|---|---|---|
| Browser | browser-use / Playwright | web scraping, posting, DOM automation |
| Desktop (Windows) | Claude Desktop / computer-use | native apps, screen click, keyboard |
| Phone (iPhone) | Website-as-proxy + n8n WhatsApp node | see iPhone Integration section below |
| Sandbox build | Docker / E2B | code generation, install, test |
| Content creation | MoviePy / Pillow / gTTS | video, image, voice |

---

## iPhone Integration

iOS blocks all external screen-mirroring and ADB-style control — scrcpy and ADB are Android-only and do not apply here.

Three viable iPhone-compatible paths:

**1. Website-as-proxy (primary approach — already in progress)**  
A purpose-built web dashboard acts as the mobile command surface. The agent writes to it; Jaron reads and responds from iPhone via browser. No native iOS control required. This is the cleanest path given iOS constraints.

**2. n8n + WhatsApp Business node**  
n8n has a native WhatsApp Business node. The agent sends status updates and receives commands via WhatsApp message — works natively on iPhone with no special setup beyond a WhatsApp Business account.

**3. n8n + iMessage via Apple Shortcuts + webhook**  
An iPhone Shortcut can POST to an n8n webhook on a trigger (e.g., receiving a specific iMessage). Limited but functional for simple command/response loops without any server-side iOS agent.

> **Note:** Any reference to scrcpy, ADB, or Android in the brainstorming transcript in `/docs/` does not apply to this build. Those were suggestions from AI assistants unaware of the iPhone constraint.

---

## AI Model Routing

| Role | Best model | Fallback |
|---|---|---|
| Planner / orchestrator | Claude or GPT-class | — |
| Desktop UI execution | Claude Desktop (computer-use) | GPT-4V |
| Browser-first workflows | browser-use agent | Playwright headless |
| Fast classification / critic triage | Ollama local model | — |
| Code repair / workflow generation | Coding agent + n8n MCP | — |
| Short social content (X posts) | Grok or Claude Haiku | Local Llama 3 |
| Long video script | GPT-4 / DeepSeek | Claude Opus |

---

## Self-Healing Rules

- Transient failure -> retry 3x with exponential backoff.
- Agent crash -> restart with same input.
- Repeated failure -> rollback to previous checkpoint, skip action.
- Unknown error -> pause, alert dashboard, queue for diagnosis.
- Bad critic score -> rollback and reroute to different worker.
- Missing capability -> create acquisition task, do not improvise.

---

## Task Ledger Schema

```json
{
  "task_id": "uuid",
  "goal": "string",
  "subgoal": "string",
  "input_refs": [],
  "assigned_agent": "string",
  "required_capabilities": [],
  "status": "pending | running | success | failed | rolled_back",
  "attempt_count": 0,
  "checkpoint_ref": "string",
  "output_refs": [],
  "critic_score": null,
  "next_action": "string",
  "created_at": "ISO-8601",
  "updated_at": "ISO-8601"
}
```

---

## Source Conversations

`/docs/conversation-transcript-2026-06-03.md` — Design brainstorming session (vibe coding) between Jaron, DeepSeek, Gemini, Grok, and Claude. Kept as a reference log of how the architecture was reasoned through. **Not an implementation spec** — treat Android/scrcpy/ADB references in it as irrelevant to this build.

---

## Status

**Phase: Design / Pre-build.**  
No code committed yet. Architecture locked to the 6-layer stack above. Build starts with Phase 0 (environment setup) per the MVS roadmap in the transcript.

---

*Structure and wording assisted by Claude (Anthropic).*  
*Jaron Bragg, June 2026.*
