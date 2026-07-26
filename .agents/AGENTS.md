# Project Rules: Airshow Traffic Monitor

These rules govern agent behavior when working in this repository. They are tailored to the specific constraints and conventions of this project.

## Four Golden Rules (from project-seed-kit)
- **Gotchas are gold.** The traps an agent cannot infer from the code are the highest-value lines in this file. Record each one the day it bites.
- **Every line costs context.** This file is a map and a rulebook, not the documentation.
- **Stale is worse than absent.** An agent trusts this file completely; updating it is part of shipping, not housekeeping.
- **Write instructions, not descriptions.** "Run X before Y" beats "X is generally run before Y."

## General Context
- **Project Scope**: A cross-platform Electron desktop dashboard for monitoring airshow air traffic (ATC audio streams, YouTube live-feed grid, embedded FlightRadar24 browser panel).
- **Stack**: Electron, TypeScript (strict), React, electron-vite, electron-builder.
- **Service Type**: This is a DESKTOP app. It has no server backend, no datastore, and no health endpoints (commands like `down`, `migrate`, `health` are not applicable).

## Commands and Execution
- The `justfile` is the single source of truth for operational commands. Where possible, refer to its recipes (e.g., `just dev`, `just up`, `just test`, `just lint`, `just fmt`). 
- If `just` is not available in the shell, use the underlying `npm` scripts defined in `package.json` that correspond to the `justfile` contracts.
- **Node Environment**: Be aware that `ELECTRON_RUN_AS_NODE` might leak from IDE terminals. Strip it if invoking Electron raw commands.

## Architecture & Boundaries
- **`src/main/`**: Main process (lifecycle, scheme registration, IPC handlers).
- **`src/preload/`**: Context bridge (`index.ts`, `index.d.ts`). `contextIsolation` is ON, `nodeIntegration` is OFF.
- **`src/renderer/src/`**: React UI. Module folders: `audio/`, `youtube/`, `components/`, `state/`.
- **`src/shared/`**: Code compiled by all processes. Must be free of Electron and DOM APIs.
- **Dependencies**: The Phase 0 dependency set is locked. Do not add new dependencies without explicit user permission.
- **Consequence-bearing actions**: Ask before, never assume: pushing or opening PRs, tagging or releasing, deploying, destructive data operations, and adding new dependencies. Never print secrets or env-file contents. On failure: stop, surface the raw error, and do not auto-recover past it.

## Definition of Done
Before any change is complete, self-check this list:
- [ ] Tests pass at every tier the change touches (`just test`)
- [ ] Lint, format, and types clean (`just lint`, `just fmt`, `just typecheck`)
- [ ] Docs updated, if behavior or structure changed
- [ ] Changelog entry, if user-facing

## Documentation & Formatting Conventions
- **Semantic Line Breaks**: All markdown prose must use semantic line breaks (one sentence per line). List bullets stay on one line if a single sentence; otherwise, use a 2-space hanging indent.
  - **Exception**: `CODE_OF_CONDUCT.md` is strictly verbatim.
  - Prettier is configured to ignore markdown formatting for this exact reason. Do not alter markdown formatting with automated formatters.
- **Docs Split**: 
  - `docs/design/` holds implementation-agnostic architecture (what & why).
  - `docs/development/` holds tooling and implementation specifics (how).
- **Decision Stamps**: Any design or architectural decision must be stamped inline where it's made using the format `(decision YYYY-MM-DD)`. A corresponding one-line entry MUST be added to `docs/decisions/README.md` in the very same commit.

## Maintaining This File
- **A milestone lands**: Update Current State in docs.
- **Something non-obvious costs time**: Add to Code & Logic Gotchas (same day).
- **The tree or commands change**: Update Architecture/Commands sections.
- **A convention changes**: Update Conventions section.
- **Agent-file sync**: This file (`AGENTS.md`) and `CLAUDE.md` must change in the same commit to stay in sync.

## Code & Logic Gotchas
- **Renderer Serving Origin**: The packaged renderer loads from a loopback HTTP server (`http://127.0.0.1:<port>`), never `file://`. (To satisfy YouTube iframe API validation).
- **LiveATC Fetching**: Every `.pls` resolve and stream fetch must use a browser-like `User-Agent` (`src/main/http.ts`). The LiveATC search page is behind Cloudflare and requires Chromium's network stack (Electron's `net.request`) rather than standard `fetch`.
- **Audio Routing**: ATC mute is handled via a gain node set to `0`, *never* by setting `element.muted` (which would kill the VAD analyser signal). The VAD analyser taps PRE-gain.
- **Audio Engine Rendering**: Never use `requestAnimationFrame` in the audio engine since it freezes when hidden. Use `setInterval` (50 ms tick) and keep `backgroundThrottling: false`.
- **FR24 Browser View**: The `WebContentsView` for FR24 paints ABOVE all DOM. Overlays/modals cannot cover it—use `fr24:setVisible(false)` when opening overlays.

## Git Workflow
- **Merge Strategy**: Merge commits only. NEVER squash or rebase-merge, as `GitVersion` relies on the merge history to compute the version.
- **Branching**: Do not push to `main`. Create `feature/*`, `fix/*`, or `docs/*` branches and open PRs targeting `develop`.
