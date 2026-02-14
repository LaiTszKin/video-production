---
name: video-production
description: Orchestrate end-to-end text-to-video production with explicit dependencies on docs-to-voice, openai-text-to-image-storyboard, and remotion-best-practices. Use when users want to convert text, chapters, or scripts into narrated videos with generated images, subtitle overlays, orientation-specific output, and a user-defined target duration.
---

# Video Production

## Dependency Contract (Required)

Always run this workflow with these three skills:

1. `openai-text-to-image-storyboard` (generate storyboard images)
2. `docs-to-voice` (generate narration audio + timeline + SRT)
3. `remotion-best-practices` (compose and render subtitle video)

If any dependency is unavailable, stop and report which dependency is missing.

## Required Inputs

Collect these inputs before production:

- `project_dir`
- source text (raw text or a text file)
- `content_name` (used in output paths)
- target video duration
- orientation (`vertical` or `horizontal`)

Do not guess duration or orientation.

If duration is missing, ask the user for duration first.
If orientation is missing, ask whether the video should be vertical or horizontal.

## Workflow

### 1) Scene planning

- Convert the source text into scene prompts in narrative order.
- Align scene count and pacing with requested duration.
- Save prompts JSON to `<project_dir>/pictures/<content_name>/prompts.json` using this schema:

```json
[
  {
    "title": "scene title",
    "prompt": "cinematic visual prompt"
  }
]
```

### 2) Generate images (`openai-text-to-image-storyboard`)

Map orientation to aspect ratio:

- `vertical` -> `9:16`
- `horizontal` -> `16:9`

Run:

```bash
python /Users/tszkinlai/.codex/skills/openai-text-to-image-storyboard/scripts/generate_storyboard_images.py \
  --project-dir <project_dir> \
  --env-file /Users/tszkinlai/.codex/skills/openai-text-to-image-storyboard/.env \
  --content-name "<content_name>" \
  --prompts-file "<project_dir>/pictures/<content_name>/prompts.json" \
  --aspect-ratio <9:16|16:9>
```

Expected output:

- `<project_dir>/pictures/<content_name>/*.png`
- `<project_dir>/pictures/<content_name>/storyboard.json`

### 3) Generate narration and subtitle files (`docs-to-voice`)

Use raw text or an input file:

```bash
python /Users/tszkinlai/.codex/skills/docs-to-voice/scripts/docs_to_voice.py \
  --project-dir <project_dir> \
  --project-name "<content_name>" \
  --text "<source_text>"
```

Expected output under `<project_dir>/audio/<content_name>/`:

- narration audio (`.aiff/.wav/.mp3` depends on mode)
- `<audio_name>.timeline.json`
- `<audio_name>.srt`

### 4) Build and render Remotion video (`remotion-best-practices`)

Before implementation, load these dependency references:

- `rules/compositions.md`
- `rules/audio.md`
- `rules/subtitles.md`
- `rules/import-srt-captions.md`
- `rules/display-captions.md`

Build Remotion composition with:

- size: `1080x1920` for vertical, `1920x1080` for horizontal
- `fps = 30`
- image sequence from generated storyboard images
- narration audio as the primary track
- SRT subtitles parsed and rendered as overlay captions

Duration rule:

- If user provides target duration, honor it and distribute scene durations accordingly.
- If user does not provide duration, ask first (mandatory).

Render MP4 to:

- `<project_dir>/video/<content_name>.mp4`

## Output Contract

Return absolute paths for:

- storyboard folder
- narration audio file
- subtitle SRT file
- final rendered MP4 file

Also report whether final duration matches user requirement.
