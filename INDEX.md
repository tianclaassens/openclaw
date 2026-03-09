# OpenClaw documentation index

Single place to find and reference docs across the repo. Paths are repo-root relative.

**Canonical user docs (Mintlify):** [https://docs.openclaw.ai](https://docs.openclaw.ai) — use for linking in README/GitHub.  
**Detailed doc links:** see the [Docs](README.md#docs) section in [README.md](README.md).

---

## Root-level docs

| Doc | Description |
|-----|-------------|
| [README.md](README.md) | Project overview, install, quick start, channels, docs pointers |
| [VISION.md](VISION.md) | Project vision, priorities, direction |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Contribution guide, maintainers, PRs |
| [DOCKER.md](DOCKER.md) | Docker build, tag, push (Docker Hub) |
| [CHANGELOG.md](CHANGELOG.md) | User-facing release notes |
| [SECURITY.md](SECURITY.md) | Security policy, reporting |
| [CLAUDE.md](CLAUDE.md) | Repo guidelines for AI/agents (symlink to AGENTS.md) |
| [AGENTS.md](AGENTS.md) | Agent/maintainer rules and conventions |

---

## Docs published at docs.openclaw.ai

Source lives under `docs/` (Mintlify). Top-level sections:

| Section | Path | Notes |
|---------|------|--------|
| Start | `docs/start/` | Getting started, wizard, onboarding, showcase |
| Install | `docs/install/` | Updating, Docker, Nix, dev channels, platforms |
| Gateway | `docs/gateway/` | Runbook, config, security, discovery, health, remote |
| Channels | `docs/channels/` | Per-channel guides, groups, troubleshooting |
| Concepts | `docs/concepts/` | Architecture, agent, session, models, queue, etc. |
| Tools | `docs/tools/` | Browser, skills, agent-send, exec, ClawHub, etc. |
| Web | `docs/web/` | Control UI, dashboard, WebChat |
| Nodes | `docs/nodes/` | Voice wake, talk, camera, audio, images |
| Platforms | `docs/platforms/` | macOS, iOS, Android, Windows, Linux, Pi, cloud |
| Automation | `docs/automation/` | Cron, webhook, Gmail Pub/Sub, [process-completion-notifications](docs/automation/process-completion-notifications.md) |
| Providers | `docs/providers/` | Model providers (OpenAI, Anthropic, etc.) |
| Reference | `docs/reference/` | RPC, templates (AGENTS, BOOTSTRAP, etc.), RELEASING, [agent-workspace-files](docs/reference/agent-workspace-files.md) |
| Help | `docs/help/` | FAQ, troubleshooting, debugging, scripts |
| Security | `docs/security/` | Threat model, reporting; see [docs/security/README.md](docs/security/README.md) |
| Plugins | `docs/plugins/` | Plugin manifest, agent tools |
| CLI | `docs/cli/` | CLI reference |
| Design / refactor / experiments | `docs/design/`, `docs/refactor/`, `docs/experiments/` | Internal design and refactor notes |

**i18n:** `docs/zh-CN/` is generated (see [docs/.i18n/README.md](docs/.i18n/README.md)); `docs/ja-JP/` may exist.

---

## READMEs by location

### Apps

| Path | Purpose |
|-----|---------|
| [apps/macos/README.md](apps/macos/README.md) | macOS app dev, packaging, signing |
| [apps/ios/README.md](apps/ios/README.md) | iOS node |
| [apps/android/README.md](apps/android/README.md) | Android node |

### Extensions (channel/plugin packages)

| Path | Purpose |
|-----|---------|
| [extensions/bluebubbles/README.md](extensions/bluebubbles/README.md) | BlueBubbles (iMessage) |
| [extensions/copilot-proxy/README.md](extensions/copilot-proxy/README.md) | Copilot proxy |
| [extensions/diffs/README.md](extensions/diffs/README.md) | Diffs extension |
| [extensions/google-gemini-cli-auth/README.md](extensions/google-gemini-cli-auth/README.md) | Google Gemini CLI auth |
| [extensions/llm-task/README.md](extensions/llm-task/README.md) | LLM task |
| [extensions/lobster/README.md](extensions/lobster/README.md) | Lobster |
| [extensions/minimax-portal-auth/README.md](extensions/minimax-portal-auth/README.md) | Minimax portal auth |
| [extensions/nostr/README.md](extensions/nostr/README.md) | Nostr |
| [extensions/open-prose/README.md](extensions/open-prose/README.md) | Open Prose |
| [extensions/qwen-portal-auth/README.md](extensions/qwen-portal-auth/README.md) | Qwen portal auth |
| [extensions/tlon/README.md](extensions/tlon/README.md) | Tlon |
| [extensions/twitch/README.md](extensions/twitch/README.md) | Twitch |
| [extensions/voice-call/README.md](extensions/voice-call/README.md) | Voice call |
| [extensions/zalo/README.md](extensions/zalo/README.md) | Zalo |
| [extensions/zalouser/README.md](extensions/zalouser/README.md) | Zalo Personal |

### Source and scripts

| Path | Purpose |
|-----|---------|
| [src/hooks/bundled/README.md](src/hooks/bundled/README.md) | Bundled hooks (session-memory, command-logger, etc.) |
| [scripts/shell-helpers/README.md](scripts/shell-helpers/README.md) | ClawDock shell helpers |

### Docs and assets

| Path | Purpose |
|-----|---------|
| [docs/security/README.md](docs/security/README.md) | Security & trust, threat model links |
| [docs/.i18n/README.md](docs/.i18n/README.md) | Docs i18n (glossary, translation memory) |
| [assets/chrome-extension/README.md](assets/chrome-extension/README.md) | Chrome extension assets |

### Other

| Path | Purpose |
|-----|---------|
| [Swabble/README.md](Swabble/README.md) | Swabble project (if present) |

---

## Other doc patterns

- **Hooks:** `src/hooks/bundled/<name>/HOOK.md` — per-hook description (e.g. session-memory, command-logger, boot-md, bootstrap-extra-files).
- **Skills:** `skills/<name>/SKILL.md` — per-skill usage; many under `skills/`.
- **Agent rules:** `src/gateway/server-methods/AGENTS.md`, `.agents/skills/` (e.g. PR_WORKFLOW.md).
- **Reference:** `docs/reference/RELEASING.md`, `docs/reference/AGENTS.default.md`, `docs/reference/templates/` (AGENTS, BOOTSTRAP, IDENTITY, SOUL, TOOLS, USER).
- **Platform release:** `docs/platforms/mac/release.md` — macOS app release checklist.
- **Process completion:** [docs/automation/process-completion-notifications.md](docs/automation/process-completion-notifications.md) — how the agent gets notified when a process (exec or external) finishes; webhooks, cron delivery.
- **Agent workspace files:** [docs/reference/agent-workspace-files.md](docs/reference/agent-workspace-files.md) — what SOUL.md, AGENTS.md, BOOTSTRAP.md, IDENTITY.md, USER.md, TOOLS.md, HEARTBEAT.md, MEMORY.md are for.
- **Preflight transcription:** [docs/nodes/preflight-transcription.md](docs/nodes/preflight-transcription.md) — voice-note transcription before mention detection in groups; config, implementation, troubleshooting.
- **Pi Docker + Whisper preflight:** [docs/platforms/raspberry-pi-whisper-preflight.md](docs/platforms/raspberry-pi-whisper-preflight.md) — checklist for Whisper API and preflight on OpenClaw-in-Docker on Raspberry Pi.

---

## Quick reference

- **Getting started:** [docs.openclaw.ai/start/getting-started](https://docs.openclaw.ai/start/getting-started) or `openclaw onboard`
- **Config reference:** [docs.openclaw.ai/gateway/configuration](https://docs.openclaw.ai/gateway/configuration)
- **Channels:** [docs.openclaw.ai/channels](https://docs.openclaw.ai/channels)
- **Troubleshooting:** [docs.openclaw.ai/channels/troubleshooting](https://docs.openclaw.ai/channels/troubleshooting), `openclaw doctor`
- **Updating:** [docs.openclaw.ai/install/updating](https://docs.openclaw.ai/install/updating)

When adding or moving docs, update this index so references stay findable.
