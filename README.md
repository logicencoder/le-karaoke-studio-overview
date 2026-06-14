# Le Karaoke Studio

AI-assisted **karaoke subtitle burn-in** and short-form video factory — upload a clip, transcribe with word-level timing, group readable lyric lines, render highlighted karaoke MP4, and optionally run an **Automate** pipeline that scrapes sources, categorizes with LLMs, and posts to social channels.

Private source: [logicencoder/le-karaoke-studio](https://github.com/logicencoder/le-karaoke-studio). API keys and platform cookies stay in your local `.env` and gitignored config — never in this overview repo.

## The problem it solves

Manual karaoke timing in a non-linear editor is slow and inconsistent. Whisper gives word timestamps, but raw tokens are not singable lines. Le Karaoke Studio turns speech-to-text into **on-screen lyrics with per-word highlights**, preset visual styles, and optional downstream automation for channels that publish captioned shorts at scale.

## Studio pipeline

The **Studio** tab walks one job end to end:

1. **Upload** — drag/drop MP4, MOV, WebM, MKV, or audio; batch queue supported.
2. **Analyze** — duration, resolution, FPS; optional trim and deinterlace.
3. **Transcribe** — OpenAI Whisper with language and model choice; **CUDA strongly preferred** for speed.
4. **Phrases** — merge tokens into lines of 2–12 words; optional ALL CAPS output for stylized channels.
5. **Render** — Remotion 4 (React) or experimental HyperFrames (HTML + headless Chrome).
6. **Preview / download** — in-browser result and exported MP4.

Separate phrase editing exists because karaoke needs readable line breaks, not raw Whisper output.

## Visual and audio styling

Subtitle presets (box style, glow, font), video effect presets (visualizers, ticker, logo overlay), beat-synced motion when music sync is enabled, and per-speaker colors when diarization is on. One engine supports multiple channel looks — news ticker, neon club, minimal captions — without separate apps.

**AFX / VFX** tabs expose audio and video effect labs for preset tuning before render.

## Downloader (DLV)

Built-in **yt-dlp** UI — paste a URL, pick quality, pull source media straight into the studio pipeline without a separate download tool.

## UVR — vocal separation

**Ultimate Vocal Remover** models split vocals vs instrumental stems before transcription or remix work — useful when the original mix is dense.

## Automate module

Background workflow for repetitive clip channels:

- Scraper watches configured sources (e.g. YouTube) for new clips in a duration window.
- Whisper transcription plus **OpenRouter** categorization.
- Render a short, then **human approve** → schedule posts to **X**, **Telegram**, and **Rumble** with rate limits.

Operator runs the factory on a trusted workstation; followers receive captioned shorts. Respect platform terms and local law when using downloaded or reposted media.

## Dashboard layout

| Tab | Purpose |
|-----|---------|
| Studio | Upload → transcribe → phrases → render |
| AFX / VFX | Effect labs and presets |
| DLV | yt-dlp download queue |
| UVR | Vocal/instrument stems |
| Automate | Scrape → categorize → approve → distribute |
| HW | GPU/CPU metrics |

## Stack and quick start

| Layer | Technology |
|-------|------------|
| API | FastAPI + uvicorn (`server/main.py`, default port **8765**) |
| Dashboard | Vite + React (default port **5175**) |
| Transcription | Whisper (GPU when available) |
| Render | Remotion 4 or HyperFrames (Node 22+) |

```bash
bash setup-karaoke-venv.sh
bash start-karaoke-studio.sh
# Dashboard: http://localhost:5175
```

Copy `.env.example` to `.env` for Automate provider keys. See [REPOS.md](REPOS.md) for repository links.

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
