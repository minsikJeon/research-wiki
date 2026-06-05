---
type: method
title: SpatialTrackerV2
status: stub
tags: [point-tracking, 3d-point-tracking, monocular, depth-estimation, camera-pose, bundle-adjustment, feed-forward, end-to-end]
sources:
  - "[[xiao-2025-spatialtracker-v2]]"
related:
  - "[[3d-point-tracking]]"
  - "[[point-tracking]]"
  - "[[tapip3d]]"
created: 2026-05-24
updated: 2026-05-24
---

# SpatialTrackerV2

Feed-forward end-to-end 3D point tracker that jointly recovers **video
depth + camera pose + 3D trajectories** in a single differentiable
pipeline, with bundle adjustment in the loop. Decomposes world-space 3D
motion into ego-motion + object-motion components.

## One-line summary

Front-end (temporal depth + camera pose) → back-end (SyncFormer dual-branch
2D/3D transformer + differentiable bundle adjustment) → outputs 3D tracks,
camera poses, video depth, and dynamic/static probability per point —
trained jointly on a 17-dataset mix.

## Inputs / outputs

- **In:** monocular RGB video + N 2D query points.
- **Out:** per-query 3D trajectories `(X, Y, Z)_t`, dynamic probability
  `p_dyn`, visibility `p_vis`, *plus* video depth and camera pose
  sequence as byproducts.

## How it works

1. **Front-end (depth + pose initializer):**
   - Temporal depth encoder built on DepthAnythingV2 with
     alternating spatial / temporal attention.
   - Two learnable tokens `Ptok`, `Stok` aggregate semantics for pose
     and scale regression.
   - Differentiable pose head outputs `(P_t, a, b)` — camera params +
     scale/shift to align depth.
   - Outputs initial scale-aligned per-frame depth + camera trajectory.
2. **Back-end (Joint Motion Optimization):**
   - **SyncFormer:** dual-branch transformer. 2D branch operates in
     image (UV) space; 3D branch operates in camera coordinate space.
     Cross-attention between branches at multiple layers prevents the
     two embedding spaces from interfering.
   - **Proxy tokens:** correlations are first encoded into compact 2D
     and 3D proxy tokens via cross-attention, then refined.
   - **Differentiable bundle adjustment:** uses the predicted dynamic
     probability to mark static points; bundle-adjusts camera poses
     using static-point consistency while leaving dynamic points free.
3. **Loss:** consistency between 2D and 3D branches; ground-truth depth
   + pose supervision where available; self-supervised camera-pose +
   tracking consistency where only pose annotations exist; standard
   tracking losses on labeled data.
4. **Training mix:** 17 datasets — Kubric, PointOdyssey, VKITTI,
   RGB-D-with-poses, video-with-poses-only, etc.

## Where it's been applied

- TAPVid-3D (Aria / DriveTrack / PStudio).
- Dynamic reconstruction: TUM-dynamic, Lightspeed, Sintel (depth + pose).
- Demos on robotic manipulation, first-person, dynamic sports.

## Known limitations

- Reproduction burden is high — 17-dataset training mix, custom dual-branch
  arch, differentiable BA.
- Static-vs-dynamic decision via learned probability — may fail in subtle
  cases that optimization-based methods would resolve.
- SyncFormer convergence / scaling not deeply ablated.
- No head-to-head comparison with [[tapip3d]] on a unified eval — both
  claim SOTA but with different protocols.

## Related methods

- **Closest 3D competitor:** [[tapip3d]] (modular: pre-computed depth +
  pose, then track in lifted cloud). SpatialTrackerV2 is the joint-training
  counterpoint.
- **Predecessor:** SpatialTracker V1 (triplane-based, slower-quality
  feed-forward 3D tracker).
- **Foundation-model dependencies:** DepthAnythingV2, DINO. Adopts VGGT's
  alternating attention pattern.
- **Competing dynamic-reconstruction baseline:** MegaSaM (per-video
  optimization, 50× slower at similar quality).
