---
type: entity
entity_type: dataset
name: PointOdyssey
status: stub
tags: [point-tracking, benchmark, long-sequence, synthetic-data]
sources:
  - "[[karaev-2024-cotracker]]"
  - "[[karaev-2024-cotracker3]]"
  - "[[aydemir-2025-track-on2]]"
  - "[[jung-2026-tapnext-plus-plus]]"
related:
  - "[[point-tracking]]"
  - "[[adam-w-harley]]"
created: 2026-05-24
updated: 2026-05-24
---

# PointOdyssey

## Composition

Long-form synthetic [[point-tracking]] benchmark introduced alongside
PIPs++ (Zheng et al., 2023). Sequences average **~2,400 frames** (up to
4,325) — orders of magnitude longer than TAP-Vid DAVIS or Kinetics. The
primary benchmark for testing **long-horizon** tracking.

## Why it matters

This is where the **online-vs-windowed gap closes or opens**.
Window/video methods run out of memory or accumulate drift on
PointOdyssey sequences (per [[aydemir-2025-track-on2]] Fig 3: video-level
methods OOM); per-frame causal trackers ([[track-on2]],
[[tapnext-plus-plus]]) sustain performance.

## Metrics commonly reported

- Standard AJ + δ_avg + OA.
- **Survival rate** — average number of frames until tracking failure.
- **AJRD** (new metric from [[jung-2026-tapnext-plus-plus]]) — measures
  re-detection after occlusion.

## Papers using it (in this wiki)

- [[karaev-2024-cotracker]] — eval (CoTracker reports here).
- [[karaev-2024-cotracker3]] — used in teacher pool / eval.
- [[aydemir-2025-track-on2]] — major eval target; where memory-based
  methods shine.
- [[jung-2026-tapnext-plus-plus]] — major eval target; introduces AJRD on
  this dataset.

## Known biases / caveats

- Synthetic — same sim-to-real concerns as Kubric, plus longer-form
  rendering artifacts.
- Trajectory distribution favors smooth long motion; sharp re-entries are
  underrepresented (motivated [[jung-2026-tapnext-plus-plus]]'s roll
  augmentation).

## Notes

Co-introduced with PIPs++ — promote to its own method page when PIPs++
is ingested. [[adam-w-harley]] connection.
