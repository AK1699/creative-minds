# Shared Context

The single interface between forges. Every forge reads its inputs from here and
writes its outputs back here — never directly from another forge's repo.

```
shared-context/
├── brief.md      # the project brief — every pipeline run starts here
├── story/        # story-forge output: script, scenes, shot list
├── visuals/      # visual-forge output: style guide, storyboard
├── images/       # image-forge output: stills per shot
├── voice/        # voice-forge output: narration/dialogue audio
├── music/        # music-forge output: score, sound design
└── video/        # video-forge output: final assembly
```
