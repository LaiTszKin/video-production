# Video Production Skill

A Codex skill for text-to-video production that follows user instructions and invokes only the needed dependency skills.

## Capabilities

- Adapts the workflow to user-provided assets instead of forcing a fixed pipeline.
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

- `openai-text-to-image-storyboard`
- `docs-to-voice`
- `remotion-best-practices`

## Typical Inputs

- `project_dir`
- source text (or existing assets)
- `content_name`
- orientation / resolution
- subtitle style preferences
- duration constraints (if needed for the requested output)

## Output Contract

Return absolute paths for:

- plan markdown file
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

## Quick Start

In Codex, call the skill directly:

```text
Use $video-production to generate the video according to my instructions.
```
