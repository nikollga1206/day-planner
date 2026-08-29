# AGENTS.md

Guidance for AI coding agents working in this repository.

## Project overview

**day-planner** — a daily planner primarily for children. The day is represented by a clock face, and tasks are represented by segments on that clock face (see `README.md`).

## Current state of the repository

The repository contains:

- `README.md` — a two-line project description.
- `planner.html` — the entire application: a single self-contained HTML file (HTML+CSS+JS, no build step, no frameworks, no dependencies). Interface language: Russian.
- `Planner_prompt_p.1_core.txt` — the original spec the app was built to.
- `planner_clock.txt` — reference day schedule (task times and colors).
- `Clock.png` — visual reference for the spiral layout, colors, and label style.

## Build and test commands

No build step. Open `planner.html` directly in a browser.

Self-test: open `planner.html#selftest` — this loads the reference day and runs the control test (angles, overlaps, 1440-minute coverage, seam-split sleep segment), printing results to the browser console. Headless check (Chrome):

```
chrome --headless=new --enable-logging=stderr --v=0 --virtual-time-budget=3000 \
  --dump-dom "file:///.../planner.html#selftest" 2>&1 | grep CONSOLE
```

All 10 checks must print ✅.

## Code organization

Everything lives in `planner.html`, in logical sections (searchable by comment banners):

- Geometry constants and time/angle utilities (`spiralCoord`, `timeToAngle`, `pointToTime`, `sectorPath`). Key rule: seam at 04:00 = 120° clockwise from top, 1 hour = 30°; inner disk = 04:00–16:00, outer ring = 16:00–04:00. Tasks crossing a level boundary are split into same-colored pieces by `piecesOf`. Radii: `R_INNER = 300` (inner disk), `R_BAND = 360`, `R_OUTER = 470`; the ring R_INNER..R_BAND is a white band between the disk and the outer task ring. `pointToTime` maps a drop onto the white band to the nearest level (threshold at (R_INNER+R_BAND)/2). `piecesOf` returns [] for invalid tasks (bad start/duration), so broken data cannot break the render.
- State: `state = { schedule, library }`, persisted to `localStorage` (key `dayPlannerState.v1`). Both `loadState` and `importJSON` validate tasks via `sanitizeSchedule` (start must parse as HH:MM, duration must be a finite number, clamped to 5..1440 min); invalid tasks are skipped and the import message reports the counts ("Импортировано N задач, пропущено некорректных: M").
- SVG rendering (`renderSchedule`, `renderTaskLabels`, `renderTimeLabels`). The seam at 04:00 is drawn as a visible radial step (R_INNER→R_OUTER) on top of the segments. Rendering is failure-safe: the background (disk + white band + outer ring) is drawn first, each task segment renders inside try/catch, and label rendering is wrapped in try/catch too — the spiral stays drawn even if some task or label fails, and the user gets a message. Task labels are straight text "emoji + name" rotated by (mid-angle − 90°), flipped if upside down; font auto-fit ladder: default → smaller + condensed (`textLength`) → emoji only (narrow segments, ~15 min or less) → leader-line callout outside the circle (when even the emoji does not fit, arc width < ~20 px: a pointer line from the outer edge and horizontal "emoji + name" text). Time labels (hour marks + task boundaries): inner-level times (04:00–16:00) sit INSIDE the white band at radius (R_INNER+R_BAND)/2 as horizontal text without leader lines; outer-level times sit strictly OUTSIDE the circle at R_OUTER+116 with leader lines from R_OUTER+70. Their font is smaller than task labels, with a white halo (`paint-order: stroke`).
- Schedule tasks carry an internal `emoji` field (from the library by name, fallback "⭐"); it is persisted to `localStorage` but NOT included in export JSON.
- Library panel UI, pointer-based drag-and-drop (mouse + touch), segment edit panel. The segment edit panel has "Начало" and "Окончание" fields (not duration): each is a pair of `<select>`s — hours 0–23 and minutes in steps of 5 (`fillTimeSelects` fills them in `initUI`); crossing midnight is allowed (end earlier than start = next day), internally the task stays start + duration_min. Library items are edited in a modal (`#libedit-overlay`, opened by the ✏️ button) with name, default duration, color palette, and emoji (palette of 24 + manual input); the creation form (`#lib-form`) has the same emoji palette (`EMOJI_PALETTE`, `buildEmojiPalette`).
- Import/export JSON — the format is documented by a comment at the top of the script; an external reminder agent will read this file, so do not change the structure.
- Reference day (`REFERENCE_DAY`, from `planner_clock.txt`) and `runSelfTest()`.

## Code style guidelines

Vanilla JS (`"use strict"`, ES2017-era features, no modules). 2-space indent, Russian UI strings, English identifiers, comments in Russian. Keep the file self-contained: no external dependencies.

## Testing instructions

No test framework. Verification = the built-in self-test (see above) plus manual interaction checks in a browser. Pure logic functions are DOM-free and can be exercised headlessly.

## Security considerations

Nothing project-specific yet. Standard rules apply: do not commit secrets, and keep dependencies pinned once a package manager is adopted.

## Maintenance of this file

Keep this file current: whenever a build system, directory layout, convention, or workflow is added or changed, update the corresponding section above.
