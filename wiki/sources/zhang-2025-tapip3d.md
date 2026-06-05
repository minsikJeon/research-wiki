---
type: source
source_type: paper
title: "TAPIP3D: Tracking Any Point in Persistent 3D Geometry"
authors:
  - Zhang, Bowei
  - Ke, Lei
  - Harley, Adam W.
  - Fragkiadaki, Katerina
year: 2025
venue: NeurIPS 2025
url: https://arxiv.org/abs/2504.14717
raw_path: papers/Zhang et al. - 2025 - TAPIP3D Tracking Any Point in Persistent 3D Geometry.pdf
status: ingested
tags: [point-tracking, 3d-point-tracking, monocular, depth-estimation, attention, world-coordinates]
sources: []
related:
  - "[[tapip3d]]"
  - "[[3d-point-tracking]]"
  - "[[point-tracking]]"
  - "[[tapvid-3d-dataset]]"
  - "[[adam-w-harley]]"
  - "[[katerina-fragkiadaki]]"
  - "[[cmu-ri]]"
created: 2026-05-24
updated: 2026-05-24
---

# TAPIP3D: Tracking Any Point in Persistent 3D Geometry

## TL;DR

TAPIP3D lifts [[point-tracking]] from the image plane into a
**camera-stabilized 3D world coordinate frame**: each frame's 2D features
are lifted to XYZ via depth + camera pose (from MegaSaM), camera motion
is cancelled, and tracking happens via a novel **3D
Neighborhood-to-Neighborhood (N2N) attention** over the resulting
spatio-temporal feature cloud. Beats prior 3D trackers (SpatialTracker,
DELTA) by large margins on TAPVid-3D and even beats 2D SOTA when
reliable depth is available. **CMU-led** (Fragkiadaki lab).

## Why it matters

- First paper to explicitly factor out camera motion *before* tracking,
  yielding much smoother and more linear 3D trajectories than the prior
  UVD-space (image + depth) approach.
- Establishes **world-centric 3D point tracking** as a distinct paradigm,
  enabled by recent advances in monocular depth + camera pose foundation
  models (MegaSaM, MoGe, DUSt3R lineage).
- Shows that **3D-aware attention** beats 2D-square-window correlation
  for tracking — the geometric prior matters, not just the depth channel.
- Direct CMU output ([[cmu-ri]] / Fragkiadaki lab) — likely a high-priority
  paper for the user.

## Key claims

- **World > camera coords for tracking.** Same architecture in world
  coords beats it in camera (UVD) coords. Camera motion is the dominant
  apparent-motion component and tracking what's left after subtracting
  it is much easier.
- **3D k-NN neighborhoods > 2D square windows.** The N2N attention
  attends from a query's 3D neighborhood to a key point's 3D neighborhood,
  with 3D relative offsets baked into attention values. This replaces
  PIPs/CoTracker-style 2D cost volumes.
- **3D > 2D when depth is reliable.** With GT or sensor depth, TAPIP3D
  beats CoTracker3 and BootsTAPIR (2D SOTA) on 2D tracking metrics —
  showing depth is informative for 2D tracking too.
- **Better depth → better 3D tracking.** Swapping in UnidepthV2 vs
  MegaSaM depth materially shifts results; depth quality is a
  bottleneck.
- New SOTA on TAPVid-3D (Aria, DriveTrack, PStudio subsets); also strong
  on LSFOdyssey, Dynamic Replica, DexYCB.

## Methods

- **Input:** RGB-D video + optional camera poses. If no depth: estimate
  with MoGe; if no poses: estimate with MegaSaM.
- **3D lifting:** 2D features + depth + intrinsics → XYZ spatio-temporal
  feature cloud. With camera extrinsics, transform into a fixed world
  frame.
- **Trajectory init:** queries given as 2D + depth (lift) or directly as
  XYZ. Initial trajectories assume zero motion.
- **N2N attention:** for each query trajectory point, find k-NN
  neighbors in 3D; for each candidate update step, find a target
  point's k-NN; cross-attention between the two groups, with 3D
  relative offsets concatenated to attention values.
- **Updates:** iterative refinement (6× transformer updates) of trajectory
  position + visibility + confidence per window.
- Sliding-window with 50% overlap.

## Results (headline)

- **TAPVid-3D** (with GT/sensor depth + GT pose): new SOTA across Aria,
  DriveTrack, PStudio. +61.8% AJ over DELTA, +50.5% APD3D.
- **TAPVid-3D + estimated depth (MegaSaM):** still SOTA, smaller margin —
  depth quality matters.
- **TAP-Vid DAVIS** 2D tracking: when given depth, TAPIP3D beats
  CoTracker3 and BootsTAPIR (2D-only methods).
- **Dynamic Replica, DexYCB, LSFOdyssey:** consistent gains over prior
  3D trackers.

## Limitations / open questions

- Inference-time depends on depth + pose estimators (MegaSaM is
  optimization-based → slow). [[spatialtracker-v2]] addresses this with
  a fully feed-forward depth+pose+tracking pipeline.
- 3D k-NN attention is more expensive than triplane / fixed windows.
  SpatialTracker uses triplane and is faster (but lower quality).
- Doesn't address tracking on transparent / textureless surfaces where
  depth estimation itself fails.

## Connections

- Method: [[tapip3d]].
- Concept introduced/championed: world-centric tracking under
  [[3d-point-tracking]].
- Contemporary 3D competitor: [[xiao-2025-spatialtracker-v2]] (different
  design philosophy — decomposed depth+pose+motion vs. lifted feature
  cloud).
- Author: [[adam-w-harley]] (also PIPs author — recurring entity).
  Senior: [[katerina-fragkiadaki]] (CMU faculty — user's institution).
- Org: [[cmu-ri]].
- Dataset introduced/used: [[tapvid-3d-dataset]]. Also Dynamic Replica,
  LSFOdyssey, DexYCB.
- MegaSaM, MoGe, DUSt3R, DepthAnything, UnidepthV2 — defer to entity
  pages until 2nd mention (most already seeded by [[spatialtracker-v2]]).

## Citation

Zhang, B., Ke, L., Harley, A. W., & Fragkiadaki, K. (2025). *TAPIP3D:
Tracking Any Point in Persistent 3D Geometry.* NeurIPS 2025.
arXiv:2504.14717v3. https://arxiv.org/abs/2504.14717
