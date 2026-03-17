# Preflight transcription

Preflight transcription runs **before** mention detection in group chats. It transcribes the first audio attachment so that voice notes can be checked for bot mentions (e.g. @BotName) even when the message has no typed text. Without it, a voice note like "Hey @Claw, what's the weather?" in a mention-gated group would be dropped because there is no text to match the mention regex.

This doc explains how it works, what you need to enable it, and how to implement or troubleshoot it.

---

## When it runs

Preflight transcription runs only when **all** of these are true:

1. **Group (or guild) message** — not a direct message.
2. **Mention required** — the channel/group has `requireMention: true` (or equivalent) so only messages that mention the bot are processed.
3. **Message has audio** — at least one attachment is audio (e.g. voice note).
4. **No typed text** — the message body has no (or only placeholder) text. If the user typed something, mention detection uses that; preflight is only for voice-only messages.
5. **Mention patterns exist** — the channel has mention regexes (e.g. bot username) to match against.
6. **Not opted out** — (Telegram only) `disableAudioPreflight` is not `true` for that group/topic.

If any condition fails, preflight is skipped. On failure (timeout, API error), preflight returns `undefined` and the message is handled with text-only mention detection (voice note may be dropped).

---

## Where it is implemented

| Channel   | Implementation | Audio input to preflight |
|----------|----------------|---------------------------|
| **Telegram** | `src/telegram/bot-message-context.ts` | **MediaPaths** + MediaTypes (local paths to downloaded files) |
| **Discord**  | `src/discord/monitor/preflight-audio.ts` → `message-handler.preflight.ts` | **MediaUrls** + MediaTypes (Discord CDN URLs) |
| Slack, WhatsApp, others | Not implemented | — |

So today only **Telegram** and **Discord** run preflight. Other channels do not; adding it requires wiring the same flow (see [Implementing for another channel](#implementing-for-another-channel)).

---

## Config required

Preflight uses the same audio pipeline as normal media understanding. You must have:

### 1. Audio understanding enabled

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
      },
    },
  },
}
```

If `tools.media.audio` is missing or `enabled: false`, `transcribeFirstAudio` returns `undefined` immediately and preflight does nothing.

### 2. At least one working transcription path

Preflight calls `runAudioTranscription`, which uses `tools.media.audio.models` (or the built-in default list). You need at least one entry that can run:

- **Provider-based:** e.g. `openai`, `deepgram`, `groq`, `google`, `mistral`. Provider must be configured and auth must work (API key or auth profile).
- **CLI-based:** custom `command` entry. The CLI must be on PATH and return transcript on stdout (or a file path when using `--output-dir` for parakeet-mlx).

If no model resolves or all fail (auth, timeout, etc.), preflight returns `undefined`.

### 3. (Telegram) Opt-in per group/topic

Preflight is **on** by default when the conditions above hold. To disable it for a specific Telegram group or topic:

```json5
{
  channels: {
    telegram: {
      groups: {
        "<chatId>": {
          requireMention: true,
          disableAudioPreflight: true,
        },
        "<chatId2>": {
          topics: {
            "<threadId>": { disableAudioPreflight: true },
          },
        },
      },
    },
  },
}
```

Discord does not expose a `disableAudioPreflight` config; the gate is purely the five conditions.

---

## Flow (high level)

1. Inbound message arrives (Telegram or Discord).
2. Channel code detects: group + requireMention + has audio + no typed text + mention regexes (+ Telegram: !disableAudioPreflight).
3. Channel builds a **MsgContext** with either:
   - **MediaPaths** (array of local file paths) and **MediaTypes**, or
   - **MediaUrls** (array of URLs) and **MediaTypes**.
4. Channel calls `transcribeFirstAudio({ ctx, cfg, agentDir })`.
5. **audio-preflight.ts:** Checks `cfg.tools?.media?.audio.enabled`; normalizes attachments from `ctx`; finds first audio attachment; calls `runAudioTranscription`.
6. **audio-transcription-runner.ts:** Uses provider registry and config to run the first successful transcription (provider or CLI).
7. Transcript is returned; channel uses it for mention detection (e.g. regex test). If transcript contains a mention, the message is accepted and the transcript can replace the body placeholder (`<media:audio>`) so the agent sees the text.

---

## What the context must contain

`transcribeFirstAudio` only sees `ctx` (MsgContext). Attachments are normalized by `normalizeAttachments(ctx)` in `attachments.normalize.ts`:

- **Option A — local files:** Set `ctx.MediaPaths` (array of absolute or workspace-relative paths) and `ctx.MediaTypes` (same length). Optional: `ctx.MediaUrls` for the same indices if you have both.
- **Option B — URLs only:** Set `ctx.MediaUrls` (array of URLs) and `ctx.MediaTypes` (same length). The pipeline will fetch the first audio URL.

So for a **new channel** you must either:

- Download the voice note to a temp file and pass **MediaPaths** + MediaTypes, or
- Pass **MediaUrls** (if your platform exposes public or signed URLs to the audio file) + MediaTypes.

The first attachment that is audio (by mime or file extension) is the one transcribed; the rest are ignored for preflight.

---

## Implementing for another channel

If you want preflight transcription on Slack, WhatsApp, or another channel:

1. **Decide when to run:** Same as above: group + requireMention + has audio + no typed text + mention regexes. Add a config flag to disable preflight for that channel if you want (e.g. `channels.slack.disableAudioPreflight`).
2. **Build MsgContext:** Before running mention detection, build a minimal context:
   - **Paths:** If you download attachments, set `MediaPaths: [path1, path2, ...]` and `MediaTypes: [mime1, mime2, ...]`.
   - **URLs:** If you have URLs, set `MediaUrls: [url1, url2, ...]` and `MediaTypes: [mime1, mime2, ...]`.
3. **Call preflight:** `const transcript = await transcribeFirstAudio({ ctx, cfg, agentDir });`
4. **Use transcript for mention check:** If `transcript` is defined, run your mention regex(es) over it. If there is a match, treat the message as mentioned and pass the transcript (or a body that includes it) into the rest of the pipeline so the agent sees the text.
5. **Fallback:** If `transcript` is undefined (disabled, no audio, or transcription failed), fall back to text-only mention detection so behavior is unchanged when preflight is off or fails.

Reference implementations:

- **Telegram (paths):** `src/telegram/bot-message-context.ts` (search for `needsPreflightTranscription` and `transcribeFirstAudio`).
- **Discord (URLs):** `src/discord/monitor/preflight-audio.ts` and `message-handler.preflight.ts` (search for `resolveDiscordPreflightAudioMentionContext` and `transcribeFirstAudio`).

---

## Troubleshooting

### Preflight never runs (transcript always undefined)

- **Audio disabled:** Ensure `tools.media.audio.enabled` is not `false` and `tools.media.audio` exists.
- **Wrong conditions:** Confirm the message is in a group, requireMention is true, there is audio, there is no typed text, and (Telegram) `disableAudioPreflight` is not true for that group/topic.
- **Context missing:** Ensure the channel passes MediaPaths or MediaUrls (and MediaTypes) for the voice note. Check logs with verbose: `OPENCLAW_VERBOSE=1` and look for `audio-preflight:` or `telegram: audio preflight` / `discord: audio preflight`.
- **No models:** Ensure at least one entry in `tools.media.audio.models` (or the default list) has a working provider/CLI and auth. Run normal (non-preflight) audio transcription on the same channel and see if it works; if it does not, fix auth/model config first.

### Preflight runs but transcription fails

- **Auth:** Check provider API keys or auth profiles (e.g. OpenAI, Deepgram). Preflight uses the same credentials as normal audio understanding.
- **Timeout:** Default is 60s (`tools.media.audio.timeoutSeconds`). Long voice notes may need a higher value.
- **Size:** Audio over `tools.media.audio.maxBytes` (default 20MB) is skipped.
- **Tiny files:** Audio under 1024 bytes is skipped (treated as empty/corrupt).
- **URLs (Discord):** Discord attachment URLs may be short-lived or require auth; if they expire before the pipeline fetches, transcription will fail. Use the same URLs quickly or consider downloading to a temp file and using MediaPaths.

### Wrong or empty transcript

- **Language:** Set `tools.media.audio.language` if your voice notes are not in the provider’s default language.
- **Model:** Try a different model in `tools.media.audio.models` (e.g. `gpt-4o-transcribe` instead of `gpt-4o-mini-transcribe` for OpenAI, or `whisper-1` to use the classic Whisper API with the same OpenAI provider and auth).
- **Prompt:** You can set `tools.media.audio.prompt`; default is "Transcribe the audio."

### Telegram-specific

- **Paths:** Telegram must have downloaded the file to a local path; `allMedia[].path` must be set. If your pipeline does not download voice notes before building context, preflight will have no attachment to transcribe.
- **Topic override:** Use `channels.telegram.groups.<chatId>.topics.<threadId>.disableAudioPreflight` to turn preflight off for a single topic.

### Discord-specific

- **Attachment URLs:** Ensure `message.attachments` includes the voice message and each attachment has `url` and `content_type` (e.g. `audio/ogg`). Without URLs, preflight cannot fetch the audio.

---

## Related docs

- [Audio](/nodes/audio) — audio transcription config, providers, mention detection summary.
- [Webhooks](/automation/webhook) — not related to preflight; for external triggers.

---

## Code references

| Piece | Location |
|-------|----------|
| Preflight entry point | `src/media-understanding/audio-preflight.ts` — `transcribeFirstAudio` |
| Transcription runner | `src/media-understanding/audio-transcription-runner.ts` — `runAudioTranscription` |
| Attachment normalization | `src/media-understanding/attachments.normalize.ts` — `normalizeAttachments` (MediaPaths / MediaUrls → MediaAttachment[]) |
| Telegram wiring | `src/telegram/bot-message-context.ts` — `needsPreflightTranscription`, temp context, `transcribeFirstAudio` |
| Discord wiring | `src/discord/monitor/preflight-audio.ts` — `resolveDiscordPreflightAudioMentionContext`; `message-handler.preflight.ts` uses it |
| Config (audio) | `tools.media.audio` in config schema; `src/config/media-audio-field-metadata.ts` |
| Telegram opt-out | `channels.telegram.groups.<id>.disableAudioPreflight`, `channels.telegram.groups.<id>.topics.<id>.disableAudioPreflight` |
