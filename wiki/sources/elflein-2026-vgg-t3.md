---
type: source
source_type: paper
title: "VGG-T3: Offline Feed-Forward 3D Reconstruction at Scale"
authors:
  - Elflein, Sven
  - Li, Ruilong
  - Agostinho, Sérgio
  - Gojcic, Zan
  - Leal-Taixé, Laura
  - Zhou, Qunjie
  - Osep, Aljosa
year: 2026
venue: arXiv (cs.CV)
url: https://arxiv.org/abs/2602.23361
raw_path: papers/2602.23361v1.pdf
status: ingested
tags: [3d-reconstruction, feed-forward, vggt, test-time-training, linear-complexity, scalability, visual-localization, kv-compression, scene-representation]
sources:
  - "[[wang-2025-vggt]]"
  - "[[wang-2025-cut3r]]"
  - "[[keetha-2025-mapanything]]"
related:
  - "[[vgg-t3]]"
  - "[[vggt]]"
  - "[[test-time-training]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[nvidia]]"
created: 2026-05-29
updated: 2026-05-29
---

# VGG-T3: Offline Feed-Forward 3D Reconstruction at Scale

## TL;DR

VGG-T3 replaces VGGT's variable-length **KV-space scene representation**
in global attention layers with a **fixed-size MLP** whose weights are
fit at test time (TTT). This converts the asymptotic complexity from
`O(n²)` to `O(n)` w.r.t. number of input views while preserving the
**global** (offline) reconstruction property — distinct from the
chunked / autoregressive linearizations (StreamVGGT, CUT3R, TTT3R).
1k unposed images in **58 s** vs VGGT's **~11 min** (11.6× speedup);
2k images in **48.5 s** (33× vs VGGT's 27 min). Built on top of
pre-trained [[vggt]] (only the QKV projection + new TTT layer
fine-tune); ~12% of VGGT scratch-training cost. Side capability: the
optimized MLP is a *queryable* scene representation → enables
**feed-forward visual localization** for novel query images.

## Why it matters

This is the **first linear-time *offline* (global) reconstruction
method that actually preserves quality** vs softmax-attention
baselines on most metrics. Prior `O(n)` options were either:

- **Chunked** (VGGT-Long, VGGT-SLAM, Slam3R) — decouples the global
  scene state, prone to drift, unsuitable for unordered image sets.
- **Recurrent / autoregressive** (CUT3R, MapAnything, Point3R,
  TTT3R, StreamVGGT) — fixed-size persistent state, but order-
  sensitive and worse on long sequences.
- **Compressed quadratic** (FastVGGT, SparseVGGT) — better constants
  but still `O(n²)` asymptotic.

VGG-T3 carves out a fourth slot: **global**, **unordered-friendly**,
**linear-time**, and **competitive with `O(n²)` quality**. For the
user's research direction (long-horizon 3D tracking, scene-level
grounding for slow-planner / fast-controller designs — see the
`long-term-real-time-3d-tracking` research note in `raw/notes/`),
this is potentially *the* slow-planner candidate — it produces a
**compact, queryable scene representation** at scale, which is
exactly what 1.5.3 of that note argued heavy chunks exist to produce.

## Key claims

- **The KV space *is* the scene representation.** VGGT's global self-
  attention layer maintains an implicit, variable-length scene
  representation in its key-value pairs. Quadratic attention is
  expensive *because* this representation grows with `n`.
- **A fixed-size MLP can replace the variable-length KV.** The MLP
  `f_W(k) → v` is fit at test time to reconstruct `(k, v)` token
  pairs across the whole image collection. After fitting, the MLP
  *is* the compressed scene — querying it is `O(1)` per query.
- **Linear time + linear memory in `n`.** Per-image, the global
  attention layer applies the (constant-cost) MLP instead of softmax
  over all KVs. Total cost is `O(n)` forward + `O(n)` TTT
  optimization steps.
- **Mini-batched TTT enables single-GPU large-scale + distributed
  multi-GPU inference.** Gradient of `L_TTT` decomposes over mini-
  batches (per image subset). Allows (a) processing 2k+ image
  collections on one GPU by CPU off-loading, (b) DDP-style multi-GPU
  with only MLP weight sync (small, no all-to-all KV comms).
- **ShortConv2D on V is critical.** A single-layer 3×3 conv over the
  spatial value tokens (before the TTT objective) improves expressivity:
  the MLP must now predict a *neighborhood* `V'` rather than `V` itself.
  Without it, the model gets stuck in a near-trivial solution.
- **Localization comes for free.** The fitted MLP is queryable with
  novel views (frozen weights, forward only) → feed-forward visual
  localization. Outperforms TTT3R on 7scenes and Wayspots.

## Methods

- **Backbone:** Frozen pre-trained [[vggt]]. Fine-tune only the
  QKV projection of global attention + the new TTT layer.
- **TTT layer:** SwiGLU MLP `f_W: R^d → R^d` as fast weights;
  Muon optimizer; dot-product loss `L_t(T_θ(k_i), v_i) = T_θ(k_i)^T v_i`.
- **Apply / Update procedure** (chunk-level for compatibility, but
  fundamentally global):
  - Apply: `o_i = T_θ(q_i)` — replaces softmax(QK^T)V.
  - Update: `W ← W − η ∇_W L_t` over current batch of (k, v).
- **ShortConv2D** before the TTT objective: spatial 3×3 conv on
  reshaped V tokens to create `V'`. Forces the MLP to predict
  neighborhood structure rather than identity.
- **Mini-batched TTT (Eq. 5):** total gradient decomposes over batch
  shards. Used for both single-GPU large-collection processing
  (CPU off-load) and multi-GPU distributed inference (DDP, no
  all-to-all KV needed).
- **Training:** 100k steps on 8× A100-80GB. Datasets comparable to
  VGGT (Tab. 7: ScanNet/++, ARKitScenes, Co3Dv2, DL3DV, TartanAirV2,
  MegaDepth, etc.). Cost: ~12% of full VGGT pretraining.
- **Visual localization mode:** keep MLP weights frozen after
  scene-fit, run a single forward on the new query view. Only the
  global attention layer of the *query* uses `T_W(q_new) → v` — the
  query never updates `W`.

## Results

### Standard benchmarks (point-map / video-depth / camera-pose)

| | 7scenes-D CD↓ | DTU CD↓ | NRGBD-D CD↓ | KITTI δ<1.25↑ |
|---|---|---|---|---|
| VGGT (`O(n²)`) | 0.024 | 1.537 | 0.014 | 0.964 |
| FastVGGT (`O(n²)`) | 0.021 | 1.683 | 0.033 | 0.964 |
| TTT3R (`O(n)`) | 0.035 | 5.708 | 0.071 | 0.818 |
| **VGG-T3 (`O(n)`)** | **0.030** | **1.654** | **0.029** | **0.967** |

VGG-T3 reduces error vs TTT3R by **2-2.5×** on DTU / ETH3D /
NRGBD-D; competitive with `O(n²)` baselines.

### Large-scale 7scenes (linear scaling)

| Method | 1k imgs | 2k imgs |
|---|---|---|
| VGGT | ~670 s (>11 min) | ~27 min |
| FastVGGT | ~250 s | OOM |
| TTT3R | 90.1 s | 126.2 s |
| **VGG-T3 (1 GPU)** | **58 s** | **173 s** |
| **VGG-T3 (4 GPU)** | **29.7 s** | **48.5 s** |

11.6× faster than VGGT at 1k images; 33× at 2k images.

### Feed-forward visual localization (Tab. 5)

- **7scenes:** Median `er = 6.71°`, 40.7% within 10cm/10° (vs
  TTT3R 7.18°, 34.6%).
- **Wayspots:** Median `er = 32.04°`, 13.4% within 10cm/10°
  (vs TTT3R 74.45°, 0.69%). Substantial gain on outdoor.

### Camera-pose estimation (Tab. 3)

Mixed: TTT3R sometimes wins on *ordered* sequences (because it's
sequential), but VGG-T3 is significantly better on *unordered* —
the realistic regime for tourist-photo / in-the-wild collections.

## Limitations / open questions

- **Wide-baseline gap vs softmax.** The MLP is less expressive than
  full softmax attention for wide-baseline pairs. Authors flag this
  as the main remaining gap (Conclusion).
- **Camera-pose accuracy lags `O(n²)` baselines.** Hypothesis: VGGT's
  dedicated camera token (separate "modality" within the attention)
  is hard for a single MLP to memorize.
- **Test-time optimization cost.** TTT is a per-scene optimization
  step (~100 iterations, < 1 minute for 1k images). Free-running
  per-frame latency is *post-fit*, so this is a one-time amortized
  cost, but it's not free — and it changes the deployment model
  (you fit then query, like NeRF, not pure feed-forward).
- **Pre-trained VGGT dependence.** Linearization-from-scratch
  collapses (Tab. 6 row (i)) — the method only works on top of a
  strong softmax-attention pretraining.
- **`O(n²)` enhanced baselines remain better at small-to-medium
  scale.** Below ~500 images, VGGT-class methods win on quality.
  VGG-T3's regime is the *large-collection* end.

## Connections

- Method page: [[vgg-t3]].
- **Direct backbone:** [[vggt]] — VGG-T3 fine-tunes a frozen VGGT.
- **Core mechanism:** [[test-time-training]] (new concept page) —
  fast-weight MLP fit at inference; same family as TTT3R.
- **Concurrent linear-time baseline:** **TTT3R** [Chen et al. 2026]
  — autoregressive, frame-wise. VGG-T3 is global / offline /
  unordered-friendly, which is the key distinction. Promote to
  primary source if a 2nd reference appears (already referenced by
  LoGeR — promote now).
- **`O(n²)` baselines:** **FastVGGT** [Shen et al. 2026, token-
  merging], **SparseVGGT** [block-sparse], **VGGT-Long** [Deng et al.
  2025], **VGGT-SLAM** [Maggio et al. 2025], **Slam3R**, **Stream3R**,
  **StreamVGGT** [Zhuo et al. 2026]. Most appear in multiple wiki
  sources now — see [[anon-2026-point4d]]'s reference list. Promote
  to wiki methods as 2nd-mention threshold is crossed.
- **Other linear-time recurrent baselines:** [[cut3r]], **Must3R**
  [Cabon et al. 2025], **Point3R**, **Long3R** [Wu et al. 2025],
  [[mapanything]] (explicit spatial memory).
- **Localization comparison:** **Reloc3R** [Dong et al. 2025] is the
  classical-correspondence + PnP comparison; VGG-T3 aims at
  feed-forward, not best-overall.
- **Authors:** Sven Elflein (lead, [[nvidia]] + Vector Institute +
  U. Toronto), Ruilong Li, Sérgio Agostinho, Zan Gojcic, Laura
  Leal-Taixé, Qunjie Zhou, Aljosa Osep — all NVIDIA-affiliated.
- **Org:** [[nvidia]] (new entity page).

## Why this matters for the user's project

VGG-T3 is the cleanest exemplar yet of *"heavy chunks make scene-
level grounding, not better tracks"* (per [[user_role]]'s research
note Part 1.5.3). The fitted MLP is *literally* a compressed scene
representation that can be queried. For a slow-planner / fast-
controller design (Caricature 3), this is exactly the kind of
artifact a planner should produce — a fixed-size, queryable scene
state that survives across the controller's per-frame loop. The
**localization-from-query** capability further suggests how a
controller could "re-anchor" itself via cheap forward queries
against the planner's representation.

## Citation

Elflein, S., Li, R., Agostinho, S., Gojcic, Z., Leal-Taixé, L., Zhou,
Q., & Osep, A. (2026). *VGG-T3: Offline Feed-Forward 3D Reconstruction
at Scale.* arXiv:2602.23361v1 [cs.CV]. https://arxiv.org/abs/2602.23361
