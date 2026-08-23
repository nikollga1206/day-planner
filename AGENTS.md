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

- Geometry constants and time/angle utilities (`spiralCoord`, `timeToAngle`, `pointToTime`, `sectorPath`). Key rule: seam at 04:00 = 120° clockwise from top, 1 hour = 30°; inner disk = 04:00–16:00, outer ring = 16:00–04:00. Tasks crossing a level boundary are split into same-colored pieces by `piecesOf`.
- State: `state = { schedule, library }`, persisted to `localStorage` (key `dayPlannerState.v1`).
- SVG rendering (`renderSchedule`, `renderTimeLabels`).
- Library panel UI, pointer-based drag-and-drop (mouse + touch), segment edit panel.
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
