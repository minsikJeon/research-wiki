---
type: method
title: MapAnything
status: stub
tags: [3d-reconstruction, feed-forward, transformer, metric-scale, multi-modal, sfm, foundation-model]
sources:
  - "[[keetha-2025-mapanything]]"
related:
  - "[[feed-forward-3d-reconstruction]]"
  - "[[pointmap-representation]]"
  - "[[vggt]]"
  - "[[any4d]]"
created: 2026-05-24
updated: 2026-05-24
---

# MapAnything

Universal feed-forward 3D reconstructor: one model solves 12+ tasks
(unconstrained SfM, calibrated SfM, MVS, monocular depth, camera
localization, depth completion, …) with **metric-scale outputs** and
**flexible multi-modal inputs**.

## One-line summary

DINOv2 image encoder + dedicated encoders for ray maps / poses / depth →
alternating-attention transformer (VGGT-style) → DPT decoder for per-view
ray + depth + masks + confidences, pose head for camera poses, MLP for
global metric scale factor.

## Inputs / outputs

- **In:** N images + any subset of {ray maps, poses, depth} per view.
- **Out:** per-view ray directions, ray depths, masks, confidences,
  pose (in view-1 frame), and a single global metric scale factor.
- Recover full metric pointmap from these.

## How it works

1. **Encoders:** DINOv2 for images; ray encoder; pose encoder (rotation,
   translation, scale); depth encoder. Patch features summed across
   modalities.
2. **Reference embedding** added to view 1; learnable scale token
   appended.
3. **Backbone:** alternating-attention transformer (per-frame + global).
4. **Heads:**
   - Single DPT decodes all N views' dense outputs (rays, depth, masks,
     confidence).
   - Average-pooling pose head → camera in view-1 frame.
   - MLP on scale token → global metric scale.
5. **Training:** flexible input augmentation (randomly drop modalities)
   so the model learns to handle any subset at inference.

## Where it's been applied

- Tested on 12+ task configurations: uncalibrated SfM, calibrated SfM,
  posed SfM, MVS, monocular depth, camera localization, depth completion,
  RGB-D localization, ...
- Matches or beats specialist feed-forward models per task.

## Known limitations

- Static scenes only.
- "Matches or beats" — not always a decisive win per individual task;
  value is in unification + metric + flexibility.
- Heavy training-data aggregation (17+ datasets) is reproduction barrier.

## Related methods

- **Closest competitor:** [[vggt]] (less flexible inputs, no metric scale).
- **Architectural lineage:** VGGT's alternating attention.
- **Sister 4D paper:** [[any4d]] — same authors + extends to 4D.
- **Counterargument:** [[depth-anything-3]] (strips further to depth+ray
  only).
- **Apache 2.0 release.**
