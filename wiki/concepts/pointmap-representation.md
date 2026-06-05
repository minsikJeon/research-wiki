---
type: concept
title: Pointmap Representation
status: growing
tags: [3d-reconstruction, pointmap, representation, dust3r]
sources:
  - "[[wang-2025-vggt]]"
  - "[[keetha-2025-mapanything]]"
  - "[[sucar-2026-v-dpm]]"
  - "[[wang-2025-cut3r]]"
  - "[[lin-2025-depth-anything-3]]"
related:
  - "[[feed-forward-3d-reconstruction]]"
  - "[[dynamic-point-maps]]"
  - "[[dust3r]]"
created: 2026-05-24
updated: 2026-05-24
---

# Pointmap Representation

## Definition

A **pointmap** `P_i ∈ R^{3×H×W}` associates each pixel of image `i` with
its corresponding 3D scene point. Crucially, points are expressed in a
**shared coordinate frame** (typically the first camera's frame, taken
as world origin). Pointmaps inherently encode camera intrinsics +
extrinsics + scene shape in a single tensor.

Introduced by DUSt3R (Wang et al., 2024) — see [[dust3r]] — as a
unified output representation for feed-forward 3D reconstruction.

## Why it matters

Pointmaps are **the** unified representation for the
[[feed-forward-3d-reconstruction]] sub-field. They're well-suited to
neural prediction (dense regression target), inherently encode geometry +
camera, and are easy to project/render/fuse.

## Key properties

- **Viewpoint-invariant** (in DUSt3R's original form): all pointmaps
  share a world frame so corresponding pixels in different images map to
  the *same* 3D point.
- **Self vs. world variants** ([[cut3r]]): some methods predict both
  the per-camera local pointmap and the world-frame pointmap, with the
  difference encoding ego-motion.
- **Derivable from depth + camera:** if you know `(D_i, g_i)` you can
  reconstruct `P_i`. Often **both** are predicted (over-complete) for
  training signal.
- **Per-pixel confidence** is typically predicted alongside (DUSt3R,
  MASt3R, CUT3R, MapAnything).

## Variants observed in this wiki

- **Static pointmap (DUSt3R-style):** [[vggt]], [[mapanything]],
  [[cut3r]], [[depth-anything-3]] (implicit via depth + rays),
  [[dust3r]].
- **Dynamic pointmap ([[dynamic-point-maps]] / DPMs, Sucar 2025):**
  pointmaps parameterized by `(viewpoint, time)`. Two-time-step
  pairs suffice to recover scene flow. Multi-view extension in
  [[v-dpm]] reduces O(N²) to O(N) maps per target time.
- **Factored (depth + ray + pose + scale):** [[mapanything]],
  [[any4d]] argue this beats coupled pointmaps for partial-annotation
  training.
- **Per-pixel parametric trajectory:** [[trajectory-fields]] —
  [[trace-anything]] replaces per-frame pointmaps with continuous splines
  in 3D.

## Tension with the minimalist view

[[depth-anything-3]] explicitly argues pointmaps are *over-complete*:
just predicting depth + ray maps is sufficient, and beats VGGT-style
multi-task heads on geometry + pose. This is an open design-space
debate — when does over-prediction help, and when does it hurt?

## Evidence across sources

- [[vggt]] uses pointmap head + over-complete depth+camera; finds joint
  prediction helps training; at inference the depth-camera derivation is
  actually more accurate.
- [[mapanything]]: factored representation enables training on
  partial-annotation datasets — major data-efficiency win.
- [[depth-anything-3]]: minimalist counter; beats VGGT.
- [[v-dpm]]: pointmaps generalize cleanly to DPMs for dynamic scenes.
- [[trace-anything]]: argues pointmap-per-frame is the wrong primitive;
  trajectory fields are more natural for video.

## Open questions

- **Pointmap vs depth+ray vs trajectory field:** which is the right
  primitive? Convergence may be on factored representations + task-
  specific decoders, rather than a single answer.
- **Confidence calibration:** all these models predict per-pixel
  confidence, but cross-method calibration / utility for downstream
  tasks is under-studied.
- **3D coordinate frame choice:** world-frame (first camera as origin)
  is a convention. Alternatives (canonical scene-centric frame,
  pre-aligned via SLAM) are under-explored.
