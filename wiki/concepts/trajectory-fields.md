---
type: concept
title: Trajectory Fields
status: stub
tags: [trajectory-fields, 4d-reconstruction, video-representation, dense-tracking]
sources:
  - "[[anon-2026-trace-anything]]"
related:
  - "[[trace-anything]]"
  - "[[4d-reconstruction]]"
  - "[[point-tracking]]"
  - "[[3d-point-tracking]]"
  - "[[dynamic-point-maps]]"
created: 2026-05-24
updated: 2026-05-24
---

# Trajectory Fields

## Definition

A **dense mapping** that assigns each `(pixel, frame)` in a video to a
**parametric 3D trajectory** — typically a spline or Bézier curve in 3D
space-time. Introduced by [[trace-anything]] (ICLR 2026 anonymous) as a
new 4D video representation.

## Why it matters

- **Atomic-level dynamics.** Pixels naturally trace 3D paths through
  time; the trajectory is more primitive than per-frame snapshots.
- **Continuous in time.** Unlike per-frame [[dynamic-point-maps]] or
  per-timestep displacement (4RC), trajectory fields are defined at
  arbitrary `t ∈ ℝ`, not just sampled timestamps.
- **Composable downstream.** Enables motion forecasting (extrapolate the
  spline), spatio-temporal fusion (sample at arbitrary time),
  goal-conditioned manipulation (plan in trajectory space).
- **Single feed-forward pass.** No depth / flow / tracking sub-models;
  no global alignment.

## How it's parameterized (in Trace Anything)

- **Control point maps per frame.** Each input frame produces a stack of
  control point maps (think: K maps for a K-th-order spline).
- **Per-pixel spline.** Each pixel's K control points define a 3D
  spline trajectory.
- **Shared world coord** — trajectories are predicted in a single 3D
  coordinate system across the whole video.

## Tension with competing representations

- **vs. per-frame [[pointmap-representation]] / [[dynamic-point-maps]]**
  ([[v-dpm]], [[any4d]]): the alternative is to predict a pointmap per
  frame + scene flow, then derive trajectories post-hoc. Trace Anything
  argues this is the wrong primitive — trajectories should be the unit
  of prediction.
- **vs. base-geometry + displacement** ([[4rc]]): 4RC's representation
  is "factored 4D" with `P = base + Δ`. Mathematically related but 4RC
  is per-timestep (not continuous), and 4RC critiques trajectory fields
  for "struggling with complex or high-frequency dynamics" given the
  spline parameterization.

## Evidence across sources

Only in [[trace-anything]] so far. The representation hasn't been
adopted yet by other groups in this wiki's batch — it's an unsettled
hypothesis. The [[4rc]] critique is the strongest direct counterargument.

## Open questions

- **High-frequency / abrupt motion:** can spline parameterization handle
  motion that the spline order can't represent? Higher-order splines
  vs. mixture-of-splines vs. discrete-segment trajectories — all open.
- **Topology change:** spline trajectories for pixels that appear /
  disappear mid-video — undefined.
- **Sparse vs. dense:** could a sparse trajectory field (only
  "interesting" pixels) be more compute-efficient at similar quality?
- **Cross-method evaluation:** does the trajectory-field
  parameterization outperform per-timestep DPMs on
  same-protocol comparisons? Not yet measured.
