# Changelog

All notable changes to this project are documented in this file.

## [0.3.0] - 2026-02-17

### Added
- Add `text-to-short-video` as the default text-first production dependency.
- Require extracting the most important text segment before short-video generation unless the user locks an exact segment.
- Add recurring-role prompt policy: reuse existing role skeletons and create new JSON prompt entries only for undefined roles.
- Add `Key Segment` and `Prompt Strategy` sections to `references/plan-template.md`.
- Add `prompts.json` path to the output contract.

### Changed
- Update `SKILL.md`, `README.md`, and `agents/openai.yaml` to enforce text abstraction plus role-aware prompt reuse before generation.

## [0.2.0] - 2026-02-16

### Added
- Require generating a pre-production plan markdown in `<project_dir>/docs/plans/` named with `<YYYY-MM-DD>-<content_name>.md`.
- Define mandatory plan sections for `Video Transcripts` and `Images To Generate`.
- Add `references/plan-template.md` as the canonical pre-generation plan template.
- Add square-bracketed guidance and placeholders in the template, including explicit instruction to remove them after filling.
- Add plan markdown path to the output contract.

### Changed
- Enforce explicit user confirmation of the plan document before running any generation or rendering step.
- Require creating plan files from the reference template before generation.
- Require replacing and removing all square-bracket placeholder/instruction text before sending the plan for user confirmation.
- Update the agent default prompt and README to reflect the template-driven plan-first workflow.

## [0.1.3] - 2026-02-15

### Added
- Require preserving the Remotion workspace by default so users can revise and re-render later.
- Add Remotion workspace path to output locations and output contract.

### Changed
- Update agent default prompt to explicitly keep the Remotion project unless cleanup is requested.

## [0.1.2] - 2026-02-15

### Added
- Require subtitle color and depth to adapt to image color elements in the subtitle-safe area.
- Add subtitle readability verification with minimum contrast target (`4.5:1`) and fallback backing plate rule.
- Add `subtitle-contrast-report.json` to the output contract for traceable subtitle style decisions.

### Changed
- Replace fixed subtitle text/stroke/shadow defaults with an adaptive color/depth strategy.
- Update README and agent default prompt to emphasize image-aware subtitle readability behavior.

## [0.1.1] - 2026-02-15

### Added
- Require explicit user confirmation before any video production starts.
- Define a unified subtitle style profile shared by single and multi outputs.
- Add subtitle segment processing rules (overlap filtering, clamping, and local time rebasing).

### Changed
- Enforce cue-by-cue subtitle timing in Remotion rendering.
- Extend output contract to include subtitle style config path and subtitle sync verification.
- Update README and agent default prompt to reflect subtitle style and sync requirements.

## [0.1.0] - 2026-02-15

### Added
- Initial `video-production` skill release.
- End-to-end workflow across storyboard generation, narration/subtitles, and Remotion rendering.
- Support for both single long video output and multi-clip segmented output.
