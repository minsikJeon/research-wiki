---
type: question
title: Are point tracking and 4D reconstruction the same problem?
status: open
tags: [point-tracking, 4d-reconstruction, 3d-point-tracking, convergence]
sources:
  - "[[anon-2026-trace-anything]]"
  - "[[karhade-2025-any4d]]"
  - "[[luo-2026-4rc]]"
  - "[[zhang-2025-tapip3d]]"
  - "[[xiao-2025-spatialtracker-v2]]"
related:
  - "[[point-tracking]]"
  - "[[3d-point-tracking]]"
  - "[[4d-reconstruction]]"
  - "[[trajectory-fields]]"
created: 2026-05-24
updated: 2026-05-24
---

# Are point tracking and 4D reconstruction the same problem?

## The question

As of 2026, the two sub-fields are visibly converging:

- **From the point-tracking side:** [[zhang-2025-tapip3d]] and
  [[xiao-2025-spatialtracker-v2]] pushed TAP into 3D. Recent dense-
  tracking work is starting to look indistinguishable from per-pixel
  scene-flow prediction.
- **From the 4D reconstruction side:** [[karhade-2025-any4d]] outputs
  dense 3D scene flow (= dense 3D point tracks).
  [[anon-2026-trace-anything]] proposes the **Trajectory Field** as the
  4D representation — literally per-pixel parametric 3D trajectories,
  i.e. **dense 3D point tracks**. [[luo-2026-4rc]] outputs queryable
  per-pixel trajectories via base-geometry + displacement.

**Are these the same problem under different names?** If so, what
unifying framework subsumes them?

## Why it matters

If the answer is yes:

- The right SOTA model for one task should be the right SOTA model for
  the other. Currently they have separate leaderboards (TAP-Vid +
  TAPVid-3D + various 4D benchmarks).
- Architectural choices should converge — but they currently don't
  (TAPNext = SSM + ViT; CoTracker = transformer + cross-track attention;
  TraceAnything = transformer + spline output; Any4D = transformer +
  scene-flow head).
- Future evaluation should fuse the benchmarks.

If the answer is no:

- The differences need to be made precise. E.g., sparse vs dense, online
  vs batch, presence of per-frame geometry output, metric vs up-to-scale,
  static-camera vs ego-motion handling.

## What we know

**Arguments for "same problem":**

- All output some form of `f(pixel, time) → 3D position`.
- Inputs are the same — RGB video (sometimes + auxiliary modalities).
- Common foundation models — both use VGGT-family backbones, DINOv2/v3
  features, DepthAnything-style depth priors.
- Cross-paper citations are increasingly thick. [[trace-anything]]
  explicitly compares against both TAP-family and 4D-family. [[4rc]]
  evaluates on "3D point tracking" as a sub-task. [[any4d]] outputs
  scene flow that is operationally identical to 3D tracks.

**Arguments for "different problems":**

- **Output structure:** TAP outputs sparse trajectories for queried
  points; 4D reconstruction outputs dense per-pixel maps. Sparse vs
  dense is a real engineering distinction even if the underlying
  function is the same.
- **Visibility / occlusion handling:** TAP methods explicitly model
  per-frame visibility (sigmoid head). 4D methods often don't —
  occluded pixels just lose meaningful values.
- **Online streaming:** TAP has strong causal-online methods
  ([[tapnext]], [[track-on2]]); 4D doesn't (only [[cut3r]] is online,
  and it's 3D-only).
- **Benchmark distribution:** TAP-Vid sequences are short (~50-200
  frames); 4D benchmarks lean longer / more dynamic.

## What we don't

- **No paper does both as primary contribution.** Methods either own
  sparse-tracking (TAPNext++, Track-On2) or dense 4D (V-DPM, Any4D,
  4RC, Trace Anything) — not both with optimal performance.
- **No "tracker / reconstructor" superset model.** [[trace-anything]]
  comes closest: trajectory field can be queried sparsely or densely.
  But its TAP-Vid performance is "competitive," not SOTA.
- **No theoretical framework** that subsumes both as instances of a
  single problem with a parameterized output mode.

## Tentative position

The two are **converging but not yet unified**. The most likely
unifying frame:

```
Output: parametric trajectory per (pixel, frame) → 3D position at any time t
Input: arbitrary video + optional sparse query points
```

A model that outputs trajectory fields and exposes a sparse-query API
would subsume both. [[trace-anything]] is the closest, but it doesn't
quite win either benchmark.

For the user's robotics context: **bet on the converged design**.
Engineering for "sparse trajectories" alone is likely a transient
optimization; the dense-trajectory output is a strict superset, with
better downstream composition properties (forecasting, fusion,
goal-conditioned manipulation per [[trace-anything]]'s demos).

## Next things to read / ingest

- **DUSt3R / MASt3R primary papers** — the root of the geometry side.
- **π³ (Pi3)** — VGGT successor mentioned in multiple papers; may
  collapse more of the gap.
- **D4RT** (cited by [[4rc]]) — "Perceiver-like unified 2D/3D point
  tracking" — sounds like a direct attempt at the unification.
- **Spann3R, MonST3R, POMATO** — the pre-VGGT 4D lineage. Useful
  context for *why* the convergence is happening now.
- **Any robotics-application paper** that consumes 3D point tracks /
  4D reconstruction — would clarify which output structure is
  actually needed downstream.
