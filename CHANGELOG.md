# Changelog

All notable changes to this project are documented in this file.

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
