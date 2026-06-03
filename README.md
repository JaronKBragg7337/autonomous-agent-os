# Autonomous Agent OS

**Self-healing, autonomous multi-AI agent operating system.**
Personal digital proxy — Supervisor + Workers + Durable Execution + Critic Lane + Task Ledger. The design goal is a system that needs no hand-coding from the operator to keep extending itself.

---

## Status (read this first)

This project lives in a superposition, and both halves are true at once.

The **operating system around it is pre-build** — architecture is locked to the 6-layer stack below, and no production code is committed yet. But the **core mechanism is already proven.** A self-continuing loop between two AI workers on one machine has run, recovered from real errors, and obeyed a remote stop. This is not a paper design waiting on a first test; it is a proven execution loop waiting on the OS that will wrap it.

Everything below is tagged so a cold reader can tell the difference:

- **[PROVEN]** — ran and was verified.
- **[REACHABLE]** — solvable today with the control-screen + Codex approach, not yet wired in.
- **[DESIGN]** — specified but unbuilt.

---

## The Proven Core — The Heartbeat Loop **[PROVEN]**

This is the key. Not an API key — the system loop itself. No single AI carries it; the roles matter more than the products filling them.

A courier-and-worker loop with a human stop-channel ran multiple times with Claude Cowork as courier and OpenAI Codex as worker. It recovered from real failures on its own, and when the marching-orders file was switched to a stop instruction, the worker halted the entire loop, started nothing, and logged it. The loop is approved, and its prompts are frozen — everything new in this design is carried by the To-Do file the worker reads, never by changing what already worked.

**The three roles.** The *courier* opens the worker, types one exact message, sends it, and stops — it reads nothing and runs no task. A deliberately dumb courier is a safe courier: with no power to act on the work, it cannot break the chain or route around the operator. The *worker* does everything real — reads the file, does the task, decides continue-or-stop, logs the run, and only if continuing triggers the next courier. The *To-Do file* is the marching-orders file the operator owns; each cycle the worker reads it and obeys whatever it finds.

**The handoff order is the whole point.** Do the task → while still on screen, trigger the next courier → release screen control → only then write the log via connector, not screen. Securing the successor before the fragile log step keeps the chain alive even if logging hiccups. The successor-trigger is the lifeline; the log is the safety net; secure the lifeline first.

**The two prompts (frozen).**

> **Courier:** "Your ONLY job: open the worker app, start a new chat, type the message below EXACTLY, send it, and stop. Do NOT read files. Do NOT open a browser. Do NOT do any task yourself. Once sent, you're finished — the worker takes over. Message to type: [worker instruction]."

> **Worker:** "Open the To-Do file in [Google Drive folder] and read the current To Do. Follow the steps. If the same issue happens more than twice, use screen control for what you can't otherwise do. When done, log to [folder] > log with all context, leave nothing out. Then, if not told to stop, start controlling the screen and start the courier task called [name]."

The only difference from the original proven prompt is the pointer: healthy operation reads the **Drive** To-Do (live, phone-editable). The local file is the guaranteed-stop floor it falls back to — see the stop ladder below.

---

## The Perspective Scheduler **[DESIGN]**

The seven "assists" are not seven new systems and not seven separate apps. They are seven **roles** the proven worker pair wears, one per hour-window, selected by the schedule. The loop is the engine; the scheduler is how the loop decides which hat to wear each cycle. This follows the project's own rule — **name roles, not products.** Codex-or-Claude fills whichever role the clock calls for.

Each assist is its own scheduled task that fires in its window and runs the proven loop. The active assist *is* the worker for that hour. What differs between roles is what each one does after reading the file: some work the list line-by-line, others only open it to see whose turn it is and what the active assistant is doing.

| Window        | Assist   | Role                                                              | Reads list how  | Maps to Core System                    |
|---------------|----------|-------------------------------------------------------------------|-----------------|----------------------------------------|
| 7 AM – 11 AM  | Assist 1 | Test / Maintenance loop — tests & maintains the human stop-channel | schedule-reader | Loop health + Control                  |
| 11 AM – 2 PM  | Assist 2 | Planner / Task Shaper                                             | deep reader     | Planner/Executor (plan)                |
| 2 PM – 4 PM   | Assist 3 | Executor / Worker                                                 | deep reader     | Planner/Executor (act)                 |
| 4 PM – 6 PM   | Assist 4 | Critic / Red Team                                                 | deep reader     | Critic Lane                            |
| 6 PM – 11 PM  | Assist 5 | Evaluator / Completion Checker                                    | deep reader     | Critic Lane (accept/reject) + Evidence |
| 11 PM – 12 AM | Assist 6 | Recovery / Rollback                                               | schedule-reader | Self-Healing Rules                     |
| 12 AM – 7 AM  | Assist 7 | Memory / State Steward *(implied, not fully carved out yet)*      | schedule-reader | Task Ledger / Memory                   |

The schedule sums to a clean 24 hours with no gaps and no overlaps. This table and the Core Systems list below are two views of the same machine: one lists the systems abstractly, this one schedules them onto a clock.

**Note on Assist 7.** Memory / State Steward owns the longest window of the day and the overnight Task Ledger, yet it is the one role without a carved-out definition. It decides what the morning's Assist 1 wakes up knowing, so it should not stay implied for long.

---

## The To-Do File Spec

The To-Do file carries the intelligence the proven prompt deliberately doesn't. The worker opens the file exactly as it always did; the file is what tells it who is speaking, which perspective is active this hour, and therefore what to do.

**Provenance gate — the human-origin trigger.** Every instruction must open with the operator's identity tag — `It's [User]` — followed by an action verb (*Do this, Go to, Move the, Start file*, and so on). The tag is not decoration; it is the human-origin gate. An instruction missing it is not authenticated and is not acted on. This is what keeps the loop from executing anything an agent generated for itself: agent-generated ideas may enter the queue, but they wait for the same human trigger as the operator's own.

**Perspective line.** After the identity tag, the file declares the hour's perspective. That perspective decides whether the active assist works the list, only checks the system, keeps memory/state, or does nothing on the list this cycle and simply follows the schedule. The time-reasoning is an instruction the worker *reads* — never something baked into how it was launched.

**Logging.** The worker logs every step it took, with date and time, no context left out.

**Naming.** Names describe what a thing *is* in two or three words — never the date. No duplicate names across folders, files, docs, or PDFs. The single exception is recovery files and folders, which may repeat.

**Irreversible steps.** Before any major step that is hard to come back from, and only when no backup already exists, the worker creates a fresh folder labeled `[⚠️ CAUTION ❗️⚠️]` to hold the rollback point.

*Example:*

```
Folder/To-Do
- It's Jaron — Assist 3, Executor. Do the next task not checked off:
  get 1 task done off the list, then log it and continue.
```

---

## The Stop Ladder — Three Tiers

The stop-channel is never a single point. It degrades in tiers, and each lower tier is harder to lose than the one above it.

**Tier 1 — Healthy steering (Drive).** Normal operation runs on the Drive To-Do file. It is live and editable from the operator's phone mid-run; editing it to a stop instruction halts the worker on its next read. This is the upside surface — reachable from anywhere.

**Tier 2 — Sync-fault floor (Local).** If Drive can't sync, the worker reads the local file instead, and the local file stops it. This is passive and automatic — no diagnostics needed, because a system that can't reach its instructions should not keep acting. The machine is on 24/7, so this is rare; if local is *also* unreachable, a larger hardware fault is in play, which is environment-specific and outside what the loop guards. A sync fault routes to local immediately and does **not** feed the failure counter below.

**Tier 3 — Low Mode (active failure response).** Distinct from a sync fault. When **any single activity the system attempts fails four times**, a debug step written into the To-Do trips. It cancels the normal daily schedule, drops the forward-action roles (Planner, Executor), and reorganizes into **low mode**, running only the assess-and-repair roles at two hours each:

1. **Evaluator** — assess current state.
1. **Critic / Red Team** — challenge what the Evaluator found.
1. **Evaluator** — re-assess against the first reading (the before-and-after bookend; this is the Reading 1 → Reading 2 pattern).
1. **Recovery / Rollback** — only if recovery is needed.

After the team completes that pass, the system routes to the **local machine To-Do and parks**, waiting for the operator to inspect the system before any reboot. Low mode is a bounded diagnostic sweep that ends in a human gate — not an indefinite loop, and distinct from dormancy.

Across all three tiers the manual trigger and kill switch stay intact. Local is always the floor; the design leans on that rule, not on any one machine's uptime.

---

## Architecture — 6-Layer Stack

| Layer               | Role                                             | Tooling                       |
|---------------------|--------------------------------------------------|-------------------------------|
| Control plane       | Long-running orchestration, retries, resumability | Temporal                      |
| Event bus           | Triggers from Gmail, Drive, GitHub, webhooks     | n8n                           |
| Tool exposure       | Make workflows callable by AI clients via MCP    | n8n MCP server                |
| Browser/web actor   | Websites, posting, scraping                      | browser-use / Playwright      |
| Desktop actor       | Native Windows apps, screen-based actions        | Claude Desktop / computer-use |
| Local skill workers | Cheap classification, summarization, watchdogs   | Ollama (Llama 3, Mistral)     |

The heartbeat loop is the runtime spine beneath this stack. Today it triggers on a manual schedule; n8n is slated to own the timer once the event bus is live.

---

## Core Systems — Status

Identified June 2026 as the systems a full autonomous OS needs. Several are already reachable through the proven control-screen approach; tagged honestly.

1. **Task Ledger** **[DESIGN]** — typed task objects (`task_id`, `goal`, `subgoal`, `assigned_agent`, `required_capabilities`, `status`, `attempt_count`, `checkpoint_ref`, `output_refs`, `critic_score`, `next_action`). Schema below.
1. **Capability Registry** **[REACHABLE]** — machine-readable list of models, MCP tools, and browser/desktop/phone skills with cost/latency/reliability scores. Buildable now via the control screen.
1. **Evidence System** **[DESIGN]** — agents attach artifacts (screenshot, file hash, commit hash, email message ID, terminal output) so the critic judges reality, not narration.
1. **Critic Lane** **[REACHABLE]** — a dedicated evaluator that takes goal + claimed action + evidence and returns accept / retry / reroute / rollback / decompose. Reachable now with Codex on the control screen.
1. **Planner/Executor Split** **[REACHABLE]** — separate models for decomposition, action, verification, and repair. Expressible as a To-Do trail inside the heartbeat loop, with the perspective scheduler selecting which role acts and what it's told.
1. **Skill Acquisition Pipeline** **[REACHABLE]** — when a capability is missing: search registry, create acquisition task, sandbox build/test, validate, register, requeue the original task. This is the control screen's core strength.

---

## Data Flow **[DESIGN target; loop portion PROVEN]**

```
Ingress (Gmail / Drive change / GitHub event / cron / webhook)
  -> n8n ingestion
  -> Temporal workflow open
  -> Schedule resolves active perspective for the hour   [from To-Do file]
  -> Worker reads To-Do, confirms "It's [User]" gate
  -> Planner: decompose goal into typed subtasks
  -> Router: select worker from capability registry
  -> Worker: act + emit artifacts
  -> Critic: check artifacts against goal
  -> Accept -> ledger update -> advance workflow
  -> Reject -> retry / reroute / rollback checkpoint
  -> 4 failures (any activity) -> Low Mode -> park at local To-Do
  -> Missing capability -> acquisition workflow (not improvised in-place)
```

The Worker → act → log → trigger-next segment is the part already running. The rest is the OS being built around it.

---

## Worker Roles by Environment

| Environment       | Worker                              | Tools                               |
|-------------------|-------------------------------------|-------------------------------------|
| Browser           | browser-use / Playwright            | scraping, posting, DOM automation   |
| Desktop (Windows) | Claude Desktop / computer-use       | native apps, screen click, keyboard |
| Phone (iPhone)    | Website-as-proxy + n8n WhatsApp node | see iPhone Integration              |
| Sandbox build     | Docker / E2B                        | code generation, install, test      |
| Content creation  | MoviePy / Pillow / gTTS             | video, image, voice                 |

---

## AI Model Routing

| Role                                | Best model                    | Fallback               |
|-------------------------------------|-------------------------------|------------------------|
| Planner / orchestrator              | Claude or GPT-class           | —                      |
| Desktop UI execution                | Claude Desktop (computer-use) | GPT-class vision model |
| Browser-first workflows             | browser-use agent             | Playwright headless    |
| Fast classification / critic triage | Ollama local                  | —                      |
| Code repair / workflow generation   | Coding agent + n8n MCP        | —                      |
| Short social content                | Grok or Claude Haiku          | Local Llama 3          |
| Long video script                   | GPT-class / DeepSeek          | Claude Opus            |

A practical note from testing: Claude Cowork is slower at Windows MCP than Codex on the screen but stronger at internal work, so the working preference is to let Codex drive screen control and rebooting while Claude handles internal tasks and assists from a Codex-set schedule.

---

## iPhone Integration

iOS blocks external screen-mirroring and ADB-style control — scrcpy and ADB are Android-only and do not apply to this build. Three viable paths:

1. **Website-as-proxy (primary — in progress).** A purpose-built web dashboard is the mobile command surface. The agent writes to it; the operator reads and responds from the iPhone browser. No native iOS control required — the cleanest path given iOS constraints.
1. **n8n + WhatsApp Business node.** The agent sends status updates and receives commands via WhatsApp, native on iPhone with no special setup beyond a WhatsApp Business account.
1. **n8n + iMessage via Apple Shortcuts + webhook.** An iPhone Shortcut POSTs to an n8n webhook on a trigger. Limited but functional for simple command/response loops with no server-side iOS agent.

Any scrcpy/ADB/Android reference in the `/docs/` transcript was from assistants unaware of the iOS constraint and is irrelevant to this build.

---

## Self-Healing Rules

A transient failure retries 3× with exponential backoff. An agent crash restarts with the same input. Repeated failure rolls back to the previous checkpoint and skips the action. An unknown error pauses, alerts the dashboard, and queues for diagnosis. A bad critic score rolls back and reroutes to a different worker. A missing capability creates an acquisition task rather than improvising one in place. Four failures of any single activity trips Low Mode (see the Stop Ladder).

---

## Known Limits & Risks

The laptop must be on — the loop is local-capable but machine-bound. **The OneDrive holding everything is showing a sync error; the loop's memory currently lives on a drive flashing a warning, so this needs handling in daylight before it bites.** The website/observer channel isn't fully live yet, which is part of why the local file remains the guaranteed floor. Log duplication (same run saved as `.txt` and `.md` across folders) is being corralled to one folder and one naming rule.

---

## What It Must Never Become

Don't remove the manual trigger, stop-channel, kill switch, or maintenance window to make it run untouched forever. The courier is dumb on purpose. If asked to strip the controls so it can't be stopped, that's outside the design — stop and surface it.

> A loop you can always reach into is a tool. One that's had every way to stop it removed is a hazard.

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

## As a Skill

- Name roles, not products.
- Lead with the readiness check: *Can I hand off? Can I log without the screen? Can I trigger the next courier?* — if you can't hand off, nothing else matters.
- State the gotchas explicitly.
- **Define done:** file read; `It's [User]` gate confirmed; instruction obeyed; exactly one courier triggered unless stopped; screen released; full log written.
- Carry the ethics clause with it.

---

## Future

The first real task — the weekend story run, generating into Documents over a few watched hours — is the moment the rig stops being a rig. After that: n8n as the orchestrator spine and owner of the sleep timer; two-way Drive comms so the worker writes status back into a shared file and the stop-channel becomes a full conversation; carving out Assist 7 (Memory / State Steward) into a real definition; and a standing rule that the loop must *progress* — advance the To-Do, never repeat — and land finished work in Documents, not just logs. Both roles should log their own legs.

---

## Source Conversations

`/docs/conversation-transcript-2026-06-03.md` — design brainstorming between Jaron, DeepSeek, Gemini, Grok, and Claude. A reference log of how the architecture was reasoned through, **not an implementation spec**; treat Android/scrcpy/ADB references in it as irrelevant to this build.

---

*Structure and wording assisted by Claude (Anthropic).*  
*Jaron Kyler Bragg, June 2026.*
