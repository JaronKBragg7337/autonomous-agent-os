# Autonomous Agent OS

**Self-healing, autonomous multi-AI agent operating system.**  
Personal digital proxy — Supervisor + Workers + Durable Execution + Critic Lane + Task Ledger.

---

## What This Is

This repo is the design home for a fully autonomous agent system that:

- Acts as Jaron at the computer — desktop control, browser control, phone control.
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
| Phone (Android) | scrcpy + ADB | tap, type, screencap, notification read |
| Sandbox build | Docker / E2B | code generation, install, test |
| Content creation | MoviePy / Pillow / gTTS | video, image, voice |

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

`/docs/conversation-transcript-2026-06-03.md` contains the full verbatim design conversation between Jaron, DeepSeek, Gemini, Grok, and Claude that produced this architecture.

---

## Status

**Phase: Design / Pre-build.**  
No code committed yet. Architecture locked to the 6-layer stack above. Build starts with Phase 0 (environment setup) per the MVS roadmap in the transcript.

---

*Structure and wording assisted by Claude (Anthropic).*  
*Jaron Bragg, June 2026.*
