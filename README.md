# Prompt Video (Claude Code Skill)

Generates complete, production-grade prompts for Claude Code to produce a video with the HyperFrames skill. It does not make the video: it scans the directory, analyzes the raw files you drop in `brutos/`, asks only what matters, optionally proposes three concepts, writes a detailed English prompt and saves it as a `.txt` in `prompts/`.

## Installation

1. Copy the whole `prompt-video/` folder into `~/.claude/skills/prompt-video/`
2. Make sure the HyperFrames plugin by HeyGen is installed (the generated prompts call `/hyperframes`)
3. Restart Claude Code
4. Run `/prompt-video` in the folder where the video project will live

## How it works

1. `/prompt-video` scans the current directory and creates `brutos/` (raw files) and `prompts/` (saved prompts) if they do not exist.
2. You drop video, audio or images into `brutos/` and say so. No files means the video is built from scratch.
3. The skill analyzes every file: metadata, frames, transcription with word timestamps, quality, and, when audio and video are both present, a segment map that ties what is said to what is shown.
4. It asks, in one round and in Portuguese, only the questions that change the result.
5. If you ask for ideas, it proposes exactly three concepts based on the analysis and waits for your pick.
6. It writes the prompt in English following the fixed skeleton in `references/prompt-skeleton.md`, shows it in chat and saves it to `prompts/YYYY-MM-DD-<slug>.txt`.

## Modes

| Inputs in `brutos/` | Mode |
|---|---|
| Nothing | Text mode, video created from scratch |
| Video only | Video mode |
| Audio only | Audio mode, narration drives everything |
| Audio + video | Audio-led mode, footage is illustrative b-roll |
| Camera video + clean audio | Talking-head mode (sync by cross-correlation) |
| Video flagged as reference | Style rules extracted, never used as footage |

## Files

- `SKILL.md`: the workflow Claude Code follows.
- `references/prompt-skeleton.md`: mandatory and conditional blocks, defaults, style rules of the generated prompt.
- `references/media-analysis.md`: how video and audio are inspected before writing.
- `examples/`: three reference prompts (talking-head reel, cinematic trailer, 3D character short), kept in Portuguese as received.

## Rules worth knowing

- Every prompt names HyperFrames explicitly.
- Nothing in `brutos/` is ever modified, moved or deleted.
- Nothing is installed globally.
- Suggestions only when you ask for them.
- Conversation in Portuguese, generated prompt in English (ask for Portuguese if you prefer).
- No em dash anywhere.
