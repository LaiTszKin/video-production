# Video Production Skill

A Codex skill for end-to-end text-to-video production. It supports both single long videos and multi-clip short-video outputs.

## Capabilities

- Plans scene prompts from source text in narrative order.
- Generates storyboard images with orientation-aware aspect ratio.
- Produces full narration audio, timeline JSON, and SRT subtitles.
- In multi-clip mode, splits full narration/subtitles by user-defined clip duration.
- Composes and renders subtitle video(s) with Remotion best practices.
- Preserves the Remotion project by default for future edits and re-rendering.
- Enforces one unified adaptive subtitle strategy across all rendered videos.
- Derives subtitle text color and depth from image color elements to keep captions readable.
- Keeps subtitle timing aligned cue-by-cue with narration (no all-at-once captions).

## Required Dependency Skills

1. `openai-text-to-image-storyboard`
2. `docs-to-voice`
3. `remotion-best-practices`

## Required Inputs

- `project_dir`
- source text (raw text or a text file)
- `content_name`
- orientation (`vertical` or `horizontal`)
- delivery mode (`single` or `multi`)
- target video duration (for `single`)
- segment duration per clip (for `multi`)

## Output Contract

- Storyboard folder: `<project_dir>/pictures/<content_name>/`
- Narration assets: `<project_dir>/audio/<content_name>/`
- Subtitle style profile: `<project_dir>/video/<content_name>/subtitle-style.json`
- Subtitle contrast report: `<project_dir>/video/<content_name>/subtitle-contrast-report.json`
- Single mode video: `<project_dir>/video/<content_name>.mp4`
- Multi mode videos: `<project_dir>/video/<content_name>/<content_name>-part-001.mp4` (ordered series)
- Remotion workspace (kept by default): `<project_dir>/video/<content_name>/remotion/`
- Multi mode manifest: `<project_dir>/video/<content_name>/segments.json`

## Files

- `SKILL.md` – workflow, dependency contract, and command examples.
- `agents/openai.yaml` – display metadata and default prompt.

## Quick Start

In Codex, call the skill directly:

```text
Use $video-production to convert text into one long video or multiple short narrated subtitle clips.
```
