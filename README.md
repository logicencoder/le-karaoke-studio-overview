# LE Karaoke Studio

AI-assisted **karaoke subtitle burn-in** and short-form media automation for Logic Encoder operators. The app turns raw media into timed lyric videos with configurable styling, rendering, and optional distribution workflows.

Private source: [logicencoder/le-karaoke-studio](https://github.com/logicencoder/le-karaoke-studio). API keys and platform cookies stay in your local `.env` and gitignored config.

## Tech stack

| Layer | Technologies |
|-------|--------------|
| API | FastAPI + uvicorn |
| Transcription | Whisper (CUDA when available) |
| Rendering | Remotion + optional HyperFrames path |
| Dashboard | React + Vite |
| Automation | yt-dlp + OpenRouter + social integrations |

## Studio pipeline

The **Studio** tab runs one job end to end:

1. **Upload** media (video/audio) and queue jobs.
2. **Analyze** duration, resolution, FPS, and optional trim settings.
3. **Transcribe** with Whisper and selected model/language.
4. **Shape phrases** into readable karaoke lines.
5. **Render** with Remotion or HyperFrames.
6. **Preview and download** final MP4 output.

This separates transcription from phrase shaping so operators can keep lyric lines readable instead of dumping raw token timing.

## Visual and audio controls

Subtitle style presets, visual effect controls, and timing behavior can be tuned per channel style. AFX/VFX and UVR tools support harder source material where vocals need cleanup before transcription.

## Downloader and Automate workflows

The DLV tab provides built-in source ingestion with yt-dlp. The Automate module handles repetitive pipelines: discover source clips, classify and prepare content, render candidates, queue human approval, then publish to selected social destinations.

Human approval stays in the loop before posting.

## Dashboard tabs

| Tab | Purpose |
|-----|---------|
| Studio | Upload → transcribe → phrases → render |
| AFX / VFX | Effect labs and preset tuning |
| DLV | yt-dlp source queue |
| UVR | Vocal/instrument stem separation |
| Automate | Discover → classify → approve → distribute |
| HW | GPU/CPU metrics |

## Quick start

```bash
bash setup-karaoke-venv.sh
bash start-karaoke-studio.sh
# Dashboard: http://localhost:5175
```

Copy `.env.example` to `.env` for provider credentials. See [REPOS.md](REPOS.md) for repository links.

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
