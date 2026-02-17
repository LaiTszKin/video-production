---
name: video-production
description: Generate long-form videos (more than 10 minutes) by following user instructions and invoking related skills only when needed (`openai-text-to-image-storyboard`, `docs-to-voice`, `remotion-best-practices`, and `text-to-short-video` only for user-requested highlight clips). For text inputs, first extract a complete long-form story arc, split it into timed chapters, and require plan confirmation before generation.
---

# Video Production

## Core Rules

1. Follow user instructions first. Do not force a fixed pipeline.
2. For text-driven requests, use a long-form pipeline by default and target final output longer than 10 minutes.
3. Before generation, turn source text into a coherent long-form structure (hook, chapter progression, and closing) unless the user already provides an exact structure.
4. Role prompt policy for prompt generation:
   - always ensure `<project_dir>/pictures/<content_name>/roles.json` exists before any prompt generation
   - if a recurring role already has a defined prompt skeleton, reuse it without rewriting identity fields
   - if a recurring role appears but is not defined, generate new role entries using the supported `prompts.json` schema
5. Call only relevant dependency skills:
   - `openai-text-to-image-storyboard`: generate storyboard images when user did not provide usable visuals
   - `docs-to-voice`: generate narration/timeline/SRT when user did not provide audio or subtitles
   - `remotion-best-practices`: compose and render the final long-form output
   - `text-to-short-video`: use only when the user explicitly asks for teaser/highlight clips
6. Before any asset generation or rendering, create a plan markdown in `<project_dir>/docs/plans/` and wait for explicit user confirmation.
7. Never ask the user whether to output a single video or multiple episodes.
   - If the user explicitly asks for episodes/parts, produce multi-episode output.
   - Otherwise default to one full long video.
8. Keep Remotion project files unless the user explicitly asks for cleanup.
9. Keep git repositories clean by ensuring Remotion dependency/build/cache artifacts are ignored.

## Interactive Clarification (Required)

Before running generation commands, check whether details are sufficient. If not, ask concise targeted questions and wait for user replies.
Do not guess preference-sensitive options.

Prioritize asking these missing items:

- target total duration (default recommendation: 10-15 minutes when user says long video)
- chapter pacing (chapter count or average chapter duration)
- subtitle style (font, size, color, stroke/background, position, max lines)
- orientation/aspect ratio or exact resolution
- narration language/voice tone (if narration must be generated)
- visual style keywords (if storyboard images must be generated)
- existing role prompt source (user-provided prompt file or existing `<project_dir>/pictures/<content_name>/prompts.json`) when recurring roles are expected
- output location / filename rules when not obvious

Ask only the minimum set required to unblock progress.

## Inputs to Collect

Collect only what is needed for the requested job (not everything by default):

- `project_dir`
- `content_name`
- source script/text (unless the user provides complete audio + subtitles)
- extracted long-form story arc with chapter breakdown (unless user provides an exact chapter structure)
- final transcript text used for narration/subtitles
- existing prompt assets (`prompts.json`, character roster, or user-defined role prompts)
- user-provided assets (images, audio, subtitles, branding)
- output requirements from user instructions

## Workflow

### 1) Read user intent and choose skill calls

Create a concise execution checklist:

- what the user already provided
- what still needs generation
- whether to run the long-form text path or fallback asset path
- which dependency skills will be called
- what will be skipped

### 2) Abstract long-form story arc and chapter map (text-first path)

When the request includes source text and the user did not lock a specific structure:

- extract one self-contained long-form narrative arc suitable for a 10+ minute video
- split content into chapters with clear transitions and duration targets
- preserve original wording for key hooks and turning points whenever possible
- ensure narrative continuity (setup -> development -> escalation -> payoff/closing)
- keep this chaptered script as the primary input for downstream generation

If the user already provides an exact chapter structure, reuse it directly and skip abstraction.

### 3) Resolve role prompt reuse and ensure `roles.json` exists before prompt generation

Before generating or updating `prompts.json`:

1. detect recurring roles across all chapters
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
  - `## Long-Form Structure`
  - `## Timing Plan`
  - `## Video Transcripts`
  - `## Prompt Strategy`
  - `## Images To Generate`
- in `Long-Form Structure`, include the extracted structure (or the user-locked structure)
- in `Timing Plan`, include chapter-level duration budget and total runtime
- in `Prompt Strategy`, state which role prompts are reused vs newly defined and which JSON schema is used
- in `Video Transcripts`, include the transcript content that will be used for narration/subtitles
- in `Images To Generate`, list each image that still needs generation (scene id or order + prompt/description + intended usage)
- replace all square-bracket placeholders with concrete content
- remove all square-bracket placeholder/instruction text before sharing the plan for confirmation
- return the absolute plan file path to the user and ask for explicit confirmation
- do not run generation/render commands until user confirms the plan document

### 5) Generate the long-form video (default path)

After plan confirmation:

- generate missing storyboard assets chapter-by-chapter
- generate narration/subtitles chapter-by-chapter when needed
- compose into one continuous Remotion timeline by default
- verify total rendered runtime is longer than 10 minutes unless the user explicitly requested shorter

### 6) Optional short highlight clips (only when requested)

If the user explicitly asks for teaser/highlight outputs in addition to the long video:

- extract requested highlight segments from the approved long-form plan
- run `text-to-short-video` only for those highlight segments

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
- final long-form MP4 (single) or ordered episode list (multi)
- optional highlight clips (if requested)
- Remotion workspace directory
- Remotion `.gitignore` file path

Also report:

- extracted long-form structure summary (and whether it was user-locked or agent-abstracted)
- chapter duration summary and total runtime
- role prompt reuse summary (reused vs newly defined roles)
- what assumptions were clarified through interactive questions
- subtitle sync verification (first/middle/last cue spot check)
