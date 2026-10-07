# NOVA

**Your AI companion.**

NOVA is a local-first desktop AI companion designed around a small animated pet, voice control, safe computer actions, and persistent personal memory. The project is intentionally free-first: Ollama is the default brain, macOS provides a zero-cost text-to-speech fallback, and online providers are optional.

## Implemented conversation, memory and computer-use foundations

The current increment turns the Phase 1 foundation into a real conversation path:

- FastAPI backend with live, readiness, configuration, and streaming WebSocket endpoints
- React + TypeScript + Vite floating-companion shell
- State-based pet animation surface for idle/listening/thinking/speaking/working/success/error states
- Async Ollama NDJSON streaming with model-aware health checks
- Optional OpenRouter SSE streaming fallback
- Local-first provider routing with `local_only`, `local_first`, `smart`, and `online_only` policies
- Bounded in-memory conversation context and centralized NOVA system prompt
- Request IDs, lifecycle logging, cancellation, normalized provider errors, and streamed response bubbles
- Environment-driven configuration with secrets kept backend-only
- Mocked provider, routing, agent, WebSocket, parser, and cancellation tests
- Setup scripts that do not overwrite an existing `.env`
- Hold-to-talk browser microphone capture and focused Option+Space shortcut
- Mono 16 kHz PCM WAV, local whisper.cpp STT, English/Hindi language selection
- Native macOS `say` sentence playback overlapping LLM generation
- Answer-language-aware installed voice selection and speech-only emoji/formatting cleanup
- Speech interruption, mic privacy feedback, transcript preview, waveform and voice settings
- Cmd+Shift+D voice diagnostics (no raw audio)
- SQLite chat history, local search, conversation restoration and New Chat
- Separate long-term memory with local remember/forget commands and relevance-ranked context
- Compact History/Memory panels, local exports and confirmed data clearing
- Structured computer-task planning, independently guarded tools, real macOS/Chrome actions and readback verification
- Compact Computer settings, task progress/confirmation, shared voice controls, Escape stop and local task summaries
- Feature-flagged Connections hub with provider metadata, capability/status cards and a backend-only secure credential boundary
- Compact native pet-first mode with automatic task-mode expansion and local “Hey Nova” / “Hello Nova” phrase matching
- Live model registry with Ollama discovery, Gemini model listing, OpenRouter pricing/catalog discovery, capability-aware routing, rolling latency preference and zero-budget enforcement

Phase 3 voice acceptance is complete on this Mac: the user confirmed physical-microphone Hindi/Hinglish speech, emoji silence, and STOP on 2026-10-05. Persistent chat history and long-term memory are also complete, with actual restart and physical-microphone acceptance. Phase 5 computer use now has real native/browser/file/clipboard actions, an LLM-driven observe/act/verify loop, independent permissions and shared voice task controls. The connector layer now includes secure Google OAuth/read-only Gmail, Calendar and Drive read tools, guarded task tools, explicit MCP stdio management and the Connections hub. **308 backend tests and 4 frontend tests pass. Final physical-microphone computer-task acceptance is pending.** Native Tauri packaging, bundled local runtime resources and global microphone shortcuts are implemented; packaged-runtime smoke verification passes.

### Multi-model routing

Open **AI Models** from the companion panel to inspect live model metadata and
change routing policy. NOVA discovers installed Ollama models without
downloading them, lists Gemini models only when a backend-only `GEMINI_API_KEY`
is configured, and refreshes the OpenRouter model catalog when an API key is
configured. Capability fields remain unknown when provider metadata does not
prove them. `FREE_ONLY=true` and `ZERO_BUDGET_MODE=true` are the defaults;
unknown or paid cloud pricing is blocked, and the local Ollama model is the
safe fallback. Gemini pricing is intentionally `UNKNOWN` unless current
provider/account metadata establishes eligibility, so it is never silently
used as a free model.

## Computer use

Open **Computer** to enable task execution. Defaults are disabled, simulation on, and semi-autonomous mode. Real mouse/keyboard and screenshots require macOS Accessibility/Screen Recording grants, shown in the permission panel. Browser actions use a visible dedicated Chrome profile. Try “Open YouTube,” “Create a folder called NOVA-Test on my Desktop,” or a research-and-save goal. Required approvals are independent of the model; Pause, Continue, Stop and Escape control the task.

See [COMPUTER_USE.md](COMPUTER_USE.md) for setup, settings, tool boundaries, task history, APIs, real acceptance evidence and current limits.

## Connections

Open **Connections** from the companion footer or native title bar. Google can
be configured for read-only Gmail OAuth, with optional Calendar and Drive
read-only scopes. GitHub now has an official OAuth/read adapter for profile,
repositories, search, files, issues, pull requests and branches; write/delete
operations are not exposed. Notion now has an official read-only integration
adapter for search, page/database information and block-content inspection.
The other planned services remain honestly marked
unavailable until their provider adapters exist. Connector
credentials stay in the backend macOS Keychain boundary. Set
`CONNECTORS_ENABLED=true` and configure the provider OAuth values before using
the account flow. Read-only Gmail, Calendar, Drive and GitHub tasks use the
same guarded task planner and never send, draft, delete, upload or modify
account data.

MCP is separately controlled by `MCP_ENABLED`. Registering a server, enabling
it, discovering tools and executing a tool are all explicit operations; NOVA
does not auto-trust arbitrary MCP processes.

## Persistent history and memory

Chats and memories are saved separately in `backend/data/nova.db`, initialized automatically. Open **History** for search, reopening, rename, archive/restore and confirmed deletion; **New Chat** starts fresh context while keeping saved memories. Open **Memory** (also available in Voice settings) to inspect/edit facts, configure extraction, export JSON/Markdown or confirm clearing.

Try “Remember that I prefer Hinglish,” “Remember I use Premiere Pro,” “Forget my preference about response language,” or “What do you remember about me?” These commands execute locally. Clearing all memories asks for confirmation and preserves chats. Temporary sentences such as “Tomorrow I need to edit a video” stay chat history, not permanent memory. Credentials/private reasoning are filtered at the persistence boundary.

See [PERSISTENCE.md](PERSISTENCE.md) for schema, settings, APIs, architecture, restart evidence, performance measurements and limitations. Defaults are `SAVE_CHAT_HISTORY=true`, `MEMORY_ENABLED=true`, `MEMORY_AUTO_EXTRACTION=true`, `MEMORY_MIN_CONFIDENCE=0.75`, `MEMORY_TOP_K=5`; Memory settings persist in SQLite.

## Requirements

Detected on the development Mac:

- macOS on Apple Silicon
- Python 3.11 or newer (Python 3.14 is supported by the current environment)
- Node.js 20 or newer
- npm 10 or newer
- Xcode command-line tools for the on-demand native computer-use helper

Optional for the current phase:

- [Ollama](https://ollama.com/) for the local model
- FFmpeg for the local speech pipeline
- Rust and Tauri prerequisites for the packaged desktop build

## Quick start

```bash
cd /Users/parvez/NOVA
./setup.sh
```

The setup script creates `backend/.venv`, installs backend dependencies, installs frontend dependencies, and checks optional local binaries. It will never replace an existing `.env`.

Start the services in two terminals:

```bash
# Terminal 1
cd /Users/parvez/NOVA/backend
source .venv/bin/activate
uvicorn app.main:app --reload --host 127.0.0.1 --port 8742

# Terminal 2
cd /Users/parvez/NOVA/frontend
npm run dev
```

Open <http://localhost:5173>. The backend health endpoint is <http://127.0.0.1:8742/api/health>.

For a convenience launcher after setup:

```bash
./setup.sh --start
```

## Phase 2 setup: Ollama

Ollama is the default local brain. Install it with Homebrew, start the server, and explicitly download the configured model:

```bash
brew install --cask ollama
ollama serve
ollama pull qwen3:4b-q4_K_M
```

The provider and model can be changed in `.env`. NOVA still starts when Ollama, its server, or the configured model is unavailable; `/api/health` reports the exact condition and the UI shows a structured error when a request cannot be served.

The setup script never downloads a model automatically. It reports:

- whether the Ollama binary is installed
- whether the local server is reachable
- whether the configured hybrid `qwen3:4b-q4_K_M` is available

## Phase 2 configuration

```dotenv
OLLAMA_ENABLED=true
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL=qwen3:4b-q4_K_M
OPENROUTER_ENABLED=true
OPENROUTER_API_KEY=
OPENROUTER_MODEL=openrouter/free
LLM_ROUTING_POLICY=local_first
LLM_REQUEST_TIMEOUT=60
LLM_MAX_TOKENS=512
LLM_SIMPLE_MAX_TOKENS=256
LLM_COMPLEX_MAX_TOKENS=1024
LLM_TEMPERATURE=0.7
LLM_THINKING_MODE=smart
LLM_THINKING_DEFAULT=false
CONVERSATION_HISTORY_LIMIT=12
```

`local_first` checks Ollama first and uses configured OpenRouter only when the local provider is unavailable. `local_only` never uses the online fallback. `online_only` selects OpenRouter, and `smart` prefers OpenRouter for requests explicitly marked `complex`.

Qwen3 thinking is controlled separately from provider routing:

| Thinking mode | Ollama behavior |
| --- | --- |
| `off` | Every request sends `think:false`, with the simple budget |
| `on` | Every request sends `think:true`, with the complex budget |
| `smart` (default) | Lightweight deterministic heuristics: analysis/debugging/project planning/algorithm complexity uses `think:true`; ordinary conversation stays fast |

Simple greetings, jokes, arithmetic and short assistant requests use `LLM_SIMPLE_MAX_TOKENS=256`. Ambiguous requests use `LLM_THINKING_DEFAULT=false` and `LLM_MAX_TOKENS=512`. Thinking requests use `LLM_COMPLEX_MAX_TOKENS=1024`. No extra LLM or network classification call is made. Explicit `mode:"complex"` is also supported by the existing WebSocket request.

**Use a hybrid model.** On this Mac, the moving `qwen3:4b` tag resolved to Thinking-2507, which advertises only `thinking.values:[true]`. Its `think:false` request can emit reasoning as ordinary content. The explicitly configured `qwen3:4b-q4_K_M` advertises `[false,true]` and genuinely supports both modes. NOVA checks cached `/api/show` metadata and rejects a known-incompatible mode with `LLM_THINKING_UNSUPPORTED` before content leaks. The previous model remains installed; Ollama itself is unchanged.

Separate Ollama `message.thinking` fields are never forwarded. An incremental filter also removes inline `<think>…</think>` blocks even if tags are split across tokens. Only final answer content goes to the pet bubble, speech queue and conversation history. Plain answer chunks stream immediately.

Ollama's `num_predict` is a **combined generation budget**, not a guaranteed final-answer token allocation when thinking is on. Complex requests can still consume it in reasoning; that returns a structured error, never an empty successful response. Fast requests avoid this failure by disabling reasoning on a compatible model. OpenRouter retains its existing API behavior; this switch controls local Ollama, not arbitrary remote model reasoning controls.

### Latency diagnostics

Cmd+Shift+D shows provider/model, thinking decision, **TTFT** (first non-empty visible answer chunk), generation latency and answer chunk count. Timing is measured from entering the provider stream, including request/metadata overhead, to first answer and stream completion; provider routing, STT and speaker playback are separate. Metrics are carried on existing `llm.completed` events and displayed only in the debug panel. `token_count` is explicitly tagged `visible_content_chunks`: it counts streamed answer chunks, not reasoning tokens or a fabricated tokenizer count.

Warm same-model measurements on this Mac (same system prompt; prior omitted `think` versus optimized smart `think:false`):

| Prompt | Before TTFT | After TTFT |
| --- | ---: | ---: |
| Hey Nova, who are you? | 4,242.74 ms | 106.81 ms |
| Tell me a joke. | 2,672.01 ms | 106.77 ms |
| What is 2 + 2? | 3,152.71 ms | 96.13 ms |

These are measured samples, not latency guarantees; cold model loading, context size and machine load change timings. The previous Thinking-only tag also reproduced the empty-answer failure after 13.46 seconds with a 512-token budget. Increasing its budget alone did not solve the underlying mode incompatibility.

## OpenRouter setup

OpenRouter is an optional fallback. Put the key only in the backend `.env`; it is never sent to the frontend bundle:

```dotenv
OPENROUTER_ENABLED=true
OPENROUTER_API_KEY=your-key-here
OPENROUTER_MODEL=openrouter/free
```

The provider abstraction is designed so free model availability can change without changing the agent or UI.

## Gemini Free Tier setup

Gemini cloud routing remains protected by `FREE_ONLY=true` and
`ZERO_BUDGET_MODE=true`. NOVA uses the updateable
`GEMINI_FREE_MODEL_ALLOWLIST` only when the exact IDs are also returned by the
live Gemini model-discovery endpoint. The current documented IDs are:

```dotenv
GEMINI_ENABLED=true
GEMINI_API_KEY=your-key-here
GEMINI_MODEL=
GEMINI_FREE_MODEL_ALLOWLIST=gemini-3.8-flash,gemini-3.5-flash,gemini-3.5-flash-lite,gemini-3.1-flash-lite,gemini-2.5-flash,gemini-2.5-flash-lite
```

The allowlist is based on Google’s official pricing documentation:
<https://ai.google.dev/gemini-api/docs/pricing>. Explicit pricing returned by
provider metadata overrides the allowlist; paid and unknown models remain
blocked. The exact list is intentionally an environment setting so it can be
updated when Google changes model availability or Free Tier pricing.

## Phase 3 setup: local voice

On this Mac, FFmpeg, whisper.cpp 1.9.1 (Metal), `say`, and the authorized multilingual **Small** model are installed. The model is at `backend/data/models/ggml-small.bin`, outside source control. NOVA uses the existing whisper.cpp binary instead of adding MLX/Python audio dependencies to Python 3.14.

On another Mac, install only missing binaries:

```bash
brew install whisper-cpp ffmpeg
```

Download a model only when you choose to. Example, multilingual Small (~466 MB):

```bash
mkdir -p backend/data/models
curl -fL --retry 2 -o backend/data/models/ggml-small.bin \
  https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-small.bin
```

Tiny (~75 MB), Base (~142 MB), Small, and Medium are supported locally. The Voice settings provide explicit `FAST` (Base), `BALANCED` (Small) and `ACCURATE` (Medium) STT profiles. Use multilingual models, not `.en`, for Hindi/Hinglish. No model downloads happen at startup or when changing settings. Custom existing models can use `WHISPER_MODEL_PATH`.

### Voice configuration

```dotenv
VOICE_ENABLED=true
STT_ENABLED=true
TTS_ENABLED=true
WHISPER_PROVIDER=local
WHISPER_MODEL=small
WHISPER_MODEL_PATH=
WHISPER_BINARY=whisper-cli
WHISPER_LANGUAGE=auto
STT_TIMEOUT=120
TTS_PROVIDER=macos
TTS_VOICE=
TTS_LANGUAGE_MODE=auto
TTS_VOICE_ENGLISH=
TTS_VOICE_HINDI=
TTS_VOICE_FALLBACK=
TTS_RATE=180
VOICE_AUTO_STOP=false
VOICE_SILENCE_TIMEOUT_MS=1200
VOICE_MAX_RECORDING_SECONDS=60
VOICE_SHORTCUT=Alt+Space
```

Native `say` both synthesizes and plays on this Mac's selected audio output. Voice names are discovered from `say -v '?'`; NOVA always passes an explicit suitable voice and never delegates to an unknown macOS default. Blank voice preferences discover installed voices. Legacy `TTS_VOICE` is honored only when its installed locale matches the answer; the separate English/Hindi settings take precedence. Legacy `VOICE_NAME`/`VOICE_SPEED` remain retained; use the TTS fields for Phase 3.

`TTS_LANGUAGE_MODE` accepts `auto`, `english`, `hindi`, or `hinglish`. Auto resolves the **cleaned spoken answer segment**, independently of the microphone language, so speech can start before the full answer finishes. Devanagari prose uses Hindi even with app names like “Chrome”; romanized Hindi uses lightweight Hinglish heuristics, with uncertainty reported in diagnostics. English uses an installed English voice. Hindi uses an installed Hindi voice, then a configured English fallback, then another installed English voice. Invalid or Chinese voice preferences cannot become an accidental fallback. If no suitable voice is installed, text continues silently with a recoverable speech error.

Hinglish prefers an installed Indian-English voice by default; `TTS_VOICE_ENGLISH` can select another installed English voice. Set spoken-answer language to **Hindi** to try a Hindi voice for romanized text. On this Mac, installed choices include Lekha (`hi_IN`), Rishi (`en_IN`), and Samantha (`en_US`); another Mac may have different voices. Voice discovery never downloads a voice.

Speech-only cleanup removes emoji sequences, visual Markdown, private reasoning/tagged tool metadata, and raw JSON. Fenced code is skipped across token boundaries with “I've displayed the code on screen.” URLs become “the link” unless the request explicitly asks to read/spell the URL. Emoji-only answers invoke no `say` process. For example, “Done bhai 😂. I opened the page 🔥.” speaks “Done bhai.” followed by “I opened the page.” The visible response and conversation history retain their answer content.

Cmd+Shift+D shows separate `stt_language`, detected answer language, language mode, selected voice/locale, fallback status, heuristic uncertainty, and original/cleaned speech lengths. `speech_chars_removed` is the nonnegative **net length reduction**, including visual cleanup and whitespace; replacement phrases can increase speech length. Final cleanup totals arrive even for emoji-only answers. These values are in-memory diagnostics, not added to response bubbles.

Voice settings are validated by `/api/voice`, apply immediately, and are remembered in browser-local storage. `.env` defines startup defaults; chat history and separate Memory settings use SQLite. Raw recordings are never retained: temporary WAV/JSON files are private and removed after transcription or cancellation, and microphone contents are not written to logs.

### Using voice

1. Launch `./setup.sh --start`, open <http://localhost:5173>, and allow microphone access.
2. **Hold** the mic button, or hold **Option+Space while NOVA is focused**.
3. Speak; release to send. No audio is captured while idle.
4. “MIC LISTENING” changes to “TRANSCRIBING LOCALLY,” then streamed response and speech.
5. Press STOP to cancel capture, STT, generation, and current/queued speech. Starting another push-to-talk interrupts the previous request.
6. “Nova stop” is recognized after a held recording is transcribed. Native packaged builds also provide an explicitly enabled local “Hey Nova” / “Hello Nova” wake mode; it remains off by default. NOVA gives a brief local “Yeah?” acknowledgement before accepting the command. Start recording to interrupt speech immediately.

The browser shortcut is **not global**. The packaged native desktop app provides the configured global shortcut; browser development remains focused-only. Option+V is an alternative focused shortcut in settings. Optional energy-based silence stop is off by default and activates only after speech energy is detected; a 60-second hard cap always applies.

Short code-mixed utterances can be misidentified as English; choose **Hindi / Hinglish** in Voice settings if “Chrome kholo” is misheard. NOVA preserves recognized language and raw/normalized transcripts. It does not invent a confidence score or silently rewrite uncertain words.

### Microphone permissions

Allow microphone access for the localhost page in Chrome/Safari site permissions. On macOS: **System Settings → Privacy & Security → Microphone → enable your browser**. Reload the page after changing permissions. Device labels may be hidden until permission is granted. Phase 3 needs no Accessibility or Automation permission.

### Voice troubleshooting

- `WHISPER_MODEL_MISSING`: put the chosen GGML model under `backend/data/models`, or set `WHISPER_MODEL_PATH`.
- `WHISPER_UNAVAILABLE`: install whisper.cpp; configure `WHISPER_BINARY` if not on PATH.
- `MIC_PERMISSION_DENIED`: allow access in the browser and macOS. Text remains available.
- `EMPTY_TRANSCRIPT`: hold while speaking, check the input device, and try language selection.
- `TTS_UNAVAILABLE` / `TTS_PROCESS_FAILED` / `TTS_LANGUAGE_UNAVAILABLE`: choose a discovered English/Hindi voice in Voice settings; text continues silently.
- Complex qwen3 responses use `LLM_COMPLEX_MAX_TOKENS`; simple requests use `LLM_SIMPLE_MAX_TOKENS` with thinking disabled. Errors remain visible rather than reporting a fake answer.
- STT and TTS can each be disabled independently. Neither failure prevents startup or text conversation.
- If you configure OpenRouter fallback, **only the transcript/conversation text** may be sent there. Audio always stays in the local browser/backend/Whisper pipeline. Use `LLM_ROUTING_POLICY=local_only` for fully offline text reasoning.

## Architecture

Read [ARCHITECTURE.md](ARCHITECTURE.md) for the boundary map and planned build order. Read [TOOLS.md](TOOLS.md) for the permission-first tool contract.

## Tests

```bash
cd backend
source .venv/bin/activate
pytest
```

No test requires Ollama, OpenRouter, a microphone, or a paid API.

Frontend voice tests use Node's built-in runner (Node 22.18+ for TypeScript stripping), without a new testing framework:

```bash
cd frontend
npm test
npm run typecheck
npm run build
```

Manual physical-mic check: hold and say “Hey Nova, tell me a joke,” release, verify local transcription and speech, press STOP mid-speech, then try again. Also test Hindi/Hinglish, denied permission, STT disabled and TTS disabled. A synthetic fixture integration test is not a substitute for mic acoustics and real permission acceptance.

For Hindi/emoji acceptance, leave spoken-answer language on **Auto**, ask for “आप कैसे हैं?” and “Main bilkul theek hoon.”, then ask for “Done bhai 😂🔥”. Verify Hindi/Indian-English pronunciation is sensible, emoji names are silent, and STOP cuts off playback. Open diagnostics to confirm the selected locale. The user confirmed all these physical-microphone/listening checks on 2026-10-05.

Verified locally on 2026-10-05:

- 300 backend tests and 4 frontend audio tests pass; Python compilation, TypeScript checks, production build and launcher syntax checks pass.
- Real browser/Ollama/native `say` requests selected Lekha (`hi_IN`) for Hindi and Rishi (`en_IN`) for romanized Hinglish, while preserving visible emojis.
- A Chinese English-voice/fallback preference safely selected Samantha (`en_US`) with `fallback_used:true`; a forced Hindi override selected Lekha for romanized text.
- An emoji-only answer remained visible, reported zero cleaned speech length, and launched no speech process.
- Sentence speech overlapped live generation; STOP terminated/reaped the native process in a measured 73 ms (one sample, not a timing guarantee).
- Browser fixture capture passed through real local Whisper and Ollama to native speech, with separate STT/answer/voice diagnostics. Physical microphone and pronunciation acceptance was separately confirmed by the user.

## Security principles

- API keys stay in the backend environment.
- No microphone recording is continuous by default.
- Model output is untrusted input to the guarded computer-task executor.
- Destructive or shell-like actions require an explicit permission layer.
- Sensitive values are not eligible for automatic memory extraction.
- The app reports unavailable capabilities instead of fabricating success.

## Troubleshooting

### The pet says the backend is offline

Start FastAPI and confirm `http://127.0.0.1:8742/api/health` returns JSON. If the frontend is served from a different origin, add it to `CORS_ORIGINS`.

### Ollama is unavailable

That is a supported runtime condition. NOVA still starts and exposes server/model availability under `llm.ollama` in its health response. Start Ollama and pull the configured model when local generation is desired, or configure a valid OpenRouter key for fallback.

### Native desktop packaging

The native Apple Silicon macOS build uses Tauri for the floating companion shell and bundles the PyInstaller backend, local Whisper runtime/model, native computer-use helper and local wake/microphone helper. Tauri/Rust is deliberately not required for the web development loop.

Install Rust once, then build the app and local DMG:

```bash
brew install rustup
rustup-init
cd /Users/parvez/NOVA
./setup.sh
./scripts/build_desktop.sh
```

The outputs are `src-tauri/target/release/bundle/macos/NOVA.app` and `src-tauri/target/release/bundle/dmg/NOVA_0.2.0_aarch64.dmg`. The build never includes `.env`, the SQLite database or user runtime data. Run `backend/.venv/bin/python scripts/native_smoke.py` after building for an authenticated local-runtime, persistence, bundled-STT and simulation-task check.

Verified on 2026-10-06: the full desktop build completed, the app passed strict ad-hoc signature validation, the DMG passed `hdiutil verify`, `NovaVoice --self-test` passed, and the packaged smoke passed authenticated HTTP/WebSocket, local Ollama simulation, persistence, bundled STT discovery and the global-stop endpoint.
