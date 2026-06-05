---
type: source
source_type: paper
title: "Efficiently Reconstructing Dynamic Scenes One D4RT at a Time"
authors:
  - Zhang, Chuhan
  - Le Moing, Guillaume
  - Koppula, Skanda
  - Rocco, Ignacio
  - Momeni, Liliane
  - Xie, Junyu
  - Sun, Shuyang
  - Sukthankar, Rahul
  - Barral, Joëlle K.
  - Hadsell, Raia
  - Ghahramani, Zoubin
  - Zisserman, Andrew
  - Zhang, Junlin
  - Sajjadi, Mehdi S. M.
year: 2025
venue: arXiv (cs.CV)
url: https://arxiv.org/abs/2512.08924
raw_path: papers/2512.08924v2.pdf
status: ingested
tags: [4d-reconstruction, feed-forward, transformer, query-based, point-tracking, scene-representation-transformer, foundation-model, dynamic-scenes]
sources: []
related:
  - "[[d4rt]]"
  - "[[4d-reconstruction]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[point4d]]"
  - "[[trajectory-chaining]]"
  - "[[mehdi-sajjadi]]"
  - "[[skanda-koppula]]"
  - "[[ignacio-rocco]]"
  - "[[carl-doersch]]"
  - "[[google-deepmind]]"
created: 2026-05-26
updated: 2026-05-26
---

# Efficiently Reconstructing Dynamic Scenes One D4RT at a Time

## TL;DR

D4RT is the **unified query-based 4D decoder** that nearly all later 4D
work (including [[point4d]], [[4rc]]) builds on or contrasts with. A
single feed-forward transformer: global self-attention encoder → fixed
**Global Scene Representation `F`** → independent **cross-attention
decoder** taking a query `(u, v, t_src, t_tgt, t_cam)` plus a local 9×9
RGB patch → predicts the 3D position of that point at the target
timestep in the camera's coordinate frame. **One model, six tasks**
(point tracks, point clouds, depth, camera extrinsics + intrinsics, 4D
correspondence) through query variation. SOTA on TAPVid-3D and Sintel/
ScanNet depth + pose, **9× faster than VGGT, 100× faster than MegaSaM**
at pose estimation. Google DeepMind + UCL + Oxford.

## Why it matters

D4RT is **the canonical 2D-pixel-query 4D decoder**. Almost every
feed-forward 4D paper in this wiki either (a) uses this design pattern
or (b) explicitly defines itself against it:

- [[anon-2026-point4d]] explicitly **extends D4RT to 3D-coordinate
  queries** to enable chunk chaining. The encoder + cross-attn-decoder
  architecture is the same.
- [[luo-2026-4rc]] uses the "encode-once, query-anywhere/anytime"
  pattern that originates here.
- [[karhade-2025-any4d]], [[sucar-2026-v-dpm]] are dense per-frame
  alternatives — comparison points to D4RT's sparse decoding.
- [[anon-2026-trace-anything]] is the trajectory-field alternative —
  trajectory-as-output vs D4RT's per-time-step query.

The design lineage runs DeepMind: SRT (Sajjadi et al. 2022,
**Scene Representation Transformer**) → D4RT (this paper). Sajjadi is the
lead on both. **D4RT is SRT extended to dynamic 4D.**

Author overlap with other wiki sources confirms a tight
**DeepMind 4D/TAP cluster**: Skanda Koppula introduced
[[tapvid-3d-dataset]] AND is on [[zholus-2025-tapnext]] AND D4RT.
Ignacio Rocco is on [[karaev-2024-cotracker]], TAPNext, and D4RT.
Carl Doersch is acknowledged. The DeepMind TAP/4D research line is
now one coherent program with TAPNext, BootsTAP, TAPVid-3D, and D4RT.

## Key claims

- **One query interface for six 4D tasks.** Query = `(u, v, t_src, t_tgt,
  t_cam)`. Vary which indices are fixed vs swept to recover point
  tracks, point clouds, depth maps, camera extrinsics, intrinsics.
- **Independent per-query cross-attention** beats letting queries
  self-attend. (Empirically: "major performance drops when enabling
  self-attention between queries.") This is the architectural
  decision Point4D and 4RC inherit.
- **Local 9×9 RGB patch** input to the query dramatically improves
  performance (Tab 7, Fig 6 — sharper depth, better fine-grained
  details). Critical for sub-pixel precision.
- **SRT-style encoder-decoder**: global self-attention encoder produces
  `F` once; lightweight cross-attention decoder queries `F` for any
  subset of points. Cost scales linearly with number of queries.
- **SOTA on TAPVid-3D 3D tracking** (Tab 4): AJ 0.304 / APD3D 0.410 on
  DriveTrack (camera coords, with GT intrinsics) vs SpatialTrackerV2
  0.195 / 0.275. Also wins in **world-coord 3D tracking** (right side
  of Tab 4) where the model must implicitly change reference frames.
- **SOTA on depth + pose** (Tab 5, 6): beats VGGT, MegaSaM, π³,
  MapAnything, SpatialTrackerV2 on Sintel video depth, Sintel/ScanNet
  pose. AbsRel(S) 0.171 on Sintel vs π³ 0.241 / VGGT 0.318.
- **200+ FPS pose estimation, 9× faster than VGGT, 100× faster than
  MegaSaM** (Fig 3). The query-based decoder is genuinely efficient.
- **Algorithm 1: efficient dense tracking** with an occupancy grid.
  Only initiate tracks from unvisited pixels; each full-video track
  marks all visible pixels as visited. **5-15× speedup** on naive
  dense tracking depending on motion.
- **Camera intrinsics + extrinsics from point queries.** Build a coarse
  `(h, w)` grid of source points; decode their 3D positions in two
  reference frames; solve `Sim(3)` via Umeyama for extrinsics; use
  pinhole formula `f_x = p_z(u-0.5)/p_x` for intrinsics. Camera params
  are emergent — no dedicated head.

## Methods

- **Architecture:** ViT encoder (frame-wise + global attention, VGGT
  pattern). Variable resolution handled by resizing to fixed square +
  embedding the aspect ratio as a token. Outputs `F ∈ R^{N×C}`.
- **Decoder:** small cross-attention transformer with **no
  self-attention between queries**.
- **Query token construction:** Fourier-encoded `(u, v)` + discrete
  timestep embeddings for `t_src`, `t_tgt`, `t_cam` + local 9×9 patch
  features.
- **Output projection:** simple learned MLP → 3D position `P ∈ R^3`.
- **Losses (multi-task):**
  - Primary: L1 on normalized 3D position, with `sign(x) · log(1+|x|)`
    dampening for far-away points (same loss Point4D uses).
  - Auxiliary: 2D image-plane reprojection (L1), surface normals
    (cosine), visibility (BCE), motion displacement (L1).
  - Confidence penalty `−log(c)` weights the 3D error.
- **Encoder size scaling** (Tab 9): ViT-B (90M) → ViT-g (1B). Clear
  improvement with size; ViT-L is the default.
- **Implemented in Kauldron** (DeepMind's training framework).

## Results (headline)

**TAPVid-3D 3D tracking (with GT intrinsics, Tab 4 right column):**

| Method | AJ↑ (DriveTrack) | AJ↑ (ADT) | AJ↑ (PStudio) |
|---|---|---|---|
| CoTracker3 + VGGT | 0.245 | 0.175 | 0.215 |
| DELTA | 0.170 | 0.199 | 0.175 |
| SpatialTrackerV2 | 0.195 | 0.303 | 0.175 |
| **D4RT** | **0.304** | **0.307** | **0.372** |

**Sintel video depth (AbsRel-S↓, Tab 5):** D4RT 0.171 vs π³ 0.241 vs
VGGT 0.318 vs MegaSaM 0.342.

**Sintel camera pose (ATE↓, Tab 6):** D4RT 0.065 vs MegaSaM 0.074 vs
π³ 0.086 vs SpatialTrackerV2 0.126 vs VGGT 0.168.

**Re10K pose AUC↑ (Tab 6):** D4RT 83.5 vs π³ 78.7 vs SpatialTrackerV2
75.7 vs VGGT 70.2.

**Speed:** 200+ FPS pose, 18-300× faster than prior methods (Tab 3).
Dense tracking with occupancy-grid algorithm: 5-15× additional speedup.

## Limitations / open questions

- **No long-video evaluation** in this paper. [[anon-2026-point4d]]
  empirically shows D4RT-style 2D-pixel-query methods break at chunk
  boundaries under occlusion. D4RT itself doesn't address sequences
  past the single-window range (concrete frame counts in the paper
  are mostly Sintel/ScanNet short clips).
- **Local 9×9 RGB patch dependency.** Performance drops noticeably
  without it (AbsRel-S 0.302→0.366). The 9×9 size is a fixed
  hyperparameter — robustness across scales is not extensively
  studied.
- **No explicit visibility handling.** Visibility is an auxiliary
  output, but tracks for occluded points aren't reliable — Point4D
  re-extracts patches from any visible frame to fix this.
- **Confidence calibration** matters a lot for pose estimation
  (Tab 8): removing the confidence loss tanks ATE 0.091→0.217.
  Calibration analysis not deeply explored.

## Connections

- Method: [[d4rt]] (promoted from secondary stub).
- **Predecessor:** SRT (Scene Representation Transformer; Sajjadi et al.
  2022) — D4RT is the dynamic 4D extension. Same lead author. Promote
  SRT to its own page on 2nd mention.
- **Direct successor:** [[anon-2026-point4d]] — extends D4RT's 2D-pixel
  queries to 3D-coordinate queries for chunk chaining. Same encoder
  pattern.
- **Sibling design (also encode-once + query):** [[luo-2026-4rc]] —
  uses target-time tokens + AdaLN conditioning instead of D4RT's
  discrete-timestep embeddings; outputs base geometry + displacement
  rather than direct 3D position.
- **Dense per-frame alternatives:** [[any4d]], [[v-dpm]] — DPT-style
  dense heads instead of sparse queries.
- **Trajectory-output alternative:** [[trace-anything]] — Bézier
  splines as the 4D primitive (and empirically the worst at long
  range per Point4D).
- **3D-tracking competitors:** [[tapip3d]], [[spatialtracker-v2]]
  (iterative refinement), CoTracker3+depth (2D tracker + depth).
  D4RT beats all on TAPVid-3D Tab 4.
- **Static 3D competitors:** [[vggt]] (D4RT is 9× faster at pose),
  [[mapanything]], [[depth-anything-3]], π³ (the strongest VGGT
  successor cited; not yet ingested).
- **Authors:** [[mehdi-sajjadi]] (lead; also on [[zholus-2025-tapnext]]
  + SRT line), [[skanda-koppula]] (also on [[tapvid-3d-dataset]] and
  TAPNext — **TAPVid-3D introducer**), [[ignacio-rocco]] (also on
  [[karaev-2024-cotracker]] + TAPNext), [[andrew-zisserman]] (Oxford
  VGG senior). [[carl-doersch]] in acknowledgments.
- **Org:** [[google-deepmind]] (most of the team) + [[oxford-vgg]]
  (Zisserman + Xie) + UCL (Koppula's secondary).
- **Datasets eval'd on:** [[tapvid-3d-dataset]], Sintel, ScanNet,
  KITTI, Bonn, Re10K. Defer Sintel/ScanNet/etc. dataset pages until
  multi-mention.

## Citation

Zhang, C., Le Moing, G., Koppula, S., Rocco, I., Momeni, L., Xie, J.,
Sun, S., Sukthankar, R., Barral, J. K., Hadsell, R., Ghahramani, Z.,
Zisserman, A., Zhang, J., & Sajjadi, M. S. M. (2025). *Efficiently
Reconstructing Dynamic Scenes One D4RT at a Time.* arXiv:2512.08924.
https://arxiv.org/abs/2512.08924
