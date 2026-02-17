# Video Production Plan

[Instruction: Copy this template to `<project_dir>/docs/plans/<YYYY-MM-DD>-<content_name>.md` before any generation starts.]
[Instruction: Replace every square-bracket placeholder with real content.]
[Instruction: After filling, delete all square-bracket instruction/placeholder text before showing the plan to the user for confirmation.]

# [VIDEO_TITLE]

- Date: [YYYY-MM-DD]
- Content Name: [CONTENT_NAME]
- Aspect Ratio / Resolution: [ASPECT_RATIO_OR_RESOLUTION]
- Target Duration (minutes): [TARGET_DURATION_MINUTES]
- Output Mode: [SINGLE_OR_MULTI_EPISODE]

## Long-Form Structure

[Instruction: For text-driven long-form jobs, paste the extracted story arc and chapter outline used for the 10+ minute video. If the user locked the structure, paste that exact structure and mark it as user-locked. If this is not text-driven, write `None`.]
[STORY_ARC_AND_CHAPTER_OUTLINE]

## Timing Plan

[Instruction: Provide chapter-level duration planning and total runtime. For long-form jobs, total should be at least 10 minutes unless the user explicitly requested shorter.]
1. [CHAPTER_ID_OR_ORDER] - [DURATION_MM_SS] - [CHAPTER_PURPOSE]
2. [CHAPTER_ID_OR_ORDER] - [DURATION_MM_SS] - [CHAPTER_PURPOSE]
3. [CHAPTER_ID_OR_ORDER] - [DURATION_MM_SS] - [CHAPTER_PURPOSE]
- Total Planned Runtime: [TOTAL_DURATION]

## Video Transcripts

[Instruction: Paste the final transcript used for narration/subtitles. Keep exact wording if timing depends on it.]
[TRANSCRIPT_TEXT]

## Prompt Strategy

[Instruction: Describe how role prompts are reused or newly defined, and which supported JSON prompt schema is used.]
- Prompt Source Priority: [USER_PROVIDED_OR_EXISTING_FILE_OR_NEW]
- Reused Roles: [ROLE_IDS_OR_NONE]
- Newly Defined Roles: [ROLE_IDS_OR_NONE]
- Prompt JSON Format: [FORMAT_A_OR_FORMAT_B]

## Images To Generate

[Instruction: List only images that still need generation. If no image generation is needed, write `None`.]
1. [SCENE_ID_OR_ORDER] - [IMAGE_PROMPT_OR_DESCRIPTION]
   - Usage: [INTENDED_USAGE]
2. [SCENE_ID_OR_ORDER] - [IMAGE_PROMPT_OR_DESCRIPTION]
   - Usage: [INTENDED_USAGE]

## Notes

[Instruction: Record assumptions clarified with the user (for example subtitle style, voice tone, duration constraints).]
[ASSUMPTIONS_AND_CONSTRAINTS]
