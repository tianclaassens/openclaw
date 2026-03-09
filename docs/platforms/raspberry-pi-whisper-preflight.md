# Pi (Docker): Whisper API + preflight transcription checklist

Use this when OpenClaw runs in Docker on a Raspberry Pi and you want **preflight transcription** (voice notes in Telegram/Discord groups with `requireMention`) to work using the **Whisper API**.

---

## Current state (from Pi check)

- **Gateway:** Running in Docker (`openclaw-openclaw-gateway-1`), image `tianclaassens/openclaw:whisper_fix_from_main`.
- **Config dir:** `/home/pi/.openclaw` on host, mounted as `/home/node/.openclaw` in the container.
- **Config:** `tools.media.audio.enabled: true`, `language: "en"`, `echoTranscript: true`. **No** `tools.media.audio.models` (so defaults apply).
- **Telegram:** Enabled, `groups.*.requireMention: true` — preflight conditions are met for group voice notes.
- **OpenAI:** `agents.defaults.model.primary` is `openai/gpt-4o`. `OPENAI_API_KEY` is set in `/home/pi/.openclaw/.env` on the host.
- **Compose:** Does **not** pass `OPENAI_API_KEY` (or other API keys) via `environment:` or `env_file:`; the container only sees vars explicitly listed in `docker-compose.yml`. The gateway process loads dotenv from the **mounted** config dir (`/home/node/.openclaw/.env`) at runtime, so the key may still be available inside the Node process. If preflight fails with auth errors, add the key to the container env (see below).

---

## Checklist: what to do

### 1. Ensure audio is enabled and Whisper API is selected (config)

In `~/.openclaw/openclaw.json` (on the Pi, so the mounted volume has it):

- `tools.media.audio.enabled` should be `true` (already is).
- To use the **Whisper API** explicitly, add a `models` array under `tools.media.audio`:

```json
"tools": {
  "media": {
    "audio": {
      "enabled": true,
      "language": "en",
      "echoTranscript": true,
      "models": [
        { "provider": "openai", "model": "whisper-1" }
      ]
    }
  }
}
```

If you omit `models`, the default is still OpenAI (`gpt-4o-mini-transcribe`); both use the same API key. Adding `whisper-1` just forces the classic Whisper model.

### 2. Ensure the gateway process has OpenAI API key

- **Option A (current):** Rely on dotenv. The gateway loads `~/.openclaw/.env` from the mounted volume (`/home/node/.openclaw/.env`). Ensure `OPENAI_API_KEY` is set in `/home/pi/.openclaw/.env` on the host. No compose change.
- **Option B (explicit):** Pass the key into the container so it’s guaranteed. In `docker-compose.yml`, under `openclaw-gateway` → `environment:`, add:

  ```yaml
  OPENAI_API_KEY: ${OPENAI_API_KEY}
  ```

  Then ensure the host env or the env file used by `docker compose` (e.g. in the same directory as the compose file) exports `OPENAI_API_KEY`. Alternatively use `env_file: - /home/pi/.openclaw/.env` (adjust path if your config dir is elsewhere). Restart the stack after changing compose.

### 3. Restart the gateway after config changes

```bash
cd /home/pi/openclaw   # or wherever your compose lives
docker compose restart openclaw-gateway
```

### 4. Verify preflight (optional)

- Send a **voice note** (no typed text) in a Telegram group that has `requireMention: true`, and mention the bot in the voice (e.g. “Hey @YourBot, what’s the weather?”).
- If preflight works: the message is transcribed, the mention is detected, and the agent replies (and with `echoTranscript: true` you may see the transcript echoed).
- If nothing happens or the message is dropped: check gateway logs for `audio-preflight` or `telegram: audio preflight` (run with `OPENCLAW_VERBOSE=1` in the container if needed). Auth errors mean the container doesn’t have a valid `OPENAI_API_KEY`; fix with step 2.

### 5. (Optional) Disable preflight for a specific group/topic

If you want to turn off preflight for one Telegram group or topic:

```json
"channels": {
  "telegram": {
    "groups": {
      "<chatId>": {
        "requireMention": true,
        "disableAudioPreflight": true
      }
    }
  }
}
```

---

## Summary

| Item | Status / action |
|------|------------------|
| `tools.media.audio.enabled` | Already `true` |
| `tools.media.audio.models` | Optional; add `[{ "provider": "openai", "model": "whisper-1" }]` to use Whisper API explicitly |
| OpenAI API key in container | In host `~/.openclaw/.env`; loaded from mounted volume at runtime. If transcription fails with auth, add `OPENAI_API_KEY` to compose `environment:` or use `env_file:` |
| Telegram `requireMention` | Already set; preflight runs when conditions match |
| Restart after config | `docker compose restart openclaw-gateway` |

See [Preflight transcription](/nodes/preflight-transcription) and [Audio](/nodes/audio) for full reference.
