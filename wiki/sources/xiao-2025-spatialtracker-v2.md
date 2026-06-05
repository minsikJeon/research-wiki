---
type: source
source_type: paper
title: "SpatialTrackerV2: 3D Point Tracking Made Easy"
authors:
  - Xiao, Yuxi
  - Wang, Jianyuan
  - Xue, Nan
  - Karaev, Nikita
  - Makarov, Yuri
  - Kang, Bingyi
  - Zhu, Xing
  - Bao, Hujun
  - Shen, Yujun
  - Zhou, Xiaowei
year: 2025
venue: arXiv (cs.CV)
url: https://arxiv.org/abs/2507.12462
raw_path: papers/Xiao et al. - 2025 - SpatialTrackerV2 3D Point Tracking Made Easy.pdf
status: ingested
tags: [point-tracking, 3d-point-tracking, monocular, depth-estimation, camera-pose, bundle-adjustment, feed-forward]
sources: []
related:
  - "[[spatialtracker-v2]]"
  - "[[3d-point-tracking]]"
  - "[[point-tracking]]"
  - "[[tapvid-3d-dataset]]"
  - "[[nikita-karaev]]"
  - "[[oxford-vgg]]"
created: 2026-05-24
updated: 2026-05-24
---

# SpatialTrackerV2: 3D Point Tracking Made Easy

## TL;DR

SpatialTrackerV2 (ST-V2) is a **fully feed-forward end-to-end** 3D point
tracker that jointly recovers **video depth, camera pose, and 3D
trajectories** from a monocular video. Unlike modular pipelines that
stack off-the-shelf depth + pose + 2D tracking, ST-V2 trains them
together with a differentiable bundle adjustment in the loop. Beats
DELTA (+61.8% AJ on TAPVid-3D), matches MegaSaM on dynamic reconstruction
**at 50× the speed**.

## Why it matters

The architectural counterpoint to [[tapip3d]]: same goal (world-space 3D
point tracking from monocular RGB), opposite philosophy.

- **[[tapip3d]]** lifts pre-computed depth + pose into a feature cloud
  and tracks there. Modular, leverages MegaSaM / MoGe.
- **ST-V2** trains depth + pose + tracking jointly end-to-end. Faster at
  inference, more scalable to weakly-labeled real data.

ST-V2's claim is that **joint training across heterogeneous data** beats
the modular approach for both speed and final 3D tracking quality.
Trained on 17 datasets including unlabeled in-the-wild video — a major
data-scale precedent for the sub-field.

## Key claims

- **Decomposition principle:** explicitly factor 3D motion into video
  depth + camera ego-motion + object motion; train each component's
  head jointly with consistency losses.
- **SyncFormer:** dual-branch transformer that processes 2D (UV) and 3D
  trajectories in separate branches connected by cross-attention.
  Reduces mutual interference between the two embedding spaces.
- **Differentiable bundle adjustment** in the loop for camera pose
  refinement using estimated 2D + 3D tracks + dynamic probability scores.
- **17-dataset training mix** spanning synthetic (Kubric, PointOdyssey,
  VKITTI), real RGB-D with poses, and video with pose-only annotations
  (using consistency losses where depth is missing).
- **+61.8% AJ, +50.5% APD3D over DELTA** on TAPVid-3D. 21.2 AJ, 31.0
  APD3D.
- **Matches MegaSaM** on consistent video depth / camera pose at 50×
  faster inference (feed-forward vs second-order optimization).

## Methods

- **Front-end:** temporal depth encoder (DepthAnythingV2 + alternating
  spatial/temporal attention). Pose head decodes camera pose, scale, and
  shift from learnable Ptok / Stok tokens. Outputs initial scale-aligned
  per-frame depth + camera trajectory.
- **Back-end (Joint Motion Optimization):** SyncFormer with 2D and 3D
  branches. 2D branch operates in UV; 3D branch operates in camera
  coordinates. Cross-attention between branches at multiple layers.
  Outputs refined trajectories `T_2d`, `T_3d`, dynamic probability
  `p_dyn`, visibility `p_vis`.
- **Bundle adjustment:** iterative; uses static-point consistency from
  the dynamic-probability head to refine camera poses while keeping
  dynamic points free.
- Training: separate losses for static/dynamic, consistency across
  branches, supervised on labeled subsets + self-supervised consistency
  on unlabeled.

## Results (headline)

- **TAPVid-3D:** new SOTA. AJ 21.2 / APD3D 31.0; +61.8% / +50.5% over
  DELTA. Outperforms TAPIP3D's reported numbers (different evaluation
  conditions — apples-to-apples comparison would need a head-to-head).
- **Video depth (TUM-dynamic, Lightspeed, Sintel):** beats MegaSaM on
  most metrics. Competitive on camera pose.
- **Speed:** 50× faster than MegaSaM (which is per-video optimization).
- Demo on robotic manipulation, first-person, dynamic sports (Fig 1) —
  intentional generality.

## Limitations / open questions

- Multi-dataset training mix is complex; reproducibility burden is high.
- Static-vs-dynamic decision via learned probability — failure modes on
  near-rigid scenes with subtle motion may differ from optimization-based
  methods.
- SyncFormer architecture is novel — convergence / scaling properties
  vs simpler joint encoders not deeply ablated in this paper.
- Open: how does ST-V2 vs [[tapip3d]] head-to-head fare on a fixed eval
  protocol? Both claim SOTA but with different setups. Worth a comparison
  page entry — see [[cmp-tap-methods]].

## Connections

- Method: [[spatialtracker-v2]]. Predecessor: SpatialTracker (defer
  promotion until 2nd mention).
- Contemporary 3D competitor: [[zhang-2025-tapip3d]].
- Concepts: [[3d-point-tracking]].
- Author: [[nikita-karaev]] (also CoTracker / CoTracker3 lead — third
  appearance in this batch). Senior: Xiaowei Zhou (Zhejiang) — defer
  entity.
- Orgs: Zhejiang University (lead), [[oxford-vgg]] (Karaev/Wang), Ant
  Group, Pixelwise AI, Bytedance Seed.
- Datasets: [[tapvid-3d-dataset]], [[kubric-dataset]],
  [[pointodyssey-dataset]], VKITTI (defer).
- Frequently-cited foundation models (defer entities until 2nd mention):
  DepthAnything, MoGe, MegaSaM, MonST3R, VGGT, DUSt3R.

## Citation

Xiao, Y., Wang, J., Xue, N., Karaev, N., Makarov, Y., Kang, B., Zhu, X.,
Bao, H., Shen, Y., & Zhou, X. (2025). *SpatialTrackerV2: 3D Point
Tracking Made Easy.* arXiv:2507.12462v2.
https://arxiv.org/abs/2507.12462
