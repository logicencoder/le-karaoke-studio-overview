# Le Karaoke Studio

AI-assisted **karaoke subtitle burn-in** and optional **short-form video automation** for LogicEncoder operators. Source code is private: [le-karaoke-studio](https://github.com/logicencoder/le-karaoke-studio).

Runs on a **local workstation or SOL server** (GPU Whisper). **Not** part of the logicencoder.com WordPress site on Hostinger.

## What it does

**What:** Upload a music video or clip → automatic speech-to-text with word-level timing → grouped on-screen lyrics → rendered MP4 with highlighted “singing” words.  
**Why:** Manual karaoke timing in an NLE is slow; Whisper plus a fixed visual template produces consistent social-ready clips.  
**Who:** Operator producing lyric videos, news-style captioned shorts, or meme clips.

## Studio workflow

| Stage | What the operator sees |
|-------|---------------------------|
| Upload | Drag/drop MP4/MOV/WebM/MKV or audio; batch queue |
| Analyze | Duration, resolution, FPS; optional trim and deinterlace |
| Transcribe | Whisper (GPU when CUDA available) with language and model choice |
| Phrases | Lines of 2–12 words; optional **ALL CAPS** output |
| Render | Remotion (React) or experimental HyperFrames HTML renderer |
| Result | In-browser preview, download MP4 |

**Why separate phrase step:** Karaoke needs readable line breaks, not raw Whisper tokens.  
**Who benefits:** Viewers get synced highlights; operator tunes max words per line and colors without editing timelines.

## Visual and audio options

**What:** Subtitle presets (box style, glow, font), video effect presets (visualizers, ticker, logo overlay), beat-synced motion when music sync is enabled, per-speaker colors when diarization is on.  
**Why:** One engine serves multiple “channels” (news ticker look vs neon club look) without separate apps.  
**Who:** Operator branding clips; audience gets recognizable style.

## Downloader (DLV tab)

**What:** Built-in yt-dlp UI — paste URL, pick quality, download into the studio pipeline.  
**Why:** Source material often starts on YouTube; avoids a separate download step.  
**Who:** Operator; respects platform ToS and local law when using downloaded media.

## UVR tab

**What:** Ultimate Vocal Remover models split vocals vs instrumental stems before further processing.  
**Why:** Cleaner transcription or remix workflows when the original mix is dense.  
**Who:** Operator preparing difficult audio sources.

## Automate module

**What:** Background scraper watches configured channels (e.g. YouTube), pulls new clips in a duration window, runs Whisper + **OpenRouter** categorization, renders a short, then queues human approve → post to **X**, **Telegram**, and **Rumble** with scheduling and rate limits.  
**Why:** Repetitive news/clip channels need a factory line, not one-off Studio uploads.  
**Who:** Operator runs the factory; followers on social platforms receive captioned shorts. API keys and cookies stay on the operator machine (never in this public repo).

## Hardware expectations

| Workload | Typical hardware |
|----------|------------------|
| Whisper | NVIDIA GPU (e.g. RTX class) strongly preferred |
| Remotion render | CPU (Chromium + ffmpeg) |
| HyperFrames | Node 22+, headless Chrome |

## Related repositories

| Repo | Role |
|------|------|
| [le-karaoke-studio](https://github.com/logicencoder/le-karaoke-studio) | Private source |
| [le-karaoke-studio-overview](https://github.com/logicencoder/le-karaoke-studio-overview) | This overview |

See [REPOS.md](REPOS.md).

## Licensing

Application code © LogicEncoder. Third-party models and platforms (Whisper, yt-dlp, social APIs) subject to their own terms.
