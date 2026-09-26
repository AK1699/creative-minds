# creative-minds — Orchestrator

creative-minds is the parent of the Creative Minds ecosystem: an AI-powered
creative production system. Each capability lives in an independent git
submodule (a "forge"); this repo owns orchestration, contracts, and project
state — never creative implementation. Do not duplicate forge logic here.

## Architecture

```
                        CREATIVE-MINDS
                             │
                    ┌────────┴────────┐
                    │   Orchestrator  │   bin/creative-minds (deterministic Python)
                    └────────┬────────┘
                             │
                    Claude Code runtime    (stages run as claude -p in each module)
                             │
         ┌───────────────────┼───────────────────┐
         ↓                   ↓                   ↓
    STORY-FORGE         IMAGE-FORGE         VIDEO-FORGE   … voice, music
         │                   │                   │
         └───────────────────┼───────────────────┘
                             ↓
                     projects/<id>/          (shared context = project files)
```

**AI does creative reasoning; code does everything else** — orchestration,
state, validation, schemas, retries, file management. Project files are the
single source of truth; conversational memory never is.

## Layout

- `bin/creative-minds` — orchestrator CLI (dependency-free Python 3)
- `schemas/` — JSON schemas: the authoritative cross-module contracts
- `projects/` — generated project state, one directory per story (gitignored)
- `<forge>/` — submodules; a forge participates by declaring `module.json`
  at its root (see `schemas/module.schema.json`)

## Module contract

The orchestrator discovers modules by scanning submodules for `module.json`.
Each stage declares: a slash command (run headlessly in the module's own
directory with the project path as argument), the project files it reads, the
files it writes, and the schema its primary output must satisfy. Modules only
ever touch `projects/<id>/` paths named in their contract — never another
module's repo. Adding a new forge requires no changes to existing forges.

## Project state

`projects/STORY-NNNN/project.json` is the pipeline state machine
(stage → pending/running/passed/failed, attempts, rewrite rounds). Subdirs:
`context/ story/ scenes/ prompts/ images/ audio/ video/ reports/`.
Every stage is resumable and individually re-runnable from disk state.

## Usage

```bash
bin/creative-minds story create --auto "a 60-second emotional story for Gen-Z"
bin/creative-minds story status STORY-0001
bin/creative-minds story run STORY-0001          # resume pending stages
bin/creative-minds story run-stage STORY-0001 story-draft   # rerun one stage
bin/creative-minds modules
bin/creative-minds validate STORY-0001 concepts
```

Quality gates: every stage output is schema-validated (bounded retries with
the errors fed back). After `story-critic`, a `fail` verdict triggers a
bounded rewrite loop (max 2 rounds) before the project is marked
`needs_human`. `finalize` is deterministic — the orchestrator promotes the
gate-passed draft to `story/final.json`.

Input modes: the request can be a bare brief (`auto`), a premise (`idea`), or
a full narrative (`story`). The `context-analyzer` stage classifies it and
locks user-provided elements; in `story` mode the orchestrator skips
concepts/select-concept and the draft preserves the user's narrative.

## Current phase

Phase 1: contracts + story pipeline through validated final story. story-forge
is the only active module. image/voice/music/video-forge are empty stubs;
visual-forge is dormant (story-forge owns the visual bible per the spec). No
image generation yet.

## Submodule hygiene

- After committing inside a forge, update the pin here: `git add <forge> && git commit`.
- Fresh clone: `git clone --recurse-submodules <url>`.
