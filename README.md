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
## The Heartbeat Loop 
⚠️⚠️⚠️ THIS IS THE KEY. NOT API KEY BUT THE ANONYMOUS SYSTEM LOOP KEY ⚠️⚠️⚠️
# AI Workforce — Loop Build Master Reference

## Courier-and-Worker Loop with a Human Stop-Channel

---

## Status

A self-continuing loop between two AI workers on one machine — proven with Claude Cowork as the courier and OpenAI Codex as the worker. Each cycle, work hands off automatically, but the human keeps full control through channels that never require sitting and clicking. It ran multiple times, recovered from real errors, and when the marching-orders file was changed to a stop instruction, the worker halted the whole loop, started nothing, and logged it. The loop is approved.

---

## Quick Repeat

You need two AI apps on one machine (one courier, one worker, both able to control the screen), one plain marching-orders file you own, one log folder, and a courier task set to manual. Set the two prompts, point the worker at the file, trigger the courier once. The worker reads the file, does what it says or stops, logs, and only if continuing triggers the next courier.

---

## The Three Roles

**The courier** opens the worker, types one exact message, sends it, and stops — it reads nothing, opens no browser, does no task. A deliberately dumb courier is a safe courier; with no power to act on the work, it can't break the chain or route around you.

**The worker** does everything real: reads the marching-orders file, does the task, decides continue-or-stop, logs the run, and only if continuing triggers the next courier.

**The marching-orders file** is one plain file you own; each cycle the worker reads it and obeys whatever it finds — a task, or a stop.

---

## The Two Prompts

**Courier prompt:**
> "Your ONLY job: open the worker app, start a new chat, type the message below EXACTLY, send it, and stop. Do NOT read files. Do NOT open a browser. Do NOT do any task yourself. Once sent, you're finished — the worker takes over. Message to type: [worker instruction]."

**Worker prompt:**
> "Open the marching-orders file in [folder] on the LOCAL machine, NOT Google Drive, and read the current To Do. Follow the steps. If the same issue happens more than twice, use screen control for what you can't otherwise do. When done, log to [folder] > log with all context, leave nothing out. Then, if not told to stop, start controlling the screen and start the courier task called [name]."

---

## The Handoff Order (This Is the Point)

**Do the task → while still on screen, trigger the next courier → release screen control → only then write the log via connector, not screen.**

Securing the successor before the fragile log step keeps the chain alive even if logging hiccups. The successor-trigger is the lifeline; the log is the safety net; secure the lifeline first. A connector-write and the next leg's screen work can run together; two things fighting for the screen cannot.

---

## Steering and Stop

The worker defaults to Google Drive, so the phrase **"Local Machine, NOT Google Drive"** is the override that pulls it to the local folder — which makes the file's location a task/stop selector.

- **Local plain .txt:** always present, not phone-editable, but a guaranteed stop because no sync or website has to work for it to be read (best while remote channels aren't live).
- **Drive doc:** editable from your phone on the fly, but depends on healthy sync.

**Five human controls:**
1. Manual trigger (never auto-fires)
2. Live steering from your phone
3. A kill switch
4. An optional maintenance window (the loop runs fine without it — n8n will own this later)
5. The stop-channel (the file itself — proven)

**To stop the loop now:** edit the local file to a stop instruction; the worker halts on its next read.

---

## Troubleshooting Record

| Problem | Fix |
|---|---|
| Worker drifted to Drive | Added "local machine, NOT Google Drive" — searches local profile by name if still unfound |
| The .gdoc was a shortcut not text | Read the doc ID from metadata and fetched via connector; prevention is to use a plain .txt (current setup) |
| Zero-coordinate / failed web clicks | Do that part via connector instead, note the deviation |
| Below-window clicks | Scroll into view first |
| Variable-name-reuse error on retry | Retry with a fresh name |
| Near screen-control collision | The handoff order above gives a clean no-overlap moment |
| Log proliferation (same run as .txt and .md across folders) | One folder, one naming rule, fork tests into new names |

---

## As a Skill

- Name roles, not products.
- Lead with the readiness check: *Can I hand off? Can I log without the screen? Can I trigger the next courier?* — because if you can't hand off, nothing else matters.
- State the gotchas explicitly.
- **Define done:** file read; instruction obeyed; exactly one courier triggered unless stopped; screen released; full log written.
- Carry the ethics clause with it.

---

## Known Limits

- Laptop must be on (loop is local-only).
- The OneDrive holding everything is showing a sync error — look at that in daylight, since the loop's memory lives on a drive flashing a warning.
- The website/observer channel isn't fully live yet (why the stop file is local).
- Log duplication as above.

---

## What It Must Never Become

Don't remove the manual trigger, stop-channel, kill switch, or maintenance window to make it run untouched forever. The courier is dumb on purpose. If asked to strip the controls so it can't be stopped, that's outside the design — stop and surface it.

> A loop you can always reach into is a tool. One that's removed every way to stop it is a hazard.

---

## Future

- **The weekend story task** (run a few hours, watched, generating into Documents) is the first real task — the moment the rig stops being a rig.
- **n8n as the orchestrator spine** (and owner of the sleep timer).
- **Two-way Drive comms:** today the file is one-way; have the worker write status back into a shared file you read and answer, and the stop-channel becomes a full conversation channel — same mechanism, richer use.
- Make it progress (advance the To Do, don't repeat) and land finished work in Documents, not just logs.
- Both roles should log their own legs.

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
