---
type: method
title: Trace Anything
status: stub
tags: [4d-reconstruction, trajectory-fields, video-representation, feed-forward, point-tracking, dense-tracking]
sources:
  - "[[anon-2026-trace-anything]]"
related:
  - "[[trajectory-fields]]"
  - "[[4d-reconstruction]]"
  - "[[point-tracking]]"
  - "[[3d-point-tracking]]"
created: 2026-05-24
updated: 2026-05-24
---

# Trace Anything

A feed-forward transformer that predicts a **Trajectory Field** for any
video — a dense mapping from each `(pixel, frame)` to a **parametric 3D
trajectory** (spline / Bézier). Single pass, no depth/flow/tracking
sub-models, no per-scene optimization.

## One-line summary

Feed-forward network ingests video frames and emits a stack of
**control point maps per frame**, defining spline-based parametric
trajectories per pixel — the full 4D representation in one pass.

## Inputs / outputs

- **In:** N RGB frames (monocular video, image pairs, or unordered photo
  sets capturing dynamic scenes).
- **Out:** **Trajectory Field** = per-frame stacks of control point maps
  defining per-pixel spline trajectories in 3D.

## How it works

1. **Network:** feed-forward transformer over input frames.
2. **Per-frame output:** stack of control point maps. Together with the
   spline interpolation rule, they specify a 3D trajectory for every
   pixel in every frame.
3. **No auxiliary estimators:** depth, flow, tracking are not run as
   sub-models; the trajectory field directly encodes everything.
4. **Trained on a new synthetic dataset:** Blender-based platform
   producing 10K+ videos × 120 frames, with dense 2D/3D trajectory
   ground truth.

## Where it's been applied

- New **Trace Anything benchmark** (200 curated videos for trajectory
  field eval).
- TAP-Vid (point tracking) — competitive at higher efficiency.
- Goal-conditioned manipulation, motion forecasting, spatio-temporal
  fusion (qualitative).

## Known limitations

- Spline parameterization may struggle with **high-frequency / abrupt
  motion** (this is the explicit [[4rc]] critique).
- "Any video*" — the asterisk applies; unstructured photo sets are a
  TBD claim.
- No metric-scale outputs (unlike [[any4d]], [[mapanything]]).
- Anonymous submission — author / institution / release timeline TBD.

## Related methods

- **Concept introduced:** [[trajectory-fields]].
- **Concurrent feed-forward 4D:** [[any4d]] (dense + metric, but only
  view-1 scene flow), [[v-dpm]] (DPMs, per-frame), [[4rc]] (conditional
  query, base+displacement).
- **Point-tracking lineage:** beats / matches CoTracker / TAPIR /
  TAPNext family at dense-trajectory granularity (sparse trackers'
  natural extension).
- **Geometry lineage cited:** DUSt3R, Fast3R, [[vggt]], π³,
  [[mapanything]], MegaSaM, MonST3R, POMATO, Easi3R, St4RTrack.
