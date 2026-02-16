---
name: video-production
description: Generate videos by following user instructions and invoking related skills only when needed (`openai-text-to-image-storyboard`, `docs-to-voice`, `remotion-best-practices`). Use when users want text or existing assets turned into rendered videos, proactively ask interactive clarification questions whenever required information is missing (for example subtitle style), and require plan document confirmation before generation.
---

# Video Production

## Core Rules

1. Follow user instructions first. Do not force a fixed pipeline.
2. Call only relevant skills for missing work:
   - `openai-text-to-image-storyboard`: generate storyboard images when user did not provide usable visuals.
   - `docs-to-voice`: generate narration/timeline/SRT when user did not provide audio or subtitles.
   - `remotion-best-practices`: compose and render the final video output.
3. Before any asset generation or rendering, create a plan markdown in `<project_dir>/docs/plans/` and wait for explicit user confirmation.
4. Never ask the user whether to output a single video or multiple clips.
   - If the user explicitly asks for clips/parts, produce multi-clip output.
   - Otherwise default to one full video.
5. Keep Remotion project files unless the user explicitly asks for cleanup.
6. Keep git repositories clean by ensuring Remotion dependency/build/cache artifacts are ignored.

## Interactive Clarification (Required)

Before running generation commands, check whether details are sufficient. If not, ask concise targeted questions and wait for user replies.
Do not guess preference-sensitive options.

Prioritize asking these missing items:

- subtitle style (font, size, color, stroke/background, position, max lines)
- orientation/aspect ratio or exact resolution
- narration language/voice tone (if narration must be generated)
- duration constraints (total duration, or clip duration only when the user requested clips)
- visual style keywords (if storyboard images must be generated)
- output location / filename rules when not obvious

Ask only the minimum set required to unblock progress.

## Inputs to Collect

Collect only what is needed for the requested job (not everything by default):

- `project_dir`
- `content_name`
- final transcript text used for narration/subtitles
- source script/text (unless the user provides complete audio + subtitles)
- user-provided assets (images, audio, subtitles, branding)
- output requirements from user instructions

## Workflow

### 1) Read user intent and choose skill calls

Create a concise execution checklist:

- what the user already provided
- what still needs generation
- which dependency skills will be called
- what will be skipped

### 2) Create production plan markdown and wait for confirmation

Before generating images/audio/subtitles/video:

- create directory `<project_dir>/docs/plans/` if missing
- create a plan file named `<YYYY-MM-DD>-<content_name>.md`
- load the reference template at `references/plan-template.md`
- copy the template into the plan file first, then fill it
- use local date for `YYYY-MM-DD`; sanitize `content_name` for filename safety
- include at least these sections:
  - `## Video Transcripts`
  - `## Images To Generate`
- in `Video Transcripts`, include the transcript content that will be used for narration/subtitles
- in `Images To Generate`, list each image that still needs generation (scene id or order + prompt/description + intended usage)
- replace all square-bracket placeholders with concrete content
- remove all square-bracket placeholder/instruction text before sharing the plan for confirmation
- return the absolute plan file path to the user and ask for explicit confirmation
- do not run generation/render commands until user confirms the plan document

### 3) Generate missing visuals (if needed)

When visuals are missing, use `openai-text-to-image-storyboard`.

Aspect ratio mapping:

- `vertical` -> `9:16`
- `horizontal` -> `16:9`

Command:

```bash
python /Users/tszkinlai/.codex/skills/openai-text-to-image-storyboard/scripts/generate_storyboard_images.py \
  --project-dir <project_dir> \
  --env-file /Users/tszkinlai/.codex/skills/openai-text-to-image-storyboard/.env \
  --content-name "<content_name>" \
  --prompts-file "<project_dir>/pictures/<content_name>/prompts.json" \
  --aspect-ratio <9:16|16:9>
```

### 4) Generate missing narration/subtitles (if needed)

When narration or subtitle assets are missing, use `docs-to-voice`.

```bash
python /Users/tszkinlai/.codex/skills/docs-to-voice/scripts/docs_to_voice.py \
  --project-dir <project_dir> \
  --project-name "<content_name>" \
  --text "<source_text>"
```

Expected output under `<project_dir>/audio/<content_name>/`:

- narration audio
- timeline JSON
- SRT subtitle file

### 5) Compose and render video (`remotion-best-practices`)

Always load and follow:

- `rules/compositions.md`
- `rules/audio.md`
- `rules/subtitles.md`
- `rules/import-srt-captions.md`
- `rules/display-captions.md`

Render rules:

- apply user-defined subtitle style to all outputs in the same job
- if subtitle style is missing, ask first (do not silently hardcode)
- render subtitles cue-by-cue by SRT timing (never all cues at frame `0`)
- `single` output: render one full MP4
- `multi` output: only when the user requested clips; split by user rule and render one MP4 per clip

### 6) Keep reusable project artifacts

Default workspace:

- `<project_dir>/video/<content_name>/remotion/`

Keep Remotion sources by default unless the user explicitly requests cleanup.

Git hygiene (required):

- create or update `<project_dir>/video/<content_name>/remotion/.gitignore`
- do not remove user-defined ignore rules already in that file
- ensure it includes at least these entries:
  - `node_modules/`
  - `.cache/`
  - `dist/`
  - `build/`
  - `out/`
  - `.DS_Store`
  - `*.log`

## Output Contract

Return absolute paths for produced artifacts:

- plan markdown file
- storyboard directory (if generated or used)
- narration audio file (if generated or used)
- subtitle SRT file (if generated or used)
- final rendered MP4 (single) or ordered clip list (multi)
- Remotion workspace directory
- Remotion `.gitignore` file path

Also report:

- what assumptions were clarified through interactive questions
- subtitle sync verification (first/middle/last cue spot check)
