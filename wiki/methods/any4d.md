---
type: method
title: Any4D
status: stub
tags: [4d-reconstruction, feed-forward, transformer, metric-scale, multi-modal, scene-flow, robotics, dense-tracking]
sources:
  - "[[karhade-2025-any4d]]"
related:
  - "[[4d-reconstruction]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[mapanything]]"
created: 2026-05-24
updated: 2026-05-24
---

# Any4D

Feed-forward dense metric 4D reconstruction in one pass. Outputs scene
flow + camera poses + per-view geometry from N video frames with optional
multi-modal sensor inputs (RGB-D, IMU, Doppler).

## One-line summary

MapAnything-style multi-view transformer + dynamic outputs: per-view ray
maps, scale-normalized depth, world-frame camera pose, world-frame
forward scene flow from view 1, and a global metric scale factor — all
in a single forward pass, 15× faster than next-best 4D method.

## Inputs / outputs

- **In:** N RGB frames + optional `O = {RGB-D, intrinsics, IMU poses,
  Doppler velocity}` per view.
- **Out per view `i`:** ray dirs `R̃_i`, scale-normalized depth `D̃_i`,
  pose in view-1 frame `T̃_i`, scene flow from view 1 to view i `F̃_i`.
- **Global:** metric scale `s̃ ∈ ℝ`.
- **Derived:** metric pointmap `G̃_i = s̃ · T̃_i · R̃_i · D̃_i`;
  motion-applied pointmap `G̃'_i = G̃_i + s̃ · F̃_i`.

## How it works

1. **Encoders:** image + multi-modal encoders (same approach as
   [[mapanything]]) — DINOv2 + dedicated encoders for ray/pose/depth/
   IMU/Doppler.
2. **Backbone:** alternating-attention transformer over view tokens
   (single feed-forward pass).
3. **Heads:** per-view DPT-style heads for `R̃`, `D̃`, `T̃`, `F̃`; global
   MLP for `s̃`.
4. **Factored representation:** egocentric (rays + depth, in local
   camera) vs allocentric (scene flow + pose, in world). This factoring
   lets training mix metric-no-motion datasets with non-metric-with-motion
   datasets — partial supervision works across the factors.

## Where it's been applied

- TAPVid-3D (3D point tracking via dense scene flow).
- 4D scene reconstruction in robotic / driving / general video contexts.
- Multi-modal demos: RGB-D, IMU, Radar Doppler inputs.

## Known limitations

- Scene flow is "from view 1 to view i" only — long-range symmetric
  trajectories may need additional handling. [[4rc]] critique:
  "only predicts scene flow for the first frame."
- Multi-modal training datasets are scarce; most ablations use RGB-only
  baseline.
- Concurrent with [[v-dpm]], [[4rc]], [[trace-anything]] — no unified
  4D eval protocol exists.

## Related methods

- **Sister static paper:** [[mapanything]] (same authors, static
  reconstruction). Any4D = MapAnything + time + scene flow.
- **Concurrent feed-forward 4D:** [[v-dpm]], [[4rc]], [[trace-anything]].
- **Sparse 3D tracking competitors:** [[tapip3d]],
  [[spatialtracker-v2]] (these are sparse; Any4D is dense).
- **MegaSaM:** per-scene optimization; Any4D is 15× faster.
