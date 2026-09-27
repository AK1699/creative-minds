---
description: Create a story interactively — suggest suitable image styles, then run the pipeline
---

The user's story request: $ARGUMENTS

You are the interactive front-end to the Creative Minds pipeline. Follow these
steps in order:

1. If $ARGUMENTS is empty, ask the user what story they want (anything from a
   one-line brief to a full narrative works — the pipeline's context analyzer
   handles all input modes).

2. Read `config/styles.json` and shortlist the styles that genuinely fit the
   request's mood, audience, era and subject — judge by each style's
   `best_for`, category and rendering character. Use AskUserQuestion to offer
   3–4 shortlisted styles (label = slug, description = why it suits THIS
   story), with the best fit first and marked "(Recommended)". Include the
   default `vintage-watercolor-storybook` only when it genuinely fits. The
   user can always answer "Other" with a different slug or ask to see more —
   `bin/creative-minds styles` lists all 145.

3. Launch the pipeline in the background (it takes ~20–30 minutes):
   `bin/creative-minds story create --style <chosen-slug> --auto "<request>"`
   Quote the request safely. Tell the user the new project id immediately.

4. Monitor progress (tail the background output; stage lines contain
   `stage:`, `✓`, `✗`, `↻`). Report meaningful events briefly: stage passes,
   validation retries, critic verdicts. Do not spam every heartbeat.

5. When the pipeline completes:
   - Summarise the story (title, logline, relationship choice, critic scores)
     from `projects/<id>/story/final.json` and `story/critique.json`.
   - Export the manual generation prompts:
     `cd image-forge && .venv/bin/python -m imageforge export-prompts ../projects/<id>`
   - Tell the user where everything is: `reports/manual-prompts.md` for
     hand-generation, `images/incoming/` for dropping results, then
     `ingest` + `captions` (see image-forge/README.md).

6. If the pipeline fails or lands in `needs_human`, show
   `bin/creative-minds story status <id>` plus the failing stage's notes and
   ask the user how to proceed — do not silently retry the whole pipeline.

Never edit project files yourself — the pipeline owns them. Your job is
style guidance, launching, monitoring and reporting.
