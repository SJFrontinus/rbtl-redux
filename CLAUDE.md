# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file HTML app (`index.html`) — a metronome timing visualizer used for tick-relative event offset debugging. No build step, no dependencies, no framework. Open directly in a browser or serve statically (deployed on Render).

## Running locally

```bash
# Any static file server works, e.g.:
bun x serve .
# or just open index.html directly in a browser
```

## Architecture

Everything lives in `index.html` — CSS variables, layout, and JavaScript in one file. No modules, no bundler.

**Core subsystems:**

- **Rotary knobs** (`makeKnob`): Canvas-drawn, drag-to-change (vertical pointer delta). Two knobs: Speed (0.5–4 t/s) and Offset (0–1 as fraction of tick interval).

- **Web Audio scheduler** (lines ~476–560): Classic lookahead pattern — a `setInterval`-equivalent (`SCHED_MS = 25ms`) fires `schedulerLoop`, which schedules audio clicks via Web Audio API up to `LOOKAHEAD = 0.12s` ahead. Timing is anchored to `audioCtx.currentTime`, not `Date.now()`, for drift-free precision.

- **Countdown** (lines ~484–496): 3-beat countdown fires before tick 0. `countdownStartT` is when beat "3" sounds; `startAudioT = countdownStartT + 3 * tickSec()` is tick 0.

- **Light state machine**: Each light cycles `dark → yellow → green|red`. Yellow means "anticipation window" — turns on `offsetSec()` before the audio click. On the last tick, a phantom beat resolves yellow after one full tick interval.

- **Hit detection** (`handleHitClick`): Pointer events anywhere on the page; finds closest yellow light within `HIT_RADIUS = 80px`. Hit = green, miss = red.

- **Light layout** (`buildLights` + `lightPositions[]`): Current pattern is "zigzag" — even-indexed ticks (0,2,4…) in left column, odd-indexed (1,3,5…) in right column, 2-column CSS grid. Position metadata in `lightPositions[]` array of `{ col, row }` for future layout pattern work.

**Key state variables:**
- `ticksPerSec`, `totalTicks`, `offsetFrac` — control parameters
- `lightStates[]` — per-tick color state (`'dark'|'yellow'|'green'|'red'`)
- `lightHits[]` — whether user clicked during each yellow window
- `schedTick`, `schedCountdown` — scheduler lookahead cursors
