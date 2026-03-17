# Process completion notifications

How the agent can kick off a process and be notified when it is done (callbacks, webhooks, exec notify-on-exit).

---

## 1. Same machine: exec tool + notifyOnExit

When the agent runs a **background** command via the exec tool, the gateway can notify the agent when that process exits.

- **Mechanism:** On process exit, the runtime enqueues a **system event** (short summary: exit code and tail of output) for that session and calls **`requestHeartbeatNow()`** so the agent is woken immediately.
- **Config:** `tools.exec.notifyOnExit` (default **true**). Optional: `tools.exec.notifyOnExitEmptySuccess` to also notify when the process exits with no output.
- **Scope:** Only for processes the agent started on the same machine via the exec tool; no HTTP callback.

**Flow:** Agent starts a background job → process exits → agent is woken in the same session and sees the system line about the exec completion.

---

## 2. External process: gateway webhook (callback)

When the “process” is **external** (CI job, script on another server, cloud function), it can **call the gateway** when it is done. The gateway then wakes the agent with a message.

- **Endpoint:** `POST /hooks/wake` or `POST /hooks/agent` (see [Webhooks](/automation/webhook)).
- **Auth:** Header `Authorization: Bearer <token>` or `x-openclaw-token: <token>` (config: `hooks.token`).
- **Enable:** `hooks.enabled: true`, `hooks.token` set, gateway reachable (e.g. `http://gateway:18789`).

### `POST /hooks/wake`

- Body: `{ "text": "Build finished successfully", "mode": "now" }`.
- Effect: That text is enqueued as a **system event** for the **main** session; if `mode: "now"`, an immediate heartbeat is requested → agent sees “Build finished successfully” and runs.

### `POST /hooks/agent`

- Body: `{ "message": "Deploy completed. Log: ...", "sessionKey": "hook:deploy:123", "wakeMode": "now", ... }`.
- Effect: Starts an **isolated** agent run (or reuses a stable session if you allow and reuse `sessionKey`); with `wakeMode: "now"` the agent runs immediately. Optional: `deliver`, `channel`, `to` to send the reply to a chat.

### Typical flow

1. Agent (or you) starts the external process and passes it a “completion URL”, e.g. `https://your-gateway/hooks/wake` plus the token.
2. When the process finishes, it does e.g. `curl -X POST .../hooks/wake -H "Authorization: Bearer TOKEN" -d '{"text":"Build done","mode":"now"}'`.
3. Gateway enqueues the event and wakes the agent; agent gets “Build done” (or full message via `/hooks/agent`) and can continue.

**Callback = external process POSTing to the gateway webhook when done.**

---

## 3. Cron job completion → outbound webhook

For **scheduled** agent runs (cron), when a job **finishes**, the gateway can POST to an external URL (outbound webhook).

- **Config:** In the cron job, `delivery.mode = "webhook"` and `delivery.to = <HTTPS URL>`.
- **Behavior:** When the job’s finished event is emitted (with summary), the gateway POSTs that payload to `delivery.to`; optional `cron.webhookToken` adds `Authorization: Bearer <token>`.
- **Use case:** Not “agent starts one process and gets notified when that process is done,” but “when this scheduled run completes, notify an external system.”

See [Cron jobs](/automation/cron-jobs) for delivery options.

---

## 4. In-process: agent.wait and lifecycle events

Inside the same process, code can wait for an agent run to finish:

- **Gateway method:** `agent.wait` with `runId` and `timeoutMs` — blocks until that run ends (or timeout).
- **Events:** Lifecycle stream (`phase`: `start` / `end` / `error`) is subscribed to by the gateway to update state and run cleanup.

This is for **same-process** coordination (e.g. subagents), not for “external process calls back.”

---

## Summary

| Scenario | How the agent is notified |
|----------|---------------------------|
| **Agent runs a background command (same machine)** | Exec tool with **notifyOnExit** (default on): process exit → system event + heartbeat → agent woken in same session. |
| **External process (CI, other server, script)** | That process **POSTs to gateway webhook** when done: `POST /hooks/wake` or `POST /hooks/agent` → agent gets the message and is woken (callback pattern). |
| **Scheduled cron run finishes** | Cron can **POST to your URL** (`delivery.mode = "webhook"`, `delivery.to = URL`) so an external system is notified (outbound). |
| **Same-process agent run** | **agent.wait** or lifecycle event subscription (internal only). |

The callback-style “notify the agent when it is done” is the **inbound webhook** (`/hooks/wake` or `/hooks/agent`).
