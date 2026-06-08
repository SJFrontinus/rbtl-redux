# Plan (Beginning): Add the "Read Between the Lions" Exercise

*Drafted 2026-06-01. This is the front matter for a future planning/build session — organized instructions to Claude, not yet a finalized implementation plan. Companion thinking notes: `thinking-new-exercise-2026-06-01.md`.*

---

## 1. Goal

Add a second exercise to the existing single-file app (`index.html`) — **Read Between the Lions (RBTL)**, an eye-movement training drill for readers (esp. young readers). The app becomes a chooser: the existing dot-timing game is one exercise; RBTL is the other. **Reuse the existing metronome/timing engine as much as possible** — RBTL is "the same process, with words as targets instead of dots."

The training effect: load a page of text; on each rendered line, two highlighted target words pace the reader's eye **left → right → next line**, two controlled saccades per line, on the beat. Content/comprehension is not scored — paced traversal is the point.

> **See the mockup first:** `rbtl-mockup-marked.png` (repo root) shows the intended result — two target words circled per line on the sample text. That image is the ground-truth statement of the mechanic. Sample text for testing: `rbtl-text-sample.txt`.

---

## 2. What Gets Reused (do NOT rebuild)

From `index.html`:
- **Web Audio lookahead scheduler** (`schedulerLoop`, `SCHED_MS`, `LOOKAHEAD`) — the drift-free timing spine.
- **3-beat countdown** logic (`countdownStartT`, `startAudioT`).
- **Color state machine** — `dark → yellow → green|red` (yellow = anticipation window, opens `offsetSec()` before the tick).
- **Hit detection pattern** (`handleHitClick`) — closest active target within a hit radius; hit→green, miss/late→red.
- **Knobs** — Speed (t/s) and Offset (anticipation lead as fraction of tick).
- Transport / start controls and overall page chrome.

The job is to **generalize the engine** so the targets it drives can be either dots (existing layout) or words (new layout), without forking the timing code.

---

## 3. What's New (RBTL-specific)

1. **Exercise selector** — pick "Timing (dots)" vs "Read Between the Lions". Existing game keeps its current behavior unchanged.
2. **Text input** — load text from a file. Start with bundled sample(s) (the LPA sample first) + a `.txt` file picker. Paste box optional later.
3. **Reading pane** with **adjustable font size and column width** — these are difficulty levers (they re-wrap the text and therefore move the targets).
4. **Post-render line detection** — after the text is laid out, measure where it visually wrapped (Range/`getClientRects` over word spans, group by vertical position) to reconstruct the on-screen lines.
5. **Randomized target selection** — per visual line, pick a random word in the left half and a random word in the right half; re-rolled every run. Highlights rendered as a **background color behind the word** (yellow → green/red).
6. **Target → tick sequencing** — flatten targets in reading order (line0-left, line0-right, line1-left, line1-right, …), one target per metronome tick (⇒ 2 ticks/line), driven by the reused scheduler.
7. **Word hit detection** — tap/click the active target word; in-time + within hit zone = green, otherwise red.
8. **Pagination** — "number of lines" = total lines in the exercise (default ~20). If more lines than fit the window, split into pages; **re-run the 3-count lead-in at each page break** (with a short pause).
9. **Distinct sounds** — the 3-count lead-in uses a *different* tone from the in-progress metronome tick (at the very start and at every page break). Apply this distinction to the dot game too.

---

## 4. Difficulty Levers (all compose)

Speed (knob) · Offset / anticipation window (knob — smaller = harder) · Font size · Column width · Number of lines (length/pages).

---

## 5. Suggested Build Phases

> Order chosen so the app keeps working at every step; the risky rendering-measurement work is isolated.

**Phase 0 — Engine extraction & exercise selector.**
Refactor `index.html` so the scheduler / countdown / color-state / sound code is shared, and the *target source* + *rendering* are pluggable. Add the exercise chooser. **Acceptance:** dot game behaves exactly as before, now selected from a menu.

**Phase 1 — Text pipeline (static).**
Load a bundled sample into an adjustable reading pane (font size + width controls). Measure rendered lines. Pick randomized left/right targets per line and paint static yellow highlights (no timing yet). **Acceptance:** open the LPA sample, see two highlighted words per visual line; change width/font and watch targets re-pick.

**Phase 2 — Wire to the metronome.**
Feed the target sequence into the reused scheduler; run the color state machine on word highlights; enable tap/click hit detection on words. **Acceptance:** a single-screen run plays start-to-finish with green/red feedback, paced by Speed/Offset.

**Phase 3 — Pagination + per-page lead-in.**
Honor "number of lines"; split into pages by viewport fit; pause and re-run the 3-count at each break. **Acceptance:** a 50-line run spans multiple pages with clean lead-ins between them.

**Phase 4 — Distinct lead-in vs metronome sounds.**
Add a second timbre/pitch for the lead-in; apply to both exercises. **Acceptance:** lead-in is audibly different from the running tick.

**Phase 5 — Polish & content.**
Hit-zone tuning, edge-case handling, more sample texts (kid-friendly stories), settings layout. 

---

## 6. Open Decisions (proposed defaults — confirm during build)

| Topic | Proposed default |
|---|---|
| **Per-line timing** | 2 targets/line ⇒ 2 ticks/line; left fires before right. Offset = yellow anticipation lead. |
| **Hit zone** | Word bounding box + ~12px padding; min radius for very short words. |
| **Edge-case lines** | ≥2 words ⇒ 2 targets; 1 word ⇒ 1 target; blank lines skipped and not counted toward "number of lines." |
| **Headings** | Treated as normal lines for target purposes (tunable later). |
| **File format** | Plain `.txt`, paragraphs separated by blank lines. Bundle `rbtl-text-sample.txt` + a story or two. |
| **Layout lock** | Freeze font size/width at Start so targets don't shift mid-run; resize is a between-runs lever. |
| **Re-roll** | Targets randomized fresh on each Start. |
| **Page-break pause** | Short pause then full 3-count; exact pause length TBD by feel. |

---

## 7. Known Technical Risks / To Watch

- **Rendered-line measurement** is the trickiest part — wrapping detection must be reliable across fonts/widths. Wrap each word in a measurable span and group by vertical offset.
- **Mid-run resize** would invalidate target positions → lock layout once a run starts.
- **Touch vs mouse** — design hit detection for both pointer types (tablet is a primary target device for kids).
- **Tiny/last page** — a final page with very few lines should still get a lead-in and end cleanly.

---

## 8. Out of Scope (for now)

Comprehension testing, scoring/streak persistence, multi-user profiles, authored story library/CMS. Note them as future directions; don't build yet.
