---
type: entity
entity_type: dataset
name: TAP-Vid
status: growing
tags: [point-tracking, benchmark, video-understanding]
sources:
  - "[[zholus-2025-tapnext]]"
  - "[[karaev-2024-cotracker]]"
  - "[[karaev-2024-cotracker3]]"
  - "[[aydemir-2025-track-on2]]"
  - "[[jung-2026-tapnext-plus-plus]]"
related:
  - "[[point-tracking]]"
  - "[[tapnext]]"
  - "[[cotracker]]"
  - "[[track-on2]]"
  - "[[kubric-dataset]]"
  - "[[tapvid-3d-dataset]]"
created: 2026-05-24
updated: 2026-05-24
---

# TAP-Vid

The benchmark for [[point-tracking]] (Tracking Any Point in video).
Introduced by Doersch et al. (NeurIPS 2022) — not yet ingested as its own
source page. The 3D extension is [[tapvid-3d-dataset]].

## Composition

- **Training:** synthetic videos generated with [[kubric-dataset]].
  Vanilla TAP-Vid ships 11K Kubric videos × 24 frames; many methods
  extend (TAPNext: 500K × 48; TAPNext++: Kubric-1024).
- **DAVIS eval:** 30 real videos, 24-105 frames each, human-labeled
  points. The most commonly-reported short-video benchmark.
- **Kinetics eval:** 1,150 real videos × 250 frames each, human-labeled.
- **RGB-Stacking eval:** robotic-manipulation-style synthetic eval
  (used by [[karaev-2024-cotracker]], CoTracker3).

## Metrics

Standard TAP metrics, all higher-is-better:

- **OA** — Occlusion Accuracy (binary visible/not).
- **δ_avg** — fraction of points within {1, 2, 4, 8, 16}-pixel
  thresholds, averaged.
- **AJ** — Average Jaccard, combines occlusion and coordinate accuracy.

## Evaluation protocols

- **First-frame** (`first`): query points come from frame 0.
- **Strided** (`strided`): query points sampled across the video —
  typically the harder setting.
- **Queried-first**: variant naming used in some papers (Track-On2,
  CoTracker3). Equivalent to `first`.

## Known biases / caveats

- Heavy reliance on synthetic training (Kubric) creates the
  [[synthetic-to-real-gap]] that motivated BootsTAP and
  [[karaev-2024-cotracker3]].
- DAVIS is small (30 videos) → headline numbers can be noisy.
- Kinetics labels can be inconsistent across annotators.
- No native 3D / dynamic-scene variant — see [[tapvid-3d-dataset]].

## Papers using it (in this wiki)

- [[zholus-2025-tapnext]] — TAPNext / BootsTAPNext.
- [[karaev-2024-cotracker]] — CoTracker.
- [[karaev-2024-cotracker3]] — CoTracker3 + scaling study.
- [[aydemir-2025-track-on2]] — Track-On2.
- [[jung-2026-tapnext-plus-plus]] — TAPNext++.

Effectively the universal benchmark for the methods in this wiki's batch.
