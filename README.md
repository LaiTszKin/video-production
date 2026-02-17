# Video Production Skill

A Codex skill for long-form video production (more than 10 minutes) that follows user instructions and invokes only the needed dependency skills.

## Capabilities

- Adapts the workflow to user-provided assets instead of forcing a fixed pipeline.
- Uses a long-form chaptered workflow as the default text-to-video path.
- Abstracts a complete long-form story arc and chapter map before generation (unless user locks a structure).
- Uses `text-to-short-video` only for selected climax scenes extracted from the source text.
- Integrates generated climax short clips into the final long-form video timeline (not short-clip-only delivery).
- Ensures `roles.json` exists first (create if missing), then reuses existing recurring-role prompts and only defines missing roles.
- Calls storyboard generation only when visuals are missing.
- Calls voice/subtitle generation only when audio or SRT is missing.
- Uses Remotion best practices to compose and render final output.
- Asks interactive clarifying questions when key information is missing (for example subtitle style, target duration, climax scene lock, and short-clip integration position).
- Creates `<project_dir>/docs/plans/<YYYY-MM-DD>-<content_name>.md` from `references/plan-template.md` before generation and waits for user confirmation.
- Uses a compact plan template with only four sections: meta data, reference text, images needed, and climax scene/video generation.
- Defaults to one long-form video unless the user explicitly requests multi-episode output.
- Preserves the Remotion project by default for later edits.
- Maintains Remotion `.gitignore` (including `node_modules/` and build/cache outputs) to keep git projects clean.

## Dependency Skills

- `openai-text-to-image-storyboard`
- `docs-to-voice`
- `remotion-best-practices`
- `text-to-short-video` (used only for climax-scene short clips in text-driven jobs)

## Typical Inputs

- `project_dir`
- source text (or existing assets)
- `content_name`
- existing role prompt source (optional `prompts.json` or role definitions)
- reference text sources (path/URL + scope notes; no full text embedding in plan)
- climax scene selection (optional lock if user wants exact scenes)
- climax short-clip integration position (default first clip at `00:00`)
- target duration (10+ minutes for long-form requests)
- chapter pacing preference
- orientation / resolution
- subtitle style preferences

## Output Contract

Return absolute paths for:

- plan markdown file
- roles.json file (used for role reuse/initialization)
- prompts.json file (if used)
- storyboard directory (if used)
- narration audio file (if used)
- subtitle SRT file (if used)
- climax-scene short clip MP4 list from `text-to-short-video` (text-driven jobs)
- final rendered long-form MP4 with integrated climax short clips
- additional climax clip MP4s (if requested)
- Remotion workspace directory
- Remotion `.gitignore` file path

## Files

- `SKILL.md` - workflow and execution rules
- `agents/openai.yaml` - display metadata and default prompt
- `references/plan-template.md` - pre-generation plan markdown template
- `references/roles-json.md` - recurring-role schema for `roles.json`

## Quick Start

In Codex, call the skill directly:

```text
Use $video-production to generate a long-form video based on my instructions.
```
