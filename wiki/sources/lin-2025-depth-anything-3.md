---
type: source
source_type: paper
title: "Depth Anything 3: Recovering the Visual Space from Any Views"
authors:
  - Lin, Haotong
  - Chen, Sili
  - Liew, Jun Hao
  - Chen, Donny Y.
  - Li, Zhenyu
  - Shi, Guang
  - Feng, Jiashi
  - Kang, Bingyi
year: 2025
venue: arXiv (cs.CV)
url: https://arxiv.org/abs/2511.10647
raw_path: papers/Lin et al. - 2025 - Depth Anything 3 Recovering the Visual Space from Any Views.pdf
status: ingested
tags: [3d-reconstruction, depth-estimation, feed-forward, transformer, monocular, foundation-model, teacher-student]
sources:
  - "[[wang-2025-vggt]]"
related:
  - "[[depth-anything-3]]"
  - "[[vggt]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[bingyi-kang]]"
  - "[[haotong-lin]]"
created: 2026-05-24
updated: 2026-05-24
---

# Depth Anything 3: Recovering the Visual Space from Any Views

## TL;DR

DA3 takes the minimalist stance: a **single plain transformer (vanilla
DINO encoder)** trained to predict **just two things — depth and ray
maps** — is sufficient to reconstruct 3D geometry from any number of
views, with or without poses. Beats [[vggt]] by **+35.7% camera pose
accuracy** and **+23.6% geometric accuracy** on a new visual-geometry
benchmark, while also beating Depth Anything 2 on monocular depth.
ByteDance Seed, **Bingyi Kang** lead — same lead as [[spatialtracker-v2]].

## Why it matters

DA3 is the **counterargument to MapAnything/VGGT multi-task complexity**:
you don't need 12 task heads, you don't need point-map prediction, you
don't need camera tokens. You need depth + rays + a good backbone.

Two insights that simplify the design space:

1. **Singular depth-ray prediction target obviates multi-task learning.**
   Predicting `(depth, ray map)` per pixel is *complete* — camera pose,
   point clouds, etc. are all derivable from this.
2. **A vanilla transformer backbone (DINO) is sufficient.** No
   alternating attention, no dual heads, no special tokens.

This is a Hinton-style "the bitter lesson" paper for 3D reconstruction:
scale + simple prediction target + standard backbone > clever
architecture.

## Key claims

- **Depth + ray maps is sufficient.** Any 3D quantity (point cloud,
  pose, MVS-style geometry) derives from this representation.
- **Vanilla DINO transformer suffices.** No bespoke architecture.
- **Teacher-student training paradigm** matches DA2 on monocular depth
  (the previous best at monocular detail) while expanding to multi-view.
- **+35.7% camera pose accuracy** over VGGT (averaged across HiRoom,
  ETH3D, ScanNet++, DTU, 7Scenes).
- **+23.6% geometric accuracy** over VGGT on multi-view reconstruction.
- **Beats DA2** in monocular depth estimation while being a multi-view
  model — no monocular performance sacrifice.
- **Trained exclusively on public academic datasets** (no internal data).
- Supports **feed-forward 3D Gaussian Splatting** as an application
  (pose-conditioned and pose-adaptive variants).
- Introduces a **Visual Geometry Benchmark** for camera pose +
  any-view geometry + visual rendering, designed to be the successor to
  VGGT's evaluation.

## Methods

- **Architecture:** standard pretrained DINO encoder + DPT decoder.
  No special 3D modules.
- **Prediction targets:** per-pixel depth + per-pixel ray (direction in
  world frame).
- **Training:**
  - Teacher: a larger model trained on a curated mix of multi-view
    public datasets.
  - Student: the deployed DA3 model, distilled from the teacher.
  - Separate variants: DA3 (multi-view), DA3-Monocular (just images),
    DA3-Metric (with metric scale).
- **Applications:** feed-forward 3DGS via the predicted depth + ray
  maps (pose-conditioned or pose-adaptive).

## Results (headline)

- **Visual Geometry Benchmark** (new): +35.7% pose accuracy /
  +23.6% geo accuracy vs [[vggt]]; also beats π³ (Pi3).
- **Monocular depth** (HiRoom, others): beats DA2 — the previous SOTA
  monocular depth model.
- **Feed-forward 3DGS:** competitive with optimization-based 3DGS
  pipelines.

## Limitations / open questions

- Static scenes; doesn't address dynamics (where [[v-dpm]], [[any4d]],
  [[4rc]] live).
- The "depth + rays is sufficient" claim depends on the benchmark mix —
  some specialized 3D tasks (e.g., feature-matching for downstream
  SLAM) may still benefit from explicit pointmap heads.
- Teacher-student distillation requires a strong teacher; full
  reproduction needs the teacher model release.

## Connections

- Method: [[depth-anything-3]].
- Concepts: [[feed-forward-3d-reconstruction]],
  [[pointmap-representation]] (counterargument — DA3 argues
  pointmaps aren't needed).
- **Direct competitor:** [[vggt]] (beaten by 35.7%/23.6%).
- **Cited as foundation by:** [[4rc]] (uses DA3 as base-geometry
  predictor in its conditional decoder), [[trace-anything]] (lists DA3
  in the "feed-forward DUSt3R lineage").
- **Predecessor:** Depth Anything 2 — still used as a separate
  monocular-only model; DA3 supersedes it for multi-view but matches
  on mono.
- **Same lead author lineage:** [[bingyi-kang]] is also on
  [[spatialtracker-v2]] — ByteDance Seed group.

## Citation

Lin, H., Chen, S., Liew, J. H., Chen, D. Y., Li, Z., Shi, G., Feng, J., &
Kang, B. (2025). *Depth Anything 3: Recovering the Visual Space from Any
Views.* arXiv:2511.10647. https://arxiv.org/abs/2511.10647
