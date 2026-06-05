---
type: method
title: D4RT (Dynamic 4D Reconstruction & Tracking)
status: growing
tags: [4d-reconstruction, feed-forward, transformer, query-based, point-tracking, scene-representation-transformer]
sources:
  - "[[zhang-2025-d4rt]]"
related:
  - "[[4d-reconstruction]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[point4d]]"
  - "[[trajectory-chaining]]"
created: 2026-05-26
updated: 2026-05-26
---

# D4RT (Dynamic 4D Reconstruction & Tracking)

Unified feed-forward 4D model with a **query-based decoder** that
predicts the 3D position of a queried 2D point at any target time,
expressed in any camera's coordinate frame. One model handles 4D
correspondence + point cloud + depth + camera extrinsics + intrinsics
through query variation. The canonical **2D-pixel-query 4D decoder**;
extended to 3D-coordinate queries by [[point4d]].

## One-line summary

Encoder (ViT with frame-wise + global attention) → fixed Global Scene
Representation `F` → independent cross-attention decoder takes query
`(u, v, t_src, t_tgt, t_cam)` + local 9×9 RGB patch → predicts 3D
position `P`. Same `F` reused across arbitrarily many queries; no
self-attention between queries.

## Inputs / outputs

- **In:** video `V ∈ R^{T×H×W×3}`; query `q = (u, v, t_src, t_tgt, t_cam)`
  + local 9×9 RGB patch around `(u, v)` at frame `t_src`.
- **Out:** 3D point position `P = D(q, F) ∈ R^3` of that point at time
  `t_tgt` in camera `t_cam`'s coordinate frame.
- **Plus** auxiliary outputs: 2D image-plane reprojection, surface
  normal, visibility, displacement, confidence.

## How it works

1. **Encode once:** ViT with interleaved frame-wise + global
   self-attention → `F`. Variable aspect ratio handled by an aspect-
   ratio token concatenated with video tokens.
2. **Construct query token:** Fourier-encoded `(u, v)` + discrete
   timestep embeddings for `t_src`, `t_tgt`, `t_cam` + local-patch
   features. Sum.
3. **Cross-attend to `F`.** Independent per-query — **no self-attention
   between queries**. (Self-attention between queries hurt performance
   in early experiments.)
4. **Output projection** → 3D position `P`. Auxiliary heads → 2D
   coords, normal, visibility, displacement, confidence.
5. **Six tasks via query variation** (Tab 1 of source):
   - **Point track:** fix `(u, v, t_src)`, sweep `t_tgt = t_cam`.
   - **Point cloud (frame `t_cam`):** sweep `(u, v, t_src)`, fix `t_cam`.
   - **Depth map (frame `t`):** set `t_src = t_tgt = t_cam = t`,
     sweep `(u, v)`, keep `Z`.
   - **Extrinsics i→j:** decode coarse `(h, w)` grid in two reference
     frames; solve `Sim(3)` via Umeyama.
   - **Intrinsics i:** decode coarse grid, recover focal length via
     pinhole `f = p_z(u-0.5)/p_x`.
6. **Dense tracking algorithm (Alg 1):** occupancy grid
   `G ∈ {0,1}^{T×H×W}`; initiate tracks only from unvisited pixels;
   each full-video track marks visible pixels as visited. 5-15×
   speedup over naive dense.

## Losses

- **L_point:** L1 on normalized 3D position, signed-log dampened (same
  as Point4D).
- **L_2d:** L1 on 2D image-plane projection.
- **L_normal:** cosine similarity on surface normals.
- **L_vis:** BCE on visibility.
- **L_displ:** L1 on motion displacement vector.
- **Confidence penalty `−log(c)`** weights `L_point`.

## Where it's been applied

- **TAPVid-3D 3D tracking** (camera + world coords): SOTA on
  DriveTrack, ADT, PStudio.
- **Sintel + ScanNet point cloud reconstruction.**
- **Sintel + ScanNet + KITTI + Bonn video depth.**
- **Sintel + ScanNet + Re10K camera pose.**

## Known limitations

- **No long-video eval in original paper.** Sequence lengths are
  short-clip / single-window. Long-horizon failure modes — observed
  by [[anon-2026-point4d]] — not addressed by D4RT.
- **Reliance on local 9×9 patch.** Performance drops without it.
- **Visibility-bound queries.** 2D pixel must be visible in source
  frame; occluded points unreliable. ([[point4d]] is the explicit fix.)
- **Per-query independence** trades cross-query consistency for
  scalability. May miss group-motion structure that joint methods
  (e.g., CoTracker) exploit.

## Related methods

- **Predecessor:** SRT (Scene Representation Transformer; Sajjadi et al.
  2022). D4RT = dynamic 4D extension of SRT.
- **Direct successor:** [[point4d]] — extends to 3D coordinate queries
  enabling long-video chunk chaining via [[trajectory-chaining]].
- **Sibling encode-once-query-anywhere design:** [[4rc]] — adds AdaLN
  target-time conditioning + outputs base-geometry + displacement
  rather than direct 3D position.
- **Dense per-frame alternatives:** [[any4d]] (factored), [[v-dpm]]
  (DPM heads on VGGT).
- **Trajectory-output alternative:** [[trace-anything]] (Bézier
  splines).
- **VGGT-family encoder pattern:** same alternating attention as
  [[vggt]], [[mapanything]], [[depth-anything-3]].
- **3D-tracking competitors beaten on TAPVid-3D:** [[tapip3d]],
  [[spatialtracker-v2]], DELTA, CoTracker3+VGGT.
