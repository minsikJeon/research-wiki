---
type: concept
title: 3D Point Tracking
status: growing
tags: [point-tracking, 3d-point-tracking, depth-estimation, camera-pose]
sources:
  - "[[zhang-2025-tapip3d]]"
  - "[[xiao-2025-spatialtracker-v2]]"
related:
  - "[[point-tracking]]"
  - "[[tapip3d]]"
  - "[[spatialtracker-v2]]"
  - "[[tapvid-3d-dataset]]"
created: 2026-05-24
updated: 2026-05-24
---

# 3D Point Tracking

## Definition

Recover 3D trajectories `(X, Y, Z)_t` for query points across a monocular
or RGB-D video, rather than 2D `(x, y)_t`. The natural extension of
[[point-tracking]] once depth + camera pose become accessible from
foundation models.

## Why it matters

Most real-world motion happens in 3D. Tracking in 2D conflates object
motion with camera motion — a stationary object on a moving camera
generates 2D motion that isn't physically meaningful. 3D tracking
separates these. Critical for robotics, AR/VR, dynamic 4D reconstruction.

## Two competing design philosophies (within this wiki)

- **Modular / lifted-cloud:** [[tapip3d]]. Use external depth + pose
  (MegaSaM, MoGe) to lift 2D features into a stabilized world-coord 3D
  feature cloud, then run a 3D-aware tracker. Tracks live in world coords
  with camera motion cancelled.
- **End-to-end / jointly trained:** [[spatialtracker-v2]]. Train depth,
  pose, and tracking jointly with differentiable bundle adjustment. Track
  decomposes into ego + object motion. Better data scalability (can train
  on real video with only pose annotations).

Both claim SOTA on TAPVid-3D — see [[cmp-tap-methods]] for the table.

## Coordinate frame choice

- **Camera coords (UVD = pixel + depth):** earlier 3D trackers
  (SpatialTracker V1, DELTA, SceneTracker).
- **World coords (XYZ after cancelling camera motion):** [[tapip3d]]
  shows this is materially better — trajectories become smoother and more
  linear because camera-induced apparent motion is removed.
- [[spatialtracker-v2]] models both branches simultaneously.

## Evidence across sources

- [[zhang-2025-tapip3d]]: world > camera coords for the same architecture.
  3D k-NN attention > 2D square cost volumes. With reliable depth,
  3D tracker beats 2D SOTA on 2D metrics.
- [[xiao-2025-spatialtracker-v2]]: joint depth + pose + tracking
  training beats modular pipelines on both speed (50× faster than
  MegaSaM) and 3D accuracy (+61.8% AJ over DELTA on TAPVid-3D).

## Open questions

- Head-to-head between [[tapip3d]] and [[spatialtracker-v2]] on a
  unified eval protocol — both claim SOTA but report under different
  conditions.
- How does 3D tracking degrade as depth quality degrades? Both papers
  do partial ablations; a systematic study across depth estimators is
  still missing.
- 3D + causal per-frame: neither paper here is strictly online. Can the
  [[tapnext]] / [[track-on2]] online-causal design extend to 3D?
- Tracking on transparent / textureless surfaces where depth estimation
  itself fails — unsolved.
