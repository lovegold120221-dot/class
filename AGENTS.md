# AGENTS.md

## Project Type

Pure static HTML/JS/CSS — no build step, no npm. Files are served directly or opened via `file://` protocol.

## Running Locally

Open `auth.html` in Chrome (required for full microphone/Deepgram API support).

## Key Files

| File | Purpose |
|------|---------|
| `auth.html` | Entry point; sets `sessionStorage.role` + `sessionStorage.name`, redirects to `index.html` |
| `index.html` | Main classroom; `SuccesApp` class (~line 839) handles both teacher/student views |
| `teacher-agent.html` | Independent Moonshot AI content generator |
| `*-history.html` | Read-only history viewers |
| `sw.js` | Service worker; **bump `CACHE_NAME`** after any HTML/asset change to force cache refresh |

## Architecture Notes

- **Firebase** (asia-southeast1): single source of truth, paths under `chats/{roomCode}/`
- **Speech pipeline**: Deepgram STT → Firebase → Google Translate → Cartesia TTS
- **Voice config**: `CARTESIA_VOICES` object maps BCP-47 codes to voice IDs
- **Auth flow**: Teacher/student role + name stored in `sessionStorage` (not persisted)

## Common Mistakes

- **Forgetting to bump CACHE_NAME** in `sw.js` — changes won't propagate to returning PWA users
- **Editing the wrong HTML file** — there are similar files (`index.html`, `index-claude.html`, `development.html`); main classroom is `index.html`
- **Testing without Chrome** — Deepgram/microphone APIs require Chrome; other browsers get Web Speech API fallback

## Subdirectories

- `admin/` — admin dashboard (user/role/room management)
- `dashboard/` — teacher monitoring dashboard (conversations, billing with Eburon AI)
- `reference/` — reference materials
