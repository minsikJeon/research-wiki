---
type: method
title: Depth Anything 3 (DA3)
status: stub
tags: [depth-estimation, 3d-reconstruction, feed-forward, transformer, foundation-model, teacher-student]
sources:
  - "[[lin-2025-depth-anything-3]]"
related:
  - "[[feed-forward-3d-reconstruction]]"
  - "[[vggt]]"
created: 2026-05-24
updated: 2026-05-24
---

# Depth Anything 3 (DA3)

Minimalist counter to VGGT/MapAnything. **Single plain transformer**
(vanilla DINO) trained to predict **just depth + ray maps**. From these,
all 3D quantities (point clouds, camera pose, etc.) are derivable.
Beats VGGT by +35.7% pose / +23.6% geometry on a new benchmark.

## One-line summary

DINO ViT encoder + DPT decoder → per-pixel depth + ray map; teacher-
student distillation; trained only on public datasets; no special 3D
inductive biases, no multi-task heads.

## Inputs / outputs

- **In:** 1 to N RGB images, with or without camera poses.
- **Out per view:** dense depth + dense ray map.
- **Derived:** point clouds, camera poses, multi-view fused geometry.

## How it works

1. **Architecture:** standard pretrained DINO encoder + DPT decoder.
2. **Prediction:** per-pixel depth + ray direction (in world frame).
3. **Training (teacher-student):**
   - Teacher: larger model trained on curated public multi-view data.
   - Student: deployed DA3, distilled from teacher.
   - Variants: DA3 (multi-view), DA3-Monocular, DA3-Metric.
4. **Application:** feed-forward 3D Gaussian Splatting via predicted
   depth + ray (pose-conditioned or pose-adaptive).

## Where it's been applied

- A new **Visual Geometry Benchmark** covering camera pose / any-view
  geometry / visual rendering — established alongside the paper.
- HiRoom, ETH3D, ScanNet++, DTU, 7Scenes evaluations.
- Monocular depth (still beats DA2 here despite being multi-view).
- Feed-forward 3DGS.

## Known limitations

- Static scenes only.
- The "depth+ray is sufficient" claim depends on the benchmark mix —
  some downstream uses may still want explicit pointmap heads.
- Teacher model must be released for full reproduction.

## Related methods

- **Predecessor:** Depth Anything 2 — still relevant for monocular-only
  deployment.
- **Direct competitor:** [[vggt]] (beaten by 35.7%/23.6%).
- **Cited as foundation by:** [[4rc]] (uses DA3 as base-geometry
  predictor in conditional decoder).
- **Same lead lineage:** Bingyi Kang (also on [[spatialtracker-v2]]),
  ByteDance Seed group.
