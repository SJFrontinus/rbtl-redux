# Thinking: Adding a New Exercise to rbtl-redux

*Started 2026-06-01 — thinking-partner session*

## The Goal (as stated)
- Add a **new exercise** alongside the existing "basic exercise" in `index.html`.
- This session: develop the *beginnings of a plan* — organize instructions to Claude before going into planning/build mode. Be more complete than a one-line ask.

## What Exists Now
- `index.html` — single-file "Metronome Timing Visualizer." No build, no deps.
- **Basic exercise** = tick-relative hit-detection game: lights cycle dark→yellow→green/red; click during yellow anticipation window; Web Audio scheduler for drift-free timing.
- Subsystems: rotary knobs (Speed, Offset), Web Audio lookahead scheduler, 3-beat countdown, light state machine, hit detection, zigzag light layout.
- `rbtl-text-sample` (untracked) — dense prose on Lp(a) vs LDL atherogenicity. Likely *content* for the new exercise.

## Resolved — The Exercise (clear now)
- **RBTL = "Read Between the Lions"** — a Lions Club literacy project. Cute name, not literal.
- A reading-improvement exercise for **eye-movement training**, esp. *young readers*. Content/comprehension is NOT the point — it's about traversing a page of text in a paced, rhythmic fashion.
- **Mechanic:** load a page of text → on each line pick **two target words** (one ~left half, one ~right half) → those words get the **same 3-color sequence as the dots**, rendered as a **background highlight behind the word**: yellow (anticipation) → green (hit) / red (miss).
- The reader's eye is paced **left target → right target → next line left → right …** = two controlled saccades per line, on the beat. *This rhythmic traversal is the training.*
- **"Almost identical in process to the existing program — words as targets instead of dots."**
- **Reuses the metronome engine wholesale**: scheduler, 3-beat countdown, color state machine, Speed/Offset knobs all carry over.
- **One app, exercise picker.** Existing timing game = one exercise; RBTL = another.

## Decisions (locked this session)
1. **Real tap/click.** Reader attempts to tap/click each target word in rhythm. Too late OR too far = **red (miss)**; in time + close enough = **green (hit)**. Same hit-detection spirit as the dot game (a hit zone around the target).
2. **Targets selected post-render.** Lay out the text first, detect visual line wraps, then pick the left/right target per visual line. **Layout size + width are user-adjustable and are themselves a difficulty lever** — wider/narrower or bigger/smaller font re-wraps the text, moving the targets.
3. **"Number of lines" setting** (carried over UI style). If more lines than fit the window → **paginate**. At each page break, **re-run the 3-count lead-in** (likely with a pause). May need experimentation on the break pause.
4. **Distinct sounds:** the **3-count lead-in** gets a *different sound* from the **in-progress metronome tick** — applies both at the very start and at every pagination break. (New requirement, applies conceptually to the dot game too.)

## Decisions (cont.)
5. **Target selection = RANDOMIZED.** Each run, pick a random word in the left half and a random word in the right half of each visual line; re-rolled every Start. (Anti-memorization; variety.)
6. **"Number of lines" = total lines in the whole exercise.** Default ~20 (usually fits one page). User can choose 50 / 100+ for a multi-page challenge. Good story content is the motivator for longer runs.

## Proposed Defaults (to confirm during build, not blocking)
- **Per-line timing:** 2 targets/line ⇒ 2 ticks/line; left target fires before right. Offset knob keeps its anticipation-lead role.
- **Hit zone:** word bounding box + ~12px padding (min radius for tiny words).
- **Edge-case lines:** ≥2 words ⇒ 2 targets; 1 word ⇒ 1 target; blank lines skipped, not counted toward "number of lines."
- **File loading:** start with bundled sample text(s) (LPA sample first) + a file picker for .txt; paste optional later.
- **Layout lock:** freeze layout (font size/width) at Start so targets don't shift mid-run; resizing is a between-runs difficulty lever.

## Difficulty Levers (emerged)
Speed (knob) · Offset/anticipation window (knob) · Font size · Column width · Number of lines. All compose.

## Deliverable
- Plan doc written to repo: `PLAN-rbtl-exercise.md` — organized instructions to Claude (goal, reuse, new work, phased build, open decisions w/ defaults).

## Connections
- Classic reading-fluency tech: metronome pacing, guided fixation points, reducing *regressions* (backward eye movements), training steady left-to-right saccades. Two-fixations-per-line is a recognized fluency drill.

## Next Steps
- (only if they arise naturally)
