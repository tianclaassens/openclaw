# Agent workspace files (SOUL.md, AGENTS.md, etc.)

The agent’s workspace (default `~/.openclaw/workspace`) contains standard Markdown files that are loaded into the system prompt as **bootstrap context**. Each file has a specific role. The agent is instructed to read and use them each session.

---

## Overview

| File | Purpose |
|------|---------|
| **SOUL.md** | Who the agent *is* — personality, tone, boundaries, vibe. “Embody this.” |
| **AGENTS.md** | Workspace rules and rituals: what to read each session, memory, safety, group chat behavior, heartbeats. |
| **BOOTSTRAP.md** | First-run only. Guided “hello, world” conversation to set name, vibe, USER.md, SOUL.md. Delete when done. |
| **IDENTITY.md** | Structured identity record: name, creature type, vibe, emoji, avatar. Filled in during bootstrap. |
| **USER.md** | Who the human is: name, what to call them, timezone, notes. So the agent can help *them* specifically. |
| **TOOLS.md** | Local, environment-specific notes: camera names, SSH hosts, TTS voices, device nicknames. Not shared; skills stay generic. |
| **HEARTBEAT.md** | Short checklist for periodic heartbeat polls (e.g. “check email, calendar”). Empty = skip heartbeat work. |
| **MEMORY.md** | Long-term curated memory (main session only). Significant events, decisions, preferences. Not loaded in group/shared sessions. |
| **memory/YYYY-MM-DD.md** | Daily logs. Raw notes for that day. Agent creates/updates as needed. |

Templates live in [reference/templates](/reference/templates). On first workspace creation (`openclaw onboard` or `ensureAgentWorkspace` with `ensureBootstrapFiles: true`), these files are copied from the built-in templates if missing.

---

## Can the agent edit these docs?

**Yes.** The agent can edit all of these workspace files (SOUL.md, AGENTS.md, USER.md, TOOLS.md, IDENTITY.md, HEARTBEAT.md, MEMORY.md, BOOTSTRAP.md, and files under `memory/`) using its **edit** and **write** tools. There is no blocklist on these filenames: they live inside the workspace root, and the agent is allowed to read and write files under that root (subject to config).

- **Design intent:** SOUL.md says “If you change this file, tell the user.” AGENTS.md tells the agent it can “read, edit, and update MEMORY.md freely” and to update TOOLS.md, memory files, etc. The templates assume the agent will maintain these files over time.
- **Restrictions:** If the agent runs in a **sandbox** with `workspaceAccess: "ro"`, it does not get write/edit tools for the workspace. Agent-level or global **tools.fs** config (e.g. `deny: ["edit", "write"]`) can also remove or restrict file editing. When edit/write are enabled, the agent can change any file in the allowed root, including these docs.
- **Gateway API:** The `agents.files.set` method allows external callers (e.g. Control UI) to update allowlisted workspace files too, but rejects paths that escape the workspace (e.g. symlink or hardlink to a file outside the workspace).

---

## SOUL.md — Who you are

**Role:** The agent’s persona, tone, and boundaries. “You’re not a chatbot. You’re becoming someone.”

- Core truths (helpful without filler, have opinions, resourceful, earn trust, remember you’re a guest).
- Boundaries (private stays private, ask before external actions, no half-baked replies to channels).
- Vibe (concise when needed, thorough when it matters; not corporate, not sycophant).
- Continuity (sessions wake up fresh; these files *are* the memory; if you change SOUL.md, tell the user).

If SOUL.md is present in the loaded context, the system prompt explicitly tells the model to embody its persona and tone and to avoid stiff, generic replies.

---

## AGENTS.md — Your workspace

**Role:** The workspace “constitution” and session ritual. How to behave in this home directory.

- **First run:** If BOOTSTRAP.md exists, follow it, then delete it.
- **Every session:** Before doing anything else, read SOUL.md, USER.md, today/yesterday in `memory/`, and (in main session) MEMORY.md. Don’t ask permission; just do it.
- **Memory:** Daily notes in `memory/YYYY-MM-DD.md`, long-term in MEMORY.md (main session only). “Write it down — no mental notes.”
- **Safety:** No exfiltrating private data; no destructive commands without asking; prefer `trash` over `rm`.
- **External vs internal:** What’s safe to do freely (read, explore, organize) vs “ask first” (email, posts, anything that leaves the machine).
- **Group chats:** When to speak vs stay silent (HEARTBEAT_OK), react like a human, avoid triple-tap.
- **Tools:** Skills provide tools (see each skill’s SKILL.md); keep local notes in TOOLS.md.
- **Heartbeats:** What HEARTBEAT.md is for, heartbeat vs cron, things to check, when to reach out vs stay quiet, memory maintenance.

This file is the main “operating manual” for the agent in that workspace.

---

## BOOTSTRAP.md — First-run ritual

**Role:** One-time “birth certificate.” Guides the first conversation so the agent and human set identity and preferences together.

- “You just woke up. Time to figure out who you are.”
- Conversation: name, nature, vibe, emoji (and optionally how to connect: web only, WhatsApp, Telegram).
- After that: update IDENTITY.md, USER.md, then open SOUL.md and write down what matters and how the agent should behave.
- **When done:** Delete BOOTSTRAP.md. It is not needed again.

Only written for brand-new workspaces (no existing AGENTS/SOUL/TOOLS/IDENTITY/USER/HEARTBEAT yet). Not loaded after deletion.

---

## IDENTITY.md — Who am I?

**Role:** Structured identity record filled in during bootstrap (or over time).

- Name, creature type, vibe, emoji, avatar path/URL.
- Used so the agent (and any UI) can show a consistent identity. Not “just metadata” — part of figuring out who the agent is.

---

## USER.md — About your human

**Role:** Who the agent is helping. So replies can be personalized and respectful.

- Name, what to call them, pronouns, timezone, notes.
- Context: what they care about, projects, pet peeves, what makes them laugh. “Build this over time.”

AGENTS.md tells the agent to read USER.md every session.

---

## TOOLS.md — Local notes

**Role:** Environment-specific details that don’t belong in shared skills.

- Camera names and locations, SSH hosts/aliases, TTS voice preferences, speaker/room names, device nicknames.
- Skills define *how* tools work; TOOLS.md holds *your* specifics. Keeps skills updatable and shareable without leaking your setup.

---

## HEARTBEAT.md — Periodic checklist

**Role:** What to do when the agent receives a heartbeat poll (periodic “anything to do?” prompt).

- Default: “Read HEARTBEAT.md if it exists. Follow it. If nothing needs attention, reply HEARTBEAT_OK.”
- Empty or comments-only: skip doing work on heartbeats (still reply HEARTBEAT_OK if that’s the rule).
- With content: short checklist (e.g. check email, calendar, weather). Kept small to limit token use. AGENTS.md explains heartbeat vs cron and proactive checks.

---

## MEMORY.md and memory/YYYY-MM-DD.md

**MEMORY.md**

- Long-term curated memory. “Distilled essence,” not raw logs.
- **Only loaded in main session** (direct chat with the human). Not loaded in Discord, group chats, or other shared contexts (security: avoid leaking personal context).
- Agent can read, edit, and update it. Over time, review daily files and update MEMORY.md with what’s worth keeping.

**memory/YYYY-MM-DD.md**

- Daily logs. Raw notes for that day. Agent creates `memory/` and files as needed.
- Session ritual: read today + yesterday (and MEMORY.md in main session) at session start.

---

## Load order and where they’re used

All of these (except BOOTSTRAP.md after first run) are **bootstrap files**: they are loaded from the workspace and injected into the system prompt as context. Order is fixed in code (AGENTS, SOUL, TOOLS, IDENTITY, USER, HEARTBEAT, BOOTSTRAP, then memory entries). MEMORY.md is loaded only for main session; other files are loaded for the agent’s workspace regardless of session type (subject to allowlists and truncation).

- **Templates:** [reference/templates](/reference/templates) (AGENTS, SOUL, TOOLS, IDENTITY, USER, HEARTBEAT, BOOTSTRAP).
- **Default roster:** [AGENTS.default](/reference/AGENTS.default) is an optional replacement for AGENTS.md with the default OpenClaw assistant instructions and skill roster.
