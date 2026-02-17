# Video Production Skill

A Codex skill for text-to-video production that follows user instructions and invokes only the needed dependency skills.

## Capabilities

- Adapts the workflow to user-provided assets instead of forcing a fixed pipeline.
- Uses `text-to-short-video` as the default text-to-video path.
- Abstracts the most important text segment before short-video generation (unless user locks a segment).
- Ensures `roles.json` exists first (create if missing), then reuses existing recurring-role prompts and only defines missing roles.
- Calls storyboard generation only when visuals are missing.
- Calls voice/subtitle generation only when audio or SRT is missing.
- Uses Remotion best practices to compose and render final output.
- Asks interactive clarifying questions when key information is missing (for example subtitle style).
- Creates `<project_dir>/docs/plans/<YYYY-MM-DD>-<content_name>.md` from `references/plan-template.md` before generation and waits for user confirmation.
- Uses square-bracket placeholders in the plan template and removes all placeholders/instructions after filling.
- Defaults to single-video output unless the user explicitly requests multi-clip output.
- Preserves the Remotion project by default for later edits.
- Maintains Remotion `.gitignore` (including `node_modules/` and build/cache outputs) to keep git projects clean.

## Dependency Skills

- `text-to-short-video`
- `openai-text-to-image-storyboard`
- `docs-to-voice`
- `remotion-best-practices`

## Typical Inputs

- `project_dir`
- source text (or existing assets)
- `content_name`
- existing role prompt source (optional `prompts.json` or role definitions)
- orientation / resolution
- subtitle style preferences
- duration constraints (if needed for the requested output)

## Output Contract

Return absolute paths for:

- plan markdown file
- roles.json file (used for role reuse/initialization)
- prompts.json file (if used)
- storyboard directory (if used)
- narration audio file (if used)
- subtitle SRT file (if used)
- final rendered MP4 (single) or ordered MP4 clip list (multi)
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
Use $video-production to generate the video according to my instructions.
```
