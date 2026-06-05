---
type: method
title: TAPIP3D
status: stub
tags: [point-tracking, 3d-point-tracking, monocular, depth-estimation, attention, world-coordinates]
sources:
  - "[[zhang-2025-tapip3d]]"
related:
  - "[[3d-point-tracking]]"
  - "[[point-tracking]]"
  - "[[spatialtracker-v2]]"
created: 2026-05-24
updated: 2026-05-24
---

# TAPIP3D

3D point tracking method that lifts a video into a **camera-stabilized
spatio-temporal 3D feature cloud** (using depth + camera pose from
MegaSaM), then tracks via a novel **3D Neighborhood-to-Neighborhood
(N2N) attention** in the world coordinate frame.

## One-line summary

Lift 2D features to XYZ via depth + pose → cancel camera motion to get
world coordinates → track via k-NN-based 3D-to-3D cross-attention with
relative-offset positional encoding → iteratively refine over windows.

## Inputs / outputs

- **In:** RGB video (optionally RGB-D + camera poses); query points in
  2D or 3D.
- **Out:** per-query 3D trajectory `(X, Y, Z)_t` (in world or camera
  coords) + visibility + confidence.

## How it works

1. **Depth + pose:** use MoGe (depth) and MegaSaM (camera pose) if not
   provided.
2. **Lifting:** project per-frame 2D feature maps + depth to XYZ; apply
   camera extrinsics to transform into a fixed **world** coordinate
   frame. Result: spatio-temporal 3D feature cloud.
3. **Query init:** 3D position copied across all frames (zero-motion
   assumption); confidence + visibility initialized to zero.
4. **3D N2N attention:** for each query trajectory point, find its
   k-nearest 3D neighbors (query group); for each candidate target
   point, find its k-NN (key group). Cross-attention between groups with
   **3D relative offsets concatenated to attention values** for spatial
   awareness.
5. **Iterative refinement:** 6 transformer-update iterations per window;
   outputs `Δτ`, `Δo`, `Δc` per iteration.
6. **Windowed inference:** sliding windows with 50% overlap; second half
   of previous window seeds first half of next.

## Where it's been applied

- TAPVid-3D (Aria, DriveTrack, PStudio).
- LSFOdyssey, Dynamic Replica, DexYCB.
- Also evaluated on TAP-Vid DAVIS in 2D — beats CoTracker3 and BootsTAPIR
  when given depth.

## Known limitations

- Depends on external depth + pose estimators; quality of those is a
  bottleneck (especially MegaSaM, which is optimization-based and slow).
- 3D k-NN attention is expensive compared to triplane representations
  (SpatialTracker, faster) — accuracy gain comes at compute cost.
- Doesn't handle scenes where depth estimation itself fails (transparent,
  textureless surfaces).

## Related methods

- **Closest 3D competitor:** [[spatialtracker-v2]] (different philosophy —
  fully end-to-end, jointly trains depth + pose + tracking).
- **Predecessors / cited 3D methods:** SpatialTracker (V1), DELTA,
  SceneTracker, OmniMotion. Defer to entity pages on 2nd mention.
- **2D backbone lineage:** PIPs / CoTracker iterative-refinement
  inheritance, but the cost volume is replaced with 3D N2N attention.
