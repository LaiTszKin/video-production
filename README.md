# Video Production Skill

A Codex skill for end-to-end text-to-video production. It turns text into storyboarded, narrated videos with subtitle overlays and renders final MP4 output via Remotion.

## Capabilities

- Plans scene prompts from source text in narrative order.
- Generates storyboard images with orientation-aware aspect ratio.
- Produces narration audio, timeline JSON, and SRT subtitles.
- Composes and renders subtitle video with Remotion best practices.

## Required Dependency Skills

1. `openai-text-to-image-storyboard`
2. `docs-to-voice`
3. `remotion-best-practices`

## Required Inputs

- `project_dir`
- source text (raw text or a text file)
- `content_name`
- target video duration
- orientation (`vertical` or `horizontal`)

## Output Contract

- Storyboard folder: `<project_dir>/pictures/<content_name>/`
- Narration assets: `<project_dir>/audio/<content_name>/`
- Final video: `<project_dir>/video/<content_name>.mp4`

## Files

- `SKILL.md` – workflow, dependency contract, and command examples.
- `agents/openai.yaml` – display metadata and default prompt.

## Quick Start

In Codex, call the skill directly:

```text
Use $video-production to convert text into a narrated subtitle video.
```
