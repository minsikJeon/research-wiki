---
type: source
source_type: paper
title: "VGGT: Visual Geometry Grounded Transformer"
authors:
  - Wang, Jianyuan
  - Chen, Minghao
  - Karaev, Nikita
  - Vedaldi, Andrea
  - Rupprecht, Christian
  - Novotny, David
year: 2025
venue: arXiv (cs.CV) — CVPR 2025 candidate
url: https://arxiv.org/abs/2503.11651
raw_path: papers/Wang et al. - 2025 - VGGT Visual Geometry Grounded Transformer.pdf
status: ingested
tags: [3d-reconstruction, feed-forward, transformer, pointmap, depth-estimation, camera-pose, point-tracking, foundation-model]
sources: []
related:
  - "[[vggt]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[pointmap-representation]]"
  - "[[jianyuan-wang]]"
  - "[[nikita-karaev]]"
  - "[[andrea-vedaldi]]"
  - "[[christian-rupprecht]]"
  - "[[oxford-vgg]]"
  - "[[meta-ai]]"
created: 2026-05-24
updated: 2026-05-24
---

# VGGT: Visual Geometry Grounded Transformer

## TL;DR

VGGT is a **single large transformer** that ingests 1–hundreds of images and
outputs **all key 3D quantities at once** — camera intrinsics + extrinsics,
depth maps, point maps, AND 2D point tracks — in **under one second**, with
**no test-time optimization** and no special 3D inductive biases beyond
alternating frame-wise / global attention. Beats DUSt3R / MASt3R /
VGGSfM substantially on every task it touches. **Foundational paper** for
the 3D-reconstruction-foundation-model era — nearly every paper in this
batch builds on or contrasts with VGGT.

## Why it matters

This is the **ancestor architecture** for the 3D/4D-reconstruction sub-field
that emerged in 2025-2026. Before VGGT: DUSt3R-style pairwise predictions
needed post-hoc global alignment. After VGGT: feed-forward all-at-once
inference over arbitrary view counts. Within this wiki, all of [[any4d]],
[[mapanything]], [[v-dpm]], [[depth-anything-3]], [[4rc]], [[trace-anything]],
[[cut3r]] cite VGGT as either backbone or comparison baseline.

Three specific load-bearing claims:

1. **No 3D inductive biases are needed.** A vanilla transformer (with
   alternating attention) outperforms DUSt3R, MASt3R, VGGSfM. The 3D
   knowledge is in the data, not the architecture.
2. **Joint prediction of "redundant" quantities helps.** Predicting
   depth + point maps + camera + tracks simultaneously is better than
   any subset alone, even when one quantity can be derived from another.
3. **VGGT features generalize.** Using VGGT as a frozen feature backbone
   improves downstream tasks (non-rigid tracking, novel view synthesis) —
   a true foundation model.

## Key claims

- **Architecture:** standard ViT (DINO patchification) + alternating
  frame-wise & global self-attention blocks; camera token prepended;
  separate heads for camera, depth (DPT), point map (DPT), tracking
  features.
- **Permutation-equivariant except for the first frame** (which serves
  as world-coord reference).
- **Over-complete predictions are better than parsimonious.** Predicting
  point maps directly *and* deriving them from depth+camera both work,
  but the latter is *more accurate* at inference time. Still trained
  with both heads.
- **Outperforms DUSt3R / MASt3R / VGGSfM by large margins** across
  camera estimation, multi-view depth, point cloud reconstruction, 3D
  point tracking.
- **VGGT features + frozen point tracker** = SOTA tracking (validates
  the foundation-model claim).

## Methods

- **Input:** N RGB images, no camera info needed.
- **Encoder:** DINO ViT patches each image → tokens.
- **Backbone:** L blocks alternating between (a) per-frame self-attention
  and (b) global cross-frame self-attention.
- **Heads:**
  - **Camera head:** small head on a learnable camera token per frame
    → (q, t, f) = quaternion + translation + focal length.
  - **Dense heads (DPT):** per-pixel depth `D_i`, point map `P_i` in
    world frame (first camera = origin), tracking features `T_i`.
- **Tracking module:** separate small network ingests query points + the
  tracking features → emits per-frame 2D tracks. Trained jointly with
  the main backbone.
- **Training:** large mix of public 3D-annotated datasets (mostly static
  scenes).

## Results (headline)

- Beats DUSt3R + MASt3R + VGGSfM on standard benchmarks across
  camera pose, multi-view depth, dense pointcloud reconstruction, and
  3D point tracking.
- Inference: <1 sec for hundreds of images on a single GPU.
- Often beats optimization-based methods without any post-processing;
  adding BA on top is a further win.

## Limitations / open questions

- Trained primarily on **static scenes**; dynamic scenes are out of
  distribution. [[v-dpm]], [[any4d]], [[4rc]] all address this by
  fine-tuning VGGT or extending its design with dynamic heads.
- Reproduction requires very large 3D-annotated training mix; non-trivial
  for academic labs without big training infra.
- π³ (cited concurrent work) removes VGGT's first-frame-as-reference
  asymmetry. Not yet ingested.

## Connections

- Method: [[vggt]].
- Concepts: [[feed-forward-3d-reconstruction]], [[pointmap-representation]].
- **Direct extensions in this wiki:**
  - [[v-dpm]] — adds Dynamic Point Maps heads to VGGT for 4D.
  - [[mapanything]] — VGGT-style multi-view + factored representation
    + metric scale + flexible inputs.
  - [[4rc]] — VGGT-style backbone + conditional query decoder for 4D.
  - [[depth-anything-3]] — competitor that strips VGGT further to a
    single depth+ray head; beats VGGT 35.7% pose / 23.6% geometry.
- **Used as backbone by:** [[spatialtracker-v2]] (cited for
  alternating-attention).
- **Foundation-model lineage to add when ingested:** DUSt3R, MASt3R,
  VGGSfM, MoGe, MegaSaM, Pi3 (π³), π3 finetunes VGGT.
- Authors: [[jianyuan-wang]] (lead), [[nikita-karaev]] (also CoTracker
  series), [[andrea-vedaldi]] + [[christian-rupprecht]] (Oxford VGG
  seniors).

## Citation

Wang, J., Chen, M., Karaev, N., Vedaldi, A., Rupprecht, C., & Novotny, D.
(2025). *VGGT: Visual Geometry Grounded Transformer.* arXiv:2503.11651.
https://arxiv.org/abs/2503.11651
