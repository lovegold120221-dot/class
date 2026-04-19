# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Succes Class** is a real-time multilingual classroom platform. Teachers broadcast speech/text; students receive translated audio in their native language. No build step — all pages are plain HTML/JS/Tailwind served as static files.

## Running Locally

Open `auth.html` directly in a browser. No server, no build, no npm install. Chrome is recommended for full Deepgram/mic API support.

## Architecture

The app is a collection of standalone HTML pages, each self-contained with embedded `<script>` tags. The core classroom logic lives entirely in `index.html` inside the `SuccesApp` class (~line 839).

**Page roles:**
- `auth.html` — entry point; sets role + name in `sessionStorage`, redirects to `index.html`
- `index.html` — main classroom (teacher & student views share one class, branching on `sessionStorage.role`)
- `teacher-agent.html` — Moonshot LLM AI teacher content generator (independent feature)
- `*-history.html` — read-only conversation history viewers pulling from Firebase

**Data flow:**
1. Teacher speaks → Deepgram STT → Firebase `questions/`
2. Firebase listener on student side → Google Translate → Cartesia TTS → audio playback
3. Student speaks → Deepgram STT → Firebase `studentResponses/` (pending teacher approval)
4. Teacher approves → translated text delivered to class

**Firebase Realtime Database** (asia-southeast1) is the single source of truth for all classroom state. Paths are scoped under `chats/{roomCode}/`.

**External services (all keys embedded in HTML):**
- Deepgram — speech-to-text (with Web Speech API fallback)
- Cartesia AI Sonic-3 — text-to-speech, 29+ language voices
- Google Translate free API — text translation
- Moonshot AI — AI teacher agent only (`teacher-agent.html`)

## Key Patterns

- **`SuccesApp` class** (`index.html:839`) — constructor wires Firebase listeners; `init()` routes to `initTeacher()` or `initStudent()` based on `sessionStorage.role`
- **`translateText(text, source, target)`** — calls Google Translate free endpoint
- **`processAudioQueue()`** — Cartesia TTS, queues sequentially to avoid overlapping playback
- **`CARTESIA_VOICES`** config object — maps BCP-47 language codes to Cartesia voice IDs

## Service Worker

`sw.js` uses a network-first strategy. After any significant HTML change, the `CACHE_NAME` version string in `sw.js` should be bumped so clients pick up the new assets.
