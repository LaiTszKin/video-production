---
name: video-production
description: Orchestrate text-to-video production with explicit dependencies on docs-to-voice, openai-text-to-image-storyboard, and remotion-best-practices. Supports single long video and multi-clip short-video output by generating full narration and images first, then splitting audio to user-defined clip length.
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
- orientation (`vertical` or `horizontal`)
- delivery mode (`single` or `multi`)
- target video duration (required for `single`)
- segment duration per clip (required for `multi`, for example `30s` or `60s`)

Do not guess orientation, delivery mode, or duration values.

If orientation is missing, ask whether the video should be vertical or horizontal.
If delivery mode is missing, ask whether user wants one full video (`single`) or multiple short clips (`multi`).
If `single` duration is missing, ask for target duration first.
If `multi` segment duration is missing, ask for the clip duration first.

## User Confirmation Gate (Required)

After all required inputs are collected, pause before production and ask for explicit user confirmation.

- Provide a concise preflight summary (orientation, delivery mode, duration settings, and planned output paths).
- Do not start scene planning, image generation, audio generation, segmentation, or rendering until user explicitly confirms to proceed.
- If user changes any requirement, update the summary and request confirmation again.

## Subtitle Format Standard (Required for all videos)

Apply one unified subtitle style to every video output (`single` and `multi`).
Unless user explicitly requests overrides, do not change subtitle style per clip or per video.

Style profile:

- source: use generated SRT timing directly; do not re-time subtitle timestamps.
- language: keep original subtitle language; do not auto-translate.
- layout: max `2` lines, centered, bottom offset `8%` of frame height.
- line width: max `18` full-width chars (or `36` half-width chars) per line.
- font family: `Noto Sans TC, PingFang TC, Microsoft JhengHei, sans-serif`
- font weight: `700`
- font size: `64px` (`vertical`), `56px` (`horizontal`)
- line height: `1.25`
- text color: `#FFFFFF`
- stroke: `rgba(0,0,0,0.92)` with width `6px` (`vertical`) / `4px` (`horizontal`)
- shadow: `0 2px 10px rgba(0,0,0,0.45)`

Implementation rules:

- Save subtitle style config to `<project_dir>/video/<content_name>/subtitle-style.json`.
- Use the same style config for full video and all segmented clips.
- If user asks for style changes, apply the same changed style to every output in that job.
- Timing contract:
  - `single`: convert each SRT cue to its own frame range using `startFrame = floor(startMs / 1000 * fps)` and `endFrame = ceil(endMs / 1000 * fps)`.
  - `multi`: after clipping by segment window, rebase subtitle time to segment-local time (`localMs = globalMs - segmentStartMs`) before converting to frames.
  - Never render all subtitle lines at frame `0`; each cue must keep its own time window.

## Workflow

### 1) Scene planning

- Convert the source text into scene prompts in narrative order.
- Keep scene-to-text alignment so each later audio segment can be paired with the right scene images.
- Save prompts JSON to `<project_dir>/pictures/<content_name>/prompts.json` using this schema:

```json
[
  {
    "title": "scene title",
    "prompt": "cinematic visual prompt"
  }
]
```

### 2) Generate full image set (`openai-text-to-image-storyboard`)

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

### 3) Generate full narration and subtitle files (`docs-to-voice`)

Generate one complete narration from the full source text first. Do not generate per-segment narration.

Use raw text or an input file:

```bash
python /Users/tszkinlai/.codex/skills/docs-to-voice/scripts/docs_to_voice.py \
  --project-dir <project_dir> \
  --project-name "<content_name>" \
  --text "<source_text>"
```

Expected output under `<project_dir>/audio/<content_name>/`:

- full narration audio (`.aiff/.wav/.mp3` depends on mode)
- `<audio_name>.timeline.json`
- `<audio_name>.srt`

### 4) Segment preparation (only when `delivery mode = multi`)

- Split the full narration audio into clips by user-defined segment duration.
- Split subtitles with the same boundaries to preserve caption sync.
- Build a segment manifest at `<project_dir>/video/<content_name>/segments.json`.
- Map each segment to matching scene images by timeline/text alignment. Do not pair images randomly.
- For each segment `[segmentStartMs, segmentEndMs)`:
  - keep only subtitle cues that overlap the segment window.
  - clamp cue boundaries to the segment window.
  - rebase kept cues to segment-local time (`0` to `segmentDurationMs`) before saving `segment-xxx.srt`.
  - drop cues with non-positive duration after clamping.

Recommended segment assets:

- `<project_dir>/audio/<content_name>/segments/segment-001.<ext>`
- `<project_dir>/audio/<content_name>/segments/segment-001.srt`
- `<project_dir>/video/<content_name>/segments.json`

### 5) Build and render Remotion video(s) (`remotion-best-practices`)

Before implementation, load these dependency references:

- `rules/compositions.md`
- `rules/audio.md`
- `rules/subtitles.md`
- `rules/import-srt-captions.md`
- `rules/display-captions.md`

Common composition settings:

- size: `1080x1920` for vertical, `1920x1080` for horizontal
- `fps = 30`
- subtitles rendered as overlay captions with the unified subtitle style profile
- render subtitles by cue timing (cue-by-cue visibility), not by static full-text overlay

Render rules:

- `single`: use full image sequence + full narration audio + full SRT, then render one MP4 with the unified subtitle style.
- `multi`: for each segment, use matched scene images + segmented audio + segmented SRT, then render one MP4 per segment with the same unified subtitle style.
- In Remotion, each subtitle cue must be rendered in its own `Sequence` (or equivalent frame-gated visibility) based on parsed SRT timestamps.
- If all cues appear near frame `0`, treat as a sync bug and fix before final delivery.

Output location:

- `single`: `<project_dir>/video/<content_name>.mp4`
- `multi`: `<project_dir>/video/<content_name>/<content_name>-part-001.mp4` (ordered series)

Duration rules:

- `single`: honor target total duration.
- `multi`: each clip should match user segment duration; final clip may be shorter for remainder.

## Output Contract

Return absolute paths for:

- storyboard folder
- full narration audio file
- full subtitle SRT file
- subtitle style config file
- final rendered MP4 file (`single`) or ordered MP4 clip list (`multi`)
- segment manifest file (`multi` only)

Also report whether duration requirements are satisfied (`single`: total duration, `multi`: per-clip duration + remainder clip).
Also report subtitle sync verification result (first/middle/last cue timing check against narration timeline).
