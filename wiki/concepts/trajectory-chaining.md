---
type: concept
title: Trajectory Chaining
status: stub
tags: [4d-reconstruction, long-sequence, trajectory-chaining]
sources:
  - "[[anon-2026-point4d]]"
  - "[[harley-2022-pips]]"
related:
  - "[[4d-reconstruction]]"
  - "[[3d-point-tracking]]"
  - "[[point4d]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[pips]]"
created: 2026-05-26
updated: 2026-06-26
---

# Trajectory Chaining

## Definition

A strategy for extending **short-window 4D / 3D reconstruction models** to
**long sequences** (hundreds of frames) by:

1. Partitioning the video into overlapping chunks.
2. Running the short-window model independently on each chunk.
3. Aligning chunk-local coordinate frames into a global frame (typically
   via `Sim(3)` from dense depth on shared overlap frames).
4. **Propagating per-point identity** across chunk boundaries so
   trajectories stitch correctly.

The static-3D version of (1)-(3) is well known
(VGGT-Long, InfiniteVGGT, StreamingVGGT). The 4D version requires (4),
which is the hard part — and where representation choice matters most.

**2D-point-tracking ancestor.** [[harley-2022-pips]] §3.7 is the original
chunked-window chaining recipe in deep TAP: re-initialize from the
latest high-visibility timestep (threshold starts at 0.99, decreases
until a valid pivot is found), and **re-use the original query feature
across re-initializations** to lock identity onto the target. The same
"propagate identity across chunk boundaries" challenge — and the same
failure mode (chain breaks when the target is occluded across the
entire chunk) — is what 4D methods rediscover at a different
granularity.

## Why it matters

The current feed-forward 4D wave ([[any4d]], [[4rc]], [[v-dpm]],
[[trace-anything]]) is **bottlenecked at ~100 frames** by transformer
memory and training data length. Real videos in robotics / AR /
generative AI are routinely longer. Chaining is the obvious lever.

## The 2D-vs-3D-query problem (point4d's insight)

Per-point identity propagation is the failure mode that distinguishes
chaining strategies.

- **2D-pixel-query methods** ([[d4rt]], [[trace-anything]], [[4rc]],
  [[any4d]], [[v-dpm]]): to re-query a point in the next chunk, you
  must project its predicted 3D position back to a pixel. **Undefined
  for occluded / off-frame points** (the very situations long videos
  contain). For visible points, reprojection compounds camera-prediction
  error.
- **3D-coordinate-query methods** ([[point4d]]): the 3D coordinate is
  well-defined regardless of visibility. Re-query directly by `Sim(3)`-
  transforming the endpoint. No reprojection, no matching.

Empirically ([[anon-2026-point4d]] Fig 6): 2D-query methods compound
error at every chunk boundary; 3D-query methods degrade slowly.

## Variants

- **Static 3D chaining** (cited in [[point4d]]): VGGT-Long (loop
  closure), InfiniteVGGT (streaming), StreamingVGGT, Loger (test-time
  training with hybrid memory). All require only Sim(3) alignment of
  the chunk geometries — no point-identity propagation.
- **2D-query 4D chaining (reprojection-based):** the baseline for
  Any4D, 4RC, V-DPM, Trace Anything in long-video evals. Picks the
  overlapping frame with best depth agreement for re-projection
  (selection-based).
- **3D-query 4D chaining ([[point4d]]):** Sim(3) endpoint
  transform + visual descriptor reuse.

## Open questions

- **Depth error compounding.** All chaining strategies (3D included)
  depend on dense depth being right at chunk overlap. How does depth
  error propagate over many chains?
- **Re-acquisition of points absent from all frames in a chunk.**
  Even 3D queries can't track a point that's invisible across the
  entire chunk. Need scene-level memory (CUT3R-style) to recover.
- **Memory-based vs chaining-based long-range models.** CUT3R uses
  a persistent state; Point4D uses overlapping-chunk chaining. Open
  whether these can be combined.
- **3D-query chaining for 3D point trackers** (TAPIP3D, SpatialTrackerV2).
  These are already 3D-query in spirit but use iterative refinement.
  The chaining lever applies but isn't standard practice yet.
