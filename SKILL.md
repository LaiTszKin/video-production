---
name: video-production
description: Generate videos by following user instructions and invoking related skills only when needed (`text-to-short-video`, `openai-text-to-image-storyboard`, `docs-to-voice`, `remotion-best-practices`). For text inputs, first abstract the most important segment, then generate video through `text-to-short-video`; when recurring roles appear, reuse existing role prompts and generate JSON prompts in the defined format only for undefined roles. Ask interactive clarification questions for missing preferences and require plan confirmation before generation.
---

# Video Production

## Core Rules

1. Follow user instructions first. Do not force a fixed pipeline.
2. For text-driven requests, use `text-to-short-video` as the default production path.
3. Before calling `text-to-short-video`, extract the most important text segment and use that segment as the short-video source text unless the user already gave an exact segment.
4. Role prompt policy for prompt generation:
   - always ensure `<project_dir>/pictures/<content_name>/roles.json` exists before any prompt generation
   - if a recurring role already has a defined prompt skeleton, reuse it without rewriting identity fields
   - if a recurring role appears but is not defined, generate new role entries using the supported `prompts.json` schema
5. Call only relevant fallback skills for missing work outside the text-to-short-video path:
   - `openai-text-to-image-storyboard`: generate storyboard images when user did not provide usable visuals.
   - `docs-to-voice`: generate narration/timeline/SRT when user did not provide audio or subtitles.
   - `remotion-best-practices`: compose and render the final video output.
6. Before any asset generation or rendering, create a plan markdown in `<project_dir>/docs/plans/` and wait for explicit user confirmation.
7. Never ask the user whether to output a single video or multiple clips.
   - If the user explicitly asks for clips/parts, produce multi-clip output.
   - Otherwise default to one full video.
8. Keep Remotion project files unless the user explicitly asks for cleanup.
9. Keep git repositories clean by ensuring Remotion dependency/build/cache artifacts are ignored.

## Interactive Clarification (Required)

Before running generation commands, check whether details are sufficient. If not, ask concise targeted questions and wait for user replies.
Do not guess preference-sensitive options.

Prioritize asking these missing items:

- subtitle style (font, size, color, stroke/background, position, max lines)
- orientation/aspect ratio or exact resolution
- narration language/voice tone (if narration must be generated)
- duration constraints (total duration, or clip duration only when the user requested clips)
- visual style keywords (if storyboard images must be generated)
- existing role prompt source (user-provided prompt file or existing `<project_dir>/pictures/<content_name>/prompts.json`) when recurring roles are expected
- output location / filename rules when not obvious

Ask only the minimum set required to unblock progress.

## Inputs to Collect

Collect only what is needed for the requested job (not everything by default):

- `project_dir`
- `content_name`
- source script/text (unless the user provides complete audio + subtitles)
- extracted key segment text for short-video generation (unless user provides an exact segment)
- final transcript text used for narration/subtitles
- existing prompt assets (`prompts.json`, character roster, or user-defined role prompts)
- user-provided assets (images, audio, subtitles, branding)
- output requirements from user instructions

## Workflow

### 1) Read user intent and choose skill calls

Create a concise execution checklist:

- what the user already provided
- what still needs generation
- whether to run the text-first path (`text-to-short-video`) or fallback asset path
- which dependency skills will be called
- what will be skipped

### 2) Abstract the most important text segment (text-first path)

When the request includes source text and the user did not lock a specific excerpt:

- extract one self-contained high-impact segment suitable for a 30-60 second short video
- preserve original wording for key hooks and turning points whenever possible
- ensure the extracted segment has narrative continuity (setup -> tension/change -> lingering curiosity)
- keep this segment as the primary input passed into `text-to-short-video`

If the user already provides the exact segment, reuse it directly and skip abstraction.

### 3) Resolve role prompt reuse and ensure `roles.json` exists before prompt generation

Before generating or updating `prompts.json`:

1. detect recurring roles from the selected text segment
2. set target role file path:
   - `<project_dir>/pictures/<content_name>/roles.json`
3. if `roles.json` exists, read it first and reuse matching roles
4. if `roles.json` does not exist, create it first using `references/roles-json.md`
   - include detected recurring roles when available
   - if no recurring roles are detected, initialize with `{"characters": []}`
5. load reusable prompt sources in this order:
   - user-provided prompt file or role definitions
   - existing `<project_dir>/pictures/<content_name>/prompts.json` (if present)
6. apply role policy:
   - defined role: reuse existing role skeleton following `references/roles-json.md`
   - undefined role: add a new role entry following `references/roles-json.md`
7. choose schema:
   - use structured format with `characters` + `scenes` when recurring roles exist
   - use simple list format only when no recurring roles need continuity

### 4) Create production plan markdown and wait for confirmation

Before generating images/audio/subtitles/video:

- create directory `<project_dir>/docs/plans/` if missing
- create a plan file named `<YYYY-MM-DD>-<content_name>.md`
- load the reference template at `references/plan-template.md`
- copy the template into the plan file first, then fill it
- use local date for `YYYY-MM-DD`; sanitize `content_name` for filename safety
- include at least these sections:
  - `## Key Segment`
  - `## Video Transcripts`
  - `## Prompt Strategy`
  - `## Images To Generate`
- in `Key Segment`, include the extracted segment (or the user-locked segment)
- in `Prompt Strategy`, state which role prompts are reused vs newly defined and which JSON schema is used
- in `Video Transcripts`, include the transcript content that will be used for narration/subtitles
- in `Images To Generate`, list each image that still needs generation (scene id or order + prompt/description + intended usage)
- replace all square-bracket placeholders with concrete content
- remove all square-bracket placeholder/instruction text before sharing the plan for confirmation
- return the absolute plan file path to the user and ask for explicit confirmation
- do not run generation/render commands until user confirms the plan document

### 5) Generate video via `text-to-short-video` (default text path)

Use `text-to-short-video` after plan confirmation when the source is text-driven:

- pass `project_dir`, `content_name`, extracted segment text, and user-requested size/aspect ratio
- ensure `prompts.json` follows the defined schema and role reuse policy from step 3
- let `text-to-short-video` handle storyboard generation and final short render flow

### 6) Fallback asset pipeline (only when text-to-short-video is not the right path)

Use this path when the user explicitly needs manual asset-level control or provides partial assets that require direct orchestration:

- generate visuals via `openai-text-to-image-storyboard` only if needed
- generate narration/subtitles via `docs-to-voice` only if needed
- compose/render via `remotion-best-practices`

Render rules:

- apply user-defined subtitle style to all outputs in the same job
- if subtitle style is missing, ask first (do not silently hardcode)
- render subtitles cue-by-cue by SRT timing (never all cues at frame `0`)
- `single` output: render one full MP4
- `multi` output: only when the user requested clips; split by user rule and render one MP4 per clip

### 7) Keep reusable project artifacts

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
- `roles.json` file used for role reuse/initialization
- `prompts.json` file used for generation
- storyboard directory (if generated or used)
- narration audio file (if generated or used)
- subtitle SRT file (if generated or used)
- final rendered MP4 (single) or ordered clip list (multi)
- Remotion workspace directory
- Remotion `.gitignore` file path

Also report:

- extracted key segment summary (and whether it was user-locked or agent-abstracted)
- role prompt reuse summary (reused vs newly defined roles)
- what assumptions were clarified through interactive questions
- subtitle sync verification (first/middle/last cue spot check)
