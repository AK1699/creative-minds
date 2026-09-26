# creative-minds — Orchestrator

creative-minds is a repo of repos. It orchestrates six specialist "forge" submodules
through the Claude Code runtime to turn a creative brief into a finished video.

## Architecture

```
                        CREATIVE-MINDS
                             │
                    ┌────────┴────────┐
                    │   Orchestrator  │
                    └────────┬────────┘
                             │
                    Claude Code runtime
                             │
         ┌───────────────────┼───────────────────┐
         ↓                   ↓                   ↓
    STORY-FORGE         IMAGE-FORGE         VIDEO-FORGE
         │                   │                   │
         ↓                   ↓                   ↓
      Story               Images              Video
         │                   │                   │
         └───────────────────┼───────────────────┘
                             ↓
                      Shared Context
```

## Forges

| Submodule      | Role                                        | Output                         |
| -------------- | ------------------------------------------- | ------------------------------ |
| `story-forge`  | Narrative: script, scenes, shot list        | `shared-context/story/`        |
| `visual-forge` | Visual direction: style, palette, storyboard| `shared-context/visuals/`      |
| `image-forge`  | Still image generation per shot             | `shared-context/images/`       |
| `voice-forge`  | Narration / dialogue audio                  | `shared-context/voice/`        |
| `music-forge`  | Score and sound design                      | `shared-context/music/`        |
| `video-forge`  | Final assembly: images + audio → video      | `shared-context/video/`        |

## Pipeline

1. A project starts as a brief in `shared-context/brief.md`.
2. **story-forge** turns the brief into a script and shot list.
3. **visual-forge** defines the look (style guide, storyboard) from the story.
4. **image-forge** generates stills for each shot, following the style guide.
5. **voice-forge** and **music-forge** produce audio from the script (can run in parallel with image-forge).
6. **video-forge** assembles images and audio into the final video.

Each forge reads its inputs from `shared-context/` and writes its outputs back
there — forges never read from each other's repos directly. `shared-context/`
is the only interface between stages.

## Working in this repo

- Each forge is a git submodule pinned to a commit. After committing inside a
  forge, update the pin here: `git add <forge> && git commit`.
- Fresh clone: `git clone --recurse-submodules <url>`.
- Pull latest across all forges: `git submodule update --remote --merge`.
- The forges are currently empty scaffolds; this file defines the contract
  they implement.
