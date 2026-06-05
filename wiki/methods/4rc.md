---
type: method
title: 4RC (4D Reconstruction via Conditional Querying)
status: stub
tags: [4d-reconstruction, transformer, feed-forward, conditional-query, scene-flow, video-reconstruction]
sources:
  - "[[luo-2026-4rc]]"
related:
  - "[[4d-reconstruction]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[vggt]]"
  - "[[depth-anything-3]]"
created: 2026-05-24
updated: 2026-05-24
---

# 4RC (4D Reconstruction via Conditional Querying)

**Encode-once, query-anywhere-and-anytime** feed-forward 4D
reconstruction. A transformer encodes the whole video into a compact
4D latent; a conditional decoder queries that latent for any source
frame × any target time, emitting **base geometry + relative 3D motion**.

## One-line summary

ViT encoder over patchified video frames (+ camera tokens) → compact 4D
latent → conditional decoder with target-time token (AdaLN) → per-query
geometry head (depth, ray, camera) + motion head (`ΔP`).

## Inputs / outputs

- **In:** monocular video `V = {I_i}_{i=1..N}`; query `(source frame i,
  target time τ)`.
- **Out:** `P^{t_i → τ}_i = P^{t_i}_i + ΔP^{t_i → τ}_i` — the 3D
  positions of pixels in frame i as they appear at time τ.

## How it works

1. **Patchify + encode:** ViT encoder over all video frames; camera
   tokens added; outputs latent tokens.
2. **Conditional decoder (K blocks):**
   - For each query `(i, τ)`: AdaLN-condition on target-time token `T̂_τ`.
   - Attend across spatial + temporal context in the latent.
3. **Heads:**
   - **Geometry head:** depth + ray + camera for the query view.
   - **Motion head:** `ΔP^{t_i → τ}_i` — per-pixel 3D displacement
     from source time to target.
4. **Factorized 4D representation:** base geometry (viewpoint-invariant)
   + time-dependent relative motion. Temporally consistent in static
   regions / under rigid motion.
5. **Pretraining backbone:** leverages [[depth-anything-3]] for the
   monocular base-geometry representation.

## Where it's been applied

- Camera pose estimation, video depth, point cloud reconstruction, 3D
  point tracking, dense motion modeling.

## Known limitations

- Compact 4D latent may bottleneck very long videos — no scaling study.
- Motion factorization (base + displacement) assumes geometry stays the
  same shape — topology change / appearing-disappearing objects may
  break it.

## Related methods

- **Architectural lineage:** [[vggt]] encoder pattern;
  [[depth-anything-3]] for base geometry.
- **Concurrent feed-forward 4D (with explicit critique in the paper):**
  [[v-dpm]] (high cost, inflexible decoding),
  [[trace-anything]] (Bézier struggles with high-frequency motion),
  [[any4d]] (only first-frame scene flow).
- **3D tracking via 4D:** dense, queryable trajectories — compares
  favorably to sparse [[tapip3d]], [[spatialtracker-v2]].
