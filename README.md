# LE Karaoke Studio

**A local GPU workstation for karaoke burn-in, music visualizers, and automated short-form video — from raw media to styled MP4 to multi-platform posts.**

LE Karaoke Studio turns video or audio into **word-timed karaoke subtitles** burned into the frame, with optional **full-screen music visualizers**, **tactical VFX overlays**, and **yt-dlp ingestion** — all in one React dashboard backed by a FastAPI server. A second mode, **Automate**, polls news and OSINT sources, transcribes clips, classifies them with LLMs, renders ticker-style shorts, and posts to **X, Telegram, and Rumble** on a schedule — with human approval when you want it.

Runs on **your machine** (GPU Whisper, Remotion or HyperFrames render). Not cloud SaaS. Not WordPress.

**Made by [Logic Encoder](https://logicencoder.com)**

Private source: [logicencoder/le-karaoke-studio](https://github.com/logicencoder/le-karaoke-studio)

---

## What problems it solves

| Pain | How LE Karaoke Studio handles it |
|------|--------------------------------|
| Manual lyric timing in an NLE | Whisper word timestamps → automatic phrase lines |
| Inconsistent subtitle look across clips | 12+ subtitle presets, 40+ visualizer modes, saved studio presets |
| Tool-hopping (download → transcribe → render → post) | One dashboard: DLV → Studio → Automate |
| Noisy vocals / mixed stems | UVR tab separates vocals before transcription |
| Repetitive breaking-news clip workflow | Automate scraper + LLM categorize + FFmpeg render + distribute queue |
| Preview before expensive render | AFX and VFX labs tune look against real audio before GO |

---

## Two products in one app

### Studio — operator-driven karaoke pipeline

You upload (or import from DLV), tune style in AFX/VFX labs, hit **GO**, and download `output.mp4`. Full control over Whisper, phrases, colors, visualizers, trim, translation, and encoder.

### Automate — hands-off discovery and distribution

Background scraper watches YouTube channels, Telegram, VK, RSS, and X. New videos flow through transcribe → OpenRouter categorization → FFmpeg ASS karaoke + news ticker → optional auto-post. **Test Lab** lets you upload one file, approve/reject, and verify before enabling the scraper.

---

## Example use cases

### Karaoke lyric video for a Slovak track

You have an MP3 of an original song. **DLV** or local upload → **Studio** → set Whisper language to `sk`, pick **karaoke_gold** preset in **AFX**, enable beat sync, **GO**. You get an MP4 with word-highlighted lyrics timed to the vocal — ready for YouTube or Telegram without After Effects keyframing.

### Automated breaking-news shorts

You enable two YouTube OSINT channels in **Automate → Sources**, set categories and X/Telegram routes in **Distribute**, turn on **manual approve** in Test Lab first. The scraper pulls a 45 s clip, transcribes, LLM labels it `geopolitics`, renders ticker + subtitles, you **Approve** in Test Lab, and it posts on your schedule within the hourly rate limit.

---

## Studio pipeline — step by step

When you press **GO** on a job, the server runs these stages (`jobs/{id}/` on disk):

| Step | What happens |
|------|----------------|
| **Upload** | Video or audio saved as `input.*` |
| **Analyze format** | ffprobe metadata; audio-only → synthetic video track; optional AC3 normalize; optional **yadif deinterlace**; optional **trim** region |
| **Whisper transcription** | Word-level timestamps → `words.json`; CUDA when available; optional **pyannote speaker diarization**; optional **DeepL translation** after transcribe |
| **Phrase grouping** | Words grouped into readable lines → `phrases.json`; max words/lines, line gap, delay ms, ALL CAPS option |
| **Render** | **Remotion** (default) or **HyperFrames** → `output.mp4`; beat sync from librosa analysis; optional NVENC/QSV/HEVC re-encode; optional audio passthrough mux |
| **Complete** | Preview and download via API |

**Per-job artifacts:** `state.json`, `words.json`, `phrases.json`, `props.json`, `audio_analysis.json`, `render.log`, `pipeline.log`, `timing.json`, `output.mp4`.

**Live progress:** WebSocket `/ws/log` pushes log lines plus `__STATE__` job JSON, `__ACTIVE__` flag, and `__EXIT__` on finish. Dashboard shows step bar, live subtitle preview, and render log tail.

**Concurrency:** one active Studio pipeline at a time (409 if busy); **GO BATCH** queues jobs sequentially.

**Recovery:** jobs stuck in `running` after server restart are marked error automatically.

---

## Dashboard — all tabs

Ten top-level tabs. Desktop sidebar + mobile-friendly layout. API on port **8765**, dashboard on **5175** (configurable via env).

### Studio

The main production tab.

**Upload:** drag-and-drop or file picker — video and audio extensions; single or **batch upload**.

**Job queue:** chips for recent jobs; open, cancel, retry, delete, clone; hidden-job cleanup.

**Metadata card:** container, duration, resolution, FPS, codecs, file size after probe.

**Trim timeline:** set `trim_start` / `trim_end` with preview when source is video.

**Whisper settings:** language, model (`tiny` → `large-v3`), device auto/CPU/CUDA, initial prompt, beam size, temperature.

**Phrase settings:** max words per line, max lines, line gap, subtitle delay ms, ALL CAPS toggle.

**Translation:** optional post-transcribe target language via DeepL (`en`, `sk`, `cs`, `de`, `uk`, `ru`, `fr`, `es`, `pl`, …).

**Subtitle styling:** preset picker, highlight/inactive colors, gradients, font size, glow, box style (classic / minimal / outline / neon), box opacity and padding, bottom offset, animations (`pop`, `glow`, `bounce`, `slide`, `beat`, `aniverse`).

**Output mode:** `karaoke` (lyrics only) or one of **40+ visualizer modes** (pulse, spectral, aurora, space blackhole, quasar jet, accretion disk, nested geodesic, crystal cage, …).

**Export:** CRF, x264 preset, encoder (CPU / NVENC / HEVC / QSV), audio bitrate, audio passthrough, loudness normalize.

**Render engine:** **Remotion** (React + Chromium) or **HyperFrames** (headless Chrome template) — badge in header shows REM vs HF.

**Actions:** **GO**, **GO BATCH**, cancel, retry, download MP4, live subtitle preview synced to phrases, optional clone-on-GO.

**Presets:** save/load/delete named **studio presets** (JSON) — subtitle + visualizer + export bundle.

---

### Audio Effects Lab (AFX)

Design the look **before** render.

**Subtitle preset rack** — same 12 presets as Studio (news_cyan, karaoke_gold, neon_pop, minimal_white, outline_red, big_impact, tiktok_bold, podcast_clean, retro_vhs, cinema_amber, aniverse_pop, …).

**40+ video effect presets** — paired subtitle colors + `output_mode` (e.g. music_pulse_cyan, quasar_jet_orange, event_horizon_purple).

**Visualizer preview** against uploaded or playlist audio — beat sync, EQ meters, beat detector toggle.

**Custom effect editor** — tweak parameters live.

**Playlist import** — analyze multiple tracks; audio analysis cached by content hash.

**Capture for GO** — push lab settings into Studio job settings in one action.

---

### Video Effects Lab (VFX)

Canvas-based **43 overlay effects** in six groups:

| Group | Examples |
|-------|----------|
| Tactical | HUD, targeting reticle, radar, scanner, lidar, night vision, thermal, crosshair, sonar |
| Cyber | Matrix, circuit, neural mesh, terminal, data HUD, hologram, neon grid, code rain, data stream, SIGINT |
| Science | DNA helix, biometric, oscilloscope, atom, graph overlay |
| Drama | Particles, explosion, glitch, lightning, film burn, rain, shockwave |
| Subtle | Ambient particles, scanlines, warp speed, film grain, light leak, bokeh |
| Broadcast | Lower third, breaking news, news chyron, name card, data box, channel bug |

**Controls:** per-effect parameters, subtitle/ticker/prompt sub-panels, video player, preview frame from server.

**LLM assist:** **Suggest** best overlay for content; **Chat** assistant (OpenRouter).

**Export paths:** burn overlay onto video via API; convert WebM screen capture → MP4 (H.264/H.265).

Syncs subtitle config with Automate `studio_global` when saved.

---

### Context Prompts

Edit text that steers AI behavior:

- **Automate LLM prompts** — system role, tweet instructions, workflow steps, per-language context blocks.
- **Whisper context profiles** for Studio — initial prompt strings per language/use case.

---

### Settings

Global defaults and credentials entry point.

**UI:** font choices for dashboard.

**Whisper:** default language, model, beam, temperature, device.

**Trim / caps / translate / diarization** defaults.

**Export:** encoder, CRF, NVENC preset, deinterlace default, render engine default.

**DLV:** downloader behavior options.

**Presets:** import/export settings JSON.

**API keys:** OpenRouter, HuggingFace, DeepL, Telegram, VK, X/Twitter, Rumble — persisted to local `.env` (not in git).

---

### HW (hardware watch)

500 ms refresh panel:

- App process RSS and CPU
- **nvidia-smi** VRAM and GPU utilization when NVIDIA present
- Host RAM
- Whisper model cache on disk

Header also polls GPU summary ~1 s for Studio tab badge.

---

### DLV (downloader)

Built-in **yt-dlp** workstation.

**Fetch:** paste URL → metadata + available formats; playlist mode for series.

**Download:** pick format, audio-only option, optional subtitles, cookie auth per platform (upload/remove cookie files).

**Controls:** pause, resume, cancel active download; update yt-dlp binary from UI.

**History:** list past downloads; clear selected or all.

**Playlist player:** built-in player with shuffle/repeat — audition before Studio.

**Send to Studio:** push finished file into upload queue.

---

### UVR (stem separation)

Separate vocals from instrumentals before transcription.

**Upload** audio file.

**Pick model** from `audio-separator` model list (server `/uvr/models`).

**Start job** — poll status every 2 s while running.

**Download** stems (vocals, instrumental, etc.) when complete.

Use vocal stem in Studio when the source mix is muddy.

---

### Automate

Eight sub-tabs for the full autonomous pipeline.

#### Test Lab

- Upload video or inspect scraper-fed **runs**
- Step indicator through upload → transcribe → categorize → render → done
- Preview `output.mp4` in panel
- **Approve** / **Reject** when manual approval mode is on
- **Re-run** from saved input
- Start/stop background scraper
- Filter runs by status

#### Stats

Processing counts, posting statistics, queue health.

#### Sources

Enable/disable scrape sources per platform:

| Platform | Type |
|----------|------|
| `youtube_channel` | yt-dlp channel/video poll |
| `telegram` | Telethon channel video documents |
| `vk` | VK wall API |
| `rss` | HTTP RSS fetch |
| `twitter` | X scrape path |

Per source: duration min/max, language, relevance filters. **Test connection** button. Default catalog includes template entries (news/OSINT channels) — **all disabled until you enable them**.

#### Pipeline

Whisper model, OpenRouter model pick, category list, duration gates, poll interval, concurrency, retention days, **manual approve mode**, minimum confidence to auto-post, video normalization toggles.

#### Chat

Operator chat with Automate backend context.

#### Models

Fetch OpenRouter and HuggingFace model lists; **benchmark** models on sample prompt; persisted scores; **best model** recommendation.

#### Distribute

Global auto-post toggle; per-category **X**, **Rumble**, **Telegram** channel routing; affiliate link injection; UTC posting window; `maxPostsPerHour` and minimum interval; thread reply mode for X.

Default categories: military, emergency, catastrophe, cyber, geopolitics, protest, intelligence, balkans, south_america.

#### Notifications

Telegram bot alerts by category; blocked-run and error notifications; **send test** ping.

**Automate render note:** output uses **FFmpeg ASS karaoke subtitles + drawtext news ticker** (and optional Python VFX overlay) — optimized for fast news clips. Studio’s **HyperFrames** path is separate and used only when you select HyperFrames in Studio settings.

---

### Video editor

Placeholder tab (`ready: false`) — reserved for future NLE-style editing. Not shipped yet.

---

## Subtitle and visualizer system

### Subtitle presets (12)

`news_cyan`, `karaoke_gold`, `neon_pop`, `minimal_white`, `outline_red`, `big_impact`, `tiktok_bold`, `podcast_clean`, `retro_vhs`, `cinema_amber`, `aniverse_pop`, and variants tied to visualizer presets.

### Visualizer modes (40+)

Including: `visualizer_2d`, `visualizer_2d_mirror`, `visualizer_3d`, `visualizer_pulse`, `visualizer_pulse_v2`, `visualizer_spectral`, `visualizer_aurora`, `visualizer_waveform`, `visualizer_beat_ring`, space-themed (`visualizer_space_blackhole`, `visualizer_quasar_jet`, `visualizer_accretion_disk`, `visualizer_event_horizon`, …), and geometric modes (`visualizer_nested_geodesic`, `visualizer_icosa_pulse`, `visualizer_crystal_cage`, …).

Remotion composition **`KaraokeVideo`**: `OffthreadVideo` + `KaraokeSubtitles` + optional visualizer React layer; duration from `props.json`.

### HyperFrames path (Studio only)

When `render_engine = hyperframes`:

1. Copy input + `phrases.json` to job `hf_public/`
2. Template `karaoke.html` with colors, fonts, beat markers, overlay style (`vignette`, `scanlines`, `particles`, `glitch`, …)
3. `npx hyperframes render` → MP4
4. Optional same GPU re-encode pass as Remotion

Requires **Node 22+** and `hyperframes` npm package.

---

## Automate pipeline — step by step

| Step | Work |
|------|------|
| **Upload / ingest** | Scraper download or Test Lab upload; probe; duration and audio gates; optional normalize |
| **Transcribe** | Whisper; optional cheap-model pre-filter and relevance check |
| **Categorize** | OpenRouter → category, location, confidence, archive flags, tweet text + thread, credit line, suggested VFX mode |
| **Render** | FFmpeg ASS lines + ticker bar; category overlay from `studio_presets/`; optional `vfx_generator` overlay |
| **Done** | `automate_runs/{id}/output.mp4`; optional enqueue to distributor |

**Human gates:** `manualApproveMode` (wait up to 24 h for approve/reject); `minConfidenceToPost`; moderation block flags.

**Posting queue:** respects UTC schedule window, hourly cap, minimum interval between posts, affiliate link frequency per category.

**Platforms:** X via twikit (Playwright fallback); Rumble via Playwright upload; Telegram channels via Telethon; alert bot for operator notifications.

---

## Job and run states

### Studio job (`state.json`)

| Status | Meaning |
|--------|---------|
| `uploaded` | File saved, not started |
| `running` | Pipeline active |
| `done` | `output.mp4` ready |
| `error` | Failed; message in `state.error` |

Each step: `pending` → `running` → `done` | `error` with percent progress.

### Automate run

| Status | Meaning |
|--------|---------|
| `running` | Pipeline in progress |
| `done` | Output ready |
| `error` | Failed with traceback |
| `blocked` | Duration/audio/moderation gate |
| `rejected` | Operator rejected in Test Lab |

Flag `pendingApproval` when waiting for human OK before render/post.

---

## API surface (grouped)

FastAPI server — dashboard calls `/api/...` (middleware strips prefix except Automate routes under `/api/automate/...`).

| Area | Capabilities |
|------|----------------|
| **Health** | `/`, `/health`, `/gpu`, `/system` |
| **Whisper models** | Download to cache, download status |
| **Studio jobs** | Upload, batch upload, list, get state, process, batch process, cancel, retry, clone, delete, phrases, audio analysis, render log, download MP4 |
| **Lab cache** | Audio analysis upload and cache by hash |
| **Remux** | FFmpeg container fix → MP4 |
| **Studio presets** | CRUD JSON presets |
| **yt-dlp** | Info, playlist info, download with progress, pause/resume/cancel, cookies, history, subtitles, thumbs, file serve, update binary |
| **UVR** | Models list, start job, status, list jobs, download stems |
| **VFX** | Preview PNG, mode list, LLM suggest/chat, generate, burn, convert WebM→MP4 |
| **WebSocket** | `/ws/log` live pipeline stream |
| **Automate** | Config sections, keys, source tests, test-run CRUD, approve/reject, scraper start/stop, stats, queue, post, history, model registry, Telegram auth, notifications test |

Static files served from `jobs/{id}/` for preview during pipeline.

---

## Operator workflows

### First-time setup

```bash
bash setup-karaoke-venv.sh    # Python venv + server deps
cp .env.example .env          # add API keys as needed
bash start-karaoke-studio.sh  # API :8765 + dashboard :5175
```

Open dashboard in browser. For HyperFrames renders, ensure Node 22+ (start script checks).

### Karaoke clip (manual)

1. **Studio** → upload MP4/MKV/audio
2. Optional: **AFX** tune preset and visualizer → capture settings
3. Optional: trim, set Whisper model/language, phrase limits
4. **GO** → watch WebSocket log and step bar
5. Download **output.mp4**

### Batch music videos

Multi-select upload → **GO BATCH** → sequential renders while you work other tabs.

### Download → render

**DLV** → paste YouTube URL → download → send to Studio → GO.

### Clean vocals first

**UVR** → separate vocals → download vocal WAV → upload to Studio.

### Breaking-news automation

1. **Settings** → enter OpenRouter, Telegram, X, Rumble keys
2. **Automate → Sources** → enable channels, set duration bounds
3. **Automate → Pipeline** → pick models, categories, approval mode
4. **Automate → Distribute** → map categories to platforms and schedule
5. **Test Lab** → upload sample → approve output
6. **Start scraper** → monitor Stats and Notifications

### Proof scripts (operator tooling)

External helpers in Logic Encoder `useful-scripts`: `util_karaoke_operator_preflight.sh` (GPU/ffmpeg/API check), `util_karaoke_job_smoke.sh` (process job + poll until done).

---

## Tech stack

| Layer | Technologies |
|-------|----------------|
| API | Python 3, FastAPI, uvicorn, orjson |
| Transcription | openai-whisper, PyTorch (CUDA), optional pyannote diarization |
| Audio analysis | librosa, scipy, soundfile |
| Translation | DeepL API, deep-translator |
| Video prep | ffmpeg/ffprobe, yadif, NVENC/QSV/HEVC |
| Studio render | Remotion 4 (React 18) **or** HyperFrames 0.6 (Node 22+) |
| Automate render | FFmpeg ASS + drawtext ticker (+ Pillow VFX generator) |
| Dashboard | Vite 5, React 18, TypeScript, lucide-react |
| VFX Lab | HTML Canvas 2D + server-side preview/burn |
| Downloader | yt-dlp |
| Stem separation | audio-separator (UVR) |
| Automate scrape | Telethon, httpx, yt-dlp |
| LLM | OpenRouter (categorize, VFX suggest, chat, model benchmarks) |
| Social post | twikit, Playwright, Telethon |
| Realtime UI | WebSocket log bus |
| Storage | Filesystem `jobs/`, `automate_runs/`, JSON configs (gitignored) |
| 3D / motion | three.js, gsap, matter-js, animejs, simplex-noise (Remotion visualizers) |

---

## Related repositories

| Repository | Role |
|------------|------|
| [le-karaoke-studio](https://github.com/logicencoder/le-karaoke-studio) | Private application code |
| [le-karaoke-studio-overview](https://github.com/logicencoder/le-karaoke-studio-overview) | This product overview |

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
