---
type: method
title: Point4D
status: stub
tags: [4d-reconstruction, long-sequence, 3d-query, trajectory-chaining, feed-forward, dense-tracking]
sources:
  - "[[anon-2026-point4d]]"
related:
  - "[[4d-reconstruction]]"
  - "[[3d-point-tracking]]"
  - "[[trajectory-chaining]]"
  - "[[depth-anything-3]]"
  - "[[d4rt]]"
  - "[[shubham-tulsiani]]"
created: 2026-05-26
updated: 2026-05-29
---

# Point4D

> **User's work (private note):** Lead-authored by the user
> (Minsik Jeon) under advisor [[shubham-tulsiani]] at [[cmu-ri]];
> NeurIPS 2026 under double-blind review. Source page kept anonymous
> in its formal frontmatter ([[anon-2026-point4d]]) to mirror venue
> review status.


Feed-forward long-range (200+ frame) 4D motion reconstruction. Decodes
per-point 3D trajectories from **3D coordinate queries** (not 2D pixel
queries) so that predicted endpoints can be directly re-queried across
overlapping chunks under occlusion / out-of-frame events.

## One-line summary

ViT encoder (initialized from [[depth-anything-3]]) → scene representation
+ depth + camera. Lightweight cross-attention decoder takes
(3D coord, source/target/camera times, visual patch from any visible frame)
→ predicts 3D position at target time. Chain across overlapping chunks via
`Sim(3)` alignment + direct re-querying of predicted 3D endpoints.

## Inputs / outputs

- **In:** monocular video `V` (up to hundreds of frames); query pixels
  `{(u_i, v_i)}` visible in some reference frame; chunk size + overlap.
- **Out:** dense per-point 3D trajectories `{P_i(t)}` in a global
  coordinate frame.

## How it works

1. **Chunk + encode.** Partition `V` into K overlapping chunks. ViT
   encoder runs per-chunk → scene representation `F_k`, per-frame
   depth `D_i`, camera pose `P_i`.
2. **Initial 3D query.** For each query pixel `(u_i, v_i)`, unproject
   via depth at the reference frame to get `p_i ∈ R^3`. Extract visual
   descriptor `S_i` (RGB patch) once.
3. **Per-chunk decode.** For each frame `t` in chunk `k`, the decoder
   `D(p_i, S_i, F_k, π_t) → P_i(t)`. Query embedding = Fourier-encoded
   `p_i` + camera/time-token MLPs + `S_i`. Cross-attention to `F_k`
   with no inter-query self-attention (independent batching).
4. **Chain.** Estimate `Sim(3)` between chunks `k` and `k+1` from
   shared overlap depth. Transform predicted endpoint `P_i(t_h)` into
   chunk `k+1`'s coordinate frame to re-query. **Re-use the same**
   `S_i` (visual descriptor never changes).
5. **Loss.** `L_point` (L1 under signed-log) + `L_conf` (confidence-
   modulated) + `L_2d` (reprojection consistency) + `L_vis`
   (visibility BCE).

## Where it's been applied

- **Long-video tracking** (200-frame seqs): PointOdyssey, Dynamic
  Replica, PStudio (from [[tapvid-3d-dataset]]). SOTA among
  feed-forward methods.
- **Single-chunk tracking** (up to 64-frame seqs): TAPVid3D
  PStudio, LSFOdyssey, Dynamic Replica. Comparable to V-DPM / 4RC /
  Any4D.

## Known limitations

- **Point must be visible in ≥1 frame per chunk.** A point absent from
  all frames in a chunk can't be tracked within it.
- **Depth-prediction error compounds across chunks** through `Sim(3)`
  alignment.
- **Chunk size / overlap is a hyperparameter** (default 48 / 8).
- No direct comparison with offline / optimization 4D methods.

## Related methods

- **Direct predecessor:** [[d4rt]] ([[zhang-2025-d4rt]]) — Point4D
  extends D4RT's query-based decoder from 2D-pixel queries to
  3D-coordinate queries. Same encoder, same independent cross-attention,
  same SRT lineage.
- **Backbone init:** [[depth-anything-3]].
- **Feed-forward 4D competitors (single-chunk):** [[any4d]], [[4rc]],
  [[v-dpm]], [[trace-anything]] — all <100 frames before Point4D.
- **3D point-tracking competitors (iterative):** [[tapip3d]],
  [[spatialtracker-v2]] — sparse-only.
- **Long-sequence 3D ancestors:** [[cut3r]], InfiniteVGGT, VGGT-Long,
  StreamingVGGT, Loger — all static 3D; Point4D extends the chunking
  idea to 4D.
