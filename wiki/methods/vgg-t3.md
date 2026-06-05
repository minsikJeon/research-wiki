---
type: method
title: VGG-T3 (Visual Geometry Grounded Test-Time Training)
status: stub
tags: [3d-reconstruction, feed-forward, vggt, test-time-training, linear-complexity, scalability, visual-localization, kv-compression]
sources:
  - "[[elflein-2026-vgg-t3]]"
related:
  - "[[vggt]]"
  - "[[test-time-training]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[cut3r]]"
  - "[[mapanything]]"
created: 2026-05-29
updated: 2026-05-29
---

# VGG-T3 (Visual Geometry Grounded Test-Time Training)

`O(n)` global / offline / unordered feed-forward 3D reconstruction.
Replaces VGGT's variable-length KV scene representation in global
attention layers with a fixed-size SwiGLU MLP fit at test time. Same
quality as `O(n²)` baselines on most benchmarks; 11.6× faster at 1k
images.

## One-line summary

Frozen [[vggt]] backbone + replace softmax(QK^T)V with `MLP(q)` whose
weights `W` are TTT-fit to reconstruct `(k, v)` over the whole image
collection. `O(n)` apply, mini-batched gradient updates allow single-GPU
huge-collection processing and DDP-style multi-GPU inference. Side
output: the fitted MLP is a queryable scene → feed-forward visual
localization.

## Inputs / outputs

- **In:** unordered image collection `{I_i}_{i=1..N}`.
- **Out:** per-image pose `P_i = (R_i, t_i)`, intrinsics `K_i`, dense
  depth `D_i ∈ R^{H×W}`.
- **Bonus out:** fitted MLP weights `W*` — a compressed, queryable
  scene representation.

## How it works

1. **Tokenize.** Run frozen VGGT encoder → per-image tokens.
2. **In every global attention layer:**
   - Compute `Q, K, V` projections (these QKV projections *are* trained).
   - Apply `ShortConv2D` on `V` to get spatial neighborhood `V'`.
   - TTT-fit `W`: `min_W Σ_i L_t(f_W(k_i), v'_i)` via Muon optimizer.
     Dot-product loss: `L_t(T_θ(k), v') = T_θ(k)^T v'`.
   - Apply: replace softmax(QK^T)V with `f_W(q_i)`.
3. **Decode per-image depth + camera** from the resulting tokens via
   the original VGGT heads.
4. **(Localization mode)** For a new query view `I_new`:
   - Run forward, get `q_new`.
   - Apply `f_{W*}(q_new)` — `W*` frozen, no update.
   - Decode pose from the resulting tokens.

## Why it's `O(n)`

- Softmax attention: `softmax(QK^T)V` is `O(n²)` because every query
  attends to every key.
- TTT: per-image apply is `O(1)` in `n` (just runs the MLP).
- Total: `O(n)` forward + `O(n)` TTT updates.

## Mini-batched TTT (key engineering)

`dL/dW = Σ_s Σ_{i ∈ s} d/dW L_t(k_i, v_i)` decomposes over mini-batches
`s`. Implications:

- **Single GPU + large collection:** shard images across CPU memory,
  load mini-batches to GPU one at a time, accumulate gradient.
- **Multi-GPU:** DDP-style. Each GPU processes a shard, then sync only
  the MLP weights (small) — no all-to-all KV communication needed.
  Linear scaling with GPU count.

## Where it's been applied

- Pointmap estimation: 7scenes, DTU, ETH3D, NRGBD.
- Video depth: Bonn, KITTI, Sintel.
- Camera pose: ScanNet, Sintel, TUM-RGBD.
- Large-scale 7scenes: up to 2k images, ordered and unordered.
- Visual localization: 7scenes, Wayspots.
- Rome-landmark qualitative: 1k+ tourist photos in <1 min.

## Known limitations

- Gap vs softmax attention in wide-baseline scenes.
- Camera pose less accurate than `O(n²)` baselines (likely VGGT's
  camera-token modality is hard for MLP to memorize).
- Per-scene TTT optimization required at deployment (~100 iters,
  <1 min for 1k images). Changes the deployment model from pure
  feed-forward → fit-then-query.
- Requires VGGT pretraining; training from scratch collapses.
- Below ~500 images, `O(n²)` baselines win on quality.

## Related methods

- **Backbone:** [[vggt]] (frozen).
- **Mechanism:** [[test-time-training]] — same MLP-as-fast-weight
  paradigm as TTT3R, but applied to *global* attention rather than
  recurrent/autoregressive context.
- **Concurrent autoregressive TTT baseline:** **TTT3R** [Chen et al. 2026]
  — frame-wise streaming. VGG-T3 differs in being offline/global/
  unordered.
- **Quadratic-attention efficient siblings:** **FastVGGT** (token
  merging), **SparseVGGT** (block-sparse).
- **Long-sequence siblings (chunked / recurrent):** **VGGT-Long**,
  **VGGT-SLAM**, **Slam3R**, **Stream3R**, **StreamVGGT**, [[cut3r]],
  **MUSt3R**, **Point3R**, **Long3R**, [[mapanything]].
- **Localization baseline:** **Reloc3R** (classical pipeline; higher
  accuracy but not feed-forward).
- **In the context of [[loger]]:** LoGeR uses chunked SWA + TTT for
  streaming long-context; VGG-T3 uses MLP-TTT for *global* offline.
  Two different ways of compressing scene state into fast weights.
