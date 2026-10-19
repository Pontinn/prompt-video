# Media analysis

Mandatory before writing a prompt that mentions any file in `brutos/`. Analysis only reads. Nothing in `brutos/` is modified, converted in place, moved or deleted. All intermediate output (frames, contact sheets, transcripts, downmixes) goes to a scratch folder outside `brutos/` and `prompts/`.

## Tools

Check what is available before running anything:

```bash
ffprobe -version
ffmpeg -version
npx hyperframes --help
```

Priority:
1. `ffprobe` and `ffmpeg` for metadata, frames, contact sheets, audio downmix.
2. HyperFrames transcription when the `/hyperframes` skill exposes it (consult that skill for the exact command). Otherwise Whisper (`faster-whisper` or `openai-whisper`) installed in a local venv inside the project.
3. Python with NumPy and SciPy in the same local venv for loudness, silence detection and cross-correlation.

If a tool is missing, install it locally (venv in the project, npm in the project folder). Never global. If installation is impossible, say so in Portuguese, do what is possible with the remaining tools and state in the prompt which facts could not be verified.

## Video

### Metadata

```bash
ffprobe -v error -show_entries format=duration:stream=index,codec_type,codec_name,width,height,r_frame_rate,avg_frame_rate,channels,sample_rate,display_aspect_ratio -of json "brutos/<file>"
```

Record: duration in seconds and milliseconds, width × height, orientation (landscape, portrait, square), fps (reduce r_frame_rate to a number), video codec, audio streams with channels and sample rate.

### Frames

Extract one frame every N seconds (N = 1 for videos under 60 s, N = 2 up to 5 min, N = 5 beyond) and a contact sheet:

```bash
ffmpeg -v error -i "brutos/<file>" -vf "fps=1/N,scale=320:-1" "<scratch>/frames/f_%04d.png"
ffmpeg -v error -i "brutos/<file>" -vf "fps=1/N,scale=240:-1,tile=6x8" "<scratch>/contact_%02d.png"
```

Look at the contact sheet and at individual frames around any change. Record:
- Content type: person talking to camera, screen recording, product shots, filmed scene, animation, slides, mixed.
- Scene changes and their approximate times (use `select='gt(scene,0.3)'` with `showinfo` when the contact sheet is not enough).
- Visible text, logos, interfaces, brand colors.
- Dominant colors and lighting.
- Problems: shaky sections, dark or blown-out sections, off-camera gaze on talking heads, dead sections with nothing happening, letterboxing, burned-in captions.
- Framing facts that affect layout: where the face sits, headroom, whether a 16:9 source can be center-cropped to 9:16 without losing the subject.

### Embedded audio

If the video has an audio stream, run the Audio section on it as well. When a separate clean audio file exists, compute the offset between them:

```python
# downmix both to mono 16 kHz, normalize, cross-correlate (scipy.signal.correlate), take the lag of the peak
# report the offset in ms and the peak-to-median ratio; a clear peak confirms the alignment
```

## Audio

### Metadata

```bash
ffprobe -v error -show_entries format=duration:stream=codec_name,channels,sample_rate,bit_rate -of json "brutos/<file>"
```

### Transcription

Produce a word-level transcript with start and end times. Save it to the scratch folder as JSON and as plain text. From it derive:
- Sentence list with start and end in ms.
- Pauses longer than 250 ms (candidates for tightening to about 120 ms).
- Repeated takes: the same sentence attempted more than once (keep the last complete one by default).
- Restarts, filler, prep words ("ok", "então", "tá"), mumbled asides.
- The hook (first sentence), the call to action (last sentences), and every key mention: product names, numbers, dates, facts, instructions, emotional beats.
- Speaking pace (words per minute) and where emphasis falls.

### Quality

With FFmpeg `ebur128` and `silencedetect`:

```bash
ffmpeg -v info -i "brutos/<file>" -af ebur128=peak=true -f null - 2>&1 | tail -n 12
ffmpeg -v info -i "brutos/<file>" -af silencedetect=noise=-35dB:d=0.25 -f null - 2>&1 | grep silence_
```

Record integrated LUFS, true peak, long silences, noise floor, clipping, level jumps.

## Audio + video together (audio-led modes)

Build the segment map. One row per sentence or per idea in the narration:

| # | start ms | end ms | what is said (short) | what the footage shows at that time, or NONE | action for the prompt |
|---|---|---|---|---|---|

Rules for the "action" column:
- Footage shows what is said: cut to that footage range at the exact mention, with a 50 ms audio lead (J cut).
- Footage does not show it: generate a composition by code (card, interface, number, logo, diagram) that illustrates the sentence.
- Footage shows something unrelated to any sentence: leave it out.
- Footage shorter than the narration: fill with generated compositions, never with frozen frames or visible repeats.
- The final duration is the edited narration (END). The footage adapts to it, never the other way around.

This table is pasted into the prompt's PLAN BEFORE BUILDING block as the draft segment table.

## Reference video (style only)

When the user says the video is a reference and not footage, extract rules instead of content:
- Cut cadence: average and shortest shot length.
- Camera and motion: static, handheld, push-ins, whip pans, parallax.
- Typography: serif or sans, size relative to frame, weight, placement, animation style.
- Palette: three to five dominant colors as hex.
- Captions: present or not, words per block, style.
- Sound: tempo in BPM (estimate from beats), presence of risers, impacts, ducking, narration.
- Structure: hook length, number of beats, how it ends.

Turn these into numbered rules in a STYLE REFERENCE block. The reference file is never listed as footage in INPUTS.

## Summary to the user

Before moving on, summarize in Portuguese in five to ten lines: files found, duration and format of each, what the video shows, what the audio says (one or two sentences), problems found, and the mode you will use. Do not paste full transcripts unless asked.
