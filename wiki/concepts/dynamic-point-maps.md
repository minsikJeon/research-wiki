---
type: concept
title: Dynamic Point Maps (DPMs)
status: stub
tags: [pointmap, dynamic-point-maps, 4d-reconstruction, scene-flow]
sources:
  - "[[sucar-2026-v-dpm]]"
related:
  - "[[pointmap-representation]]"
  - "[[4d-reconstruction]]"
  - "[[v-dpm]]"
created: 2026-05-24
updated: 2026-05-24
---

# Dynamic Point Maps (DPMs)

## Definition

A generalization of [[pointmap-representation]] to dynamic scenes,
introduced by Sucar et al. (2025) as the pair-wise DPM. A DPM
`P_i(t_j, π_k) ∈ R^{3×H×W}` associates each pixel of image `i` with its
3D location at **time** `t_j` viewed from **camera** `π_k`. The two
auxiliary indices decouple time and viewpoint from the source image.

## Why it matters

A single representation that captures **all** dynamic 3D quantities:

- Scene shape (`P_i(t_i, π_i)` = standard static pointmap at the source
  time and viewpoint).
- Scene flow (`P_i(t_1, π_0) - P_i(t_0, π_0)` = 3D motion of pixels in
  image `i`).
- Camera intrinsics + extrinsics (recoverable from the maps' geometric
  consistency).

For the pairwise (two-image) case, the four maps
`P_0(t_0, π_0), P_0(t_1, π_0), P_1(t_0, π_0), P_1(t_1, π_0)` encode the
**complete** 4D state.

## Variants

- **Pair-wise DPM** (Sucar et al., 2025, predecessor): 2 images → 4 DPMs.
  Multi-image needs post-hoc optimization to fuse pairs.
- **Multi-image / video DPM** ([[v-dpm]]): exploits redundancy to reduce
  to O(N) maps per target time. Uses VGGT backbone fine-tuned with DPM
  heads.

## Evidence across sources

Only directly in [[sucar-2026-v-dpm]] in this wiki so far. Concept page
seeded to anchor future DPM-related ingests.

## Open questions

- **DPMs vs. trajectory fields** ([[trajectory-fields]],
  [[trace-anything]]): per-time-step DPMs vs. continuous parametric
  trajectories — which is the right primitive? Trace Anything argues
  the latter; V-DPM argues the former.
- **Topology change** (objects appearing / disappearing) breaks the
  pixel-correspondence assumption underlying DPMs. Open how to handle.
- **Sparse DPMs** (only some pixels tracked) for efficiency — not
  explored.
