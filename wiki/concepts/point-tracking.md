---
type: concept
title: Point Tracking (Tracking Any Point, TAP)
status: growing
tags: [point-tracking, video-understanding, optical-flow]
sources:
  - "[[zholus-2025-tapnext]]"
  - "[[karaev-2024-cotracker]]"
  - "[[karaev-2024-cotracker3]]"
  - "[[aydemir-2025-track-on2]]"
  - "[[jung-2026-tapnext-plus-plus]]"
  - "[[zhang-2025-tapip3d]]"
  - "[[xiao-2025-spatialtracker-v2]]"
related:
  - "[[tapnext]]"
  - "[[tapnext-plus-plus]]"
  - "[[cotracker]]"
  - "[[cotracker3]]"
  - "[[track-on2]]"
  - "[[tapip3d]]"
  - "[[spatialtracker-v2]]"
  - "[[tap-vid-dataset]]"
  - "[[joint-point-tracking]]"
  - "[[3d-point-tracking]]"
  - "[[online-vs-offline-tracking]]"
  - "[[synthetic-to-real-gap]]"
created: 2026-05-24
updated: 2026-05-24
---

# Point Tracking (Tracking Any Point, TAP)

## Definition

Given a video and a set of **query points** `(t, x, y)` on solid surfaces,
predict each point's `(x, y)` (or `(X, Y, Z)` for 3D, see
[[3d-point-tracking]]) and visibility on **every other frame**, including
across occlusions. Outputs are dense like optical flow but long-range like
keypoint correspondence. Unified problem formulation introduced by the
TAP-Vid line (Doersch et al., NeurIPS 2022).

## Why it matters

Point tracks are a building block for many downstream tasks: robotic
manipulation (object-centric control), action recognition, 3D
reconstruction (dynamic scenes), controllable video generation, video
editing, biological/medical motion analysis. A general-purpose TAP model
is to video what a feature matcher is to image pairs.

## The main design axes (all with active wiki coverage)

- **Latency:** [[online-vs-offline-tracking]] — frame / window / video.
- **Cross-track information:** [[joint-point-tracking]] vs. independent
  tracks.
- **Dimensionality:** 2D vs. [[3d-point-tracking]].
- **Training data:** synthetic-only vs. real-fine-tuned — see
  [[synthetic-to-real-gap]] and [[pseudo-labeling-point-tracking]].
- **Backbone:** convnet (PIPs lineage) vs. transformer (CoTracker family)
  vs. SSM + ViT (TAPNext family) vs. memory + ViT (Track-On family).

## Standard architectural ingredients (pre-TAPNext)

Most prior TAP methods stacked:
1. Per-frame feature extraction.
2. **Cost volume** between query-point features and per-frame feature maps.
3. Iterative refinement of track estimates (often bidirectional in time).
4. Local search windows, feature interpolation, temporal smoothness priors,
   windowed inference.

[[zholus-2025-tapnext]] showed none of these are strictly necessary —
many re-emerge as learned attention patterns. [[jung-2026-tapnext-plus-plus]]
+ [[aydemir-2025-track-on2]] reinforced that long-sequence synthetic
training (no real data) is enough for SOTA. [[karaev-2024-cotracker3]]
showed an alternative path: simplify + add real-data pseudo-labels.

## Evidence across sources

- **Joint > independent:** [[karaev-2024-cotracker]],
  [[karaev-2024-cotracker3]] (cross-track attention; +5.1 occluded AJ).
- **Causal SSM beats windowed:** [[zholus-2025-tapnext]],
  [[jung-2026-tapnext-plus-plus]] (frame-by-frame at 348 FPS, SOTA on
  online benchmarks).
- **Causal memory beats windowed:** [[aydemir-2025-track-on2]]
  (memory + classification-first, synthetic-only).
- **3D > 2D when depth is reliable:** [[zhang-2025-tapip3d]],
  [[xiao-2025-spatialtracker-v2]].
- **Training video length is the dominant factor** for long-horizon
  generalization: convergent finding in [[aydemir-2025-track-on2]] and
  [[jung-2026-tapnext-plus-plus]].

## Open questions

- See [[cmp-tap-methods]] for the head-to-head method comparison.
- See [[q-emergent-tracking-heuristics]] for the "do classical heuristics
  emerge from end-to-end training?" thread.
- See [[q-sim2real-for-point-tracking]] for the synthetic-vs-real-data
  debate.
- 3D + causal per-frame is an open slot (no paper combines all three).
- Tracking on textureless / transparent / specular surfaces remains hard
  for all methods.
