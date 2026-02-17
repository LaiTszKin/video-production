# Video Production Skill

A Codex skill for long-form video production (more than 10 minutes) that follows user instructions and invokes only the needed dependency skills.

## Capabilities

- Adapts the workflow to user-provided assets instead of forcing a fixed pipeline.
- Uses a long-form chaptered workflow as the default text-to-video path.
- Abstracts a complete long-form story arc and chapter map before generation (unless user locks a structure).
- Ensures `roles.json` exists first (create if missing), then reuses existing recurring-role prompts and only defines missing roles.
- Calls storyboard generation only when visuals are missing.
- Calls voice/subtitle generation only when audio or SRT is missing.
- Uses Remotion best practices to compose and render final output.
- Uses `text-to-short-video` only when the user explicitly requests teaser/highlight clips.
- Asks interactive clarifying questions when key information is missing (for example subtitle style and target duration).
- Creates `<project_dir>/docs/plans/<YYYY-MM-DD>-<content_name>.md` from `references/plan-template.md` before generation and waits for user confirmation.
- Uses square-bracket placeholders in the plan template and removes all placeholders/instructions after filling.
- Defaults to one long-form video unless the user explicitly requests multi-episode output.
- Preserves the Remotion project by default for later edits.
- Maintains Remotion `.gitignore` (including `node_modules/` and build/cache outputs) to keep git projects clean.

## Dependency Skills

- `openai-text-to-image-storyboard`
- `docs-to-voice`
- `remotion-best-practices`
- `text-to-short-video` (optional, only for requested highlights)

## Typical Inputs

- `project_dir`
- source text (or existing assets)
- `content_name`
- existing role prompt source (optional `prompts.json` or role definitions)
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
- final rendered long-form MP4 (single) or ordered MP4 episode list (multi)
- highlight clip MP4s (if requested)
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
