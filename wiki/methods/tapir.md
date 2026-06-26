---
type: method
title: TAPIR
status: growing
tags: [point-tracking, two-stage, iterative-refinement, uncertainty-estimation, depthwise-conv, foundation-paper]
sources:
  - "[[doersch-2023-tapir]]"
related:
  - "[[point-tracking]]"
  - "[[pips]]"
  - "[[tapnext]]"
  - "[[cotracker]]"
  - "[[cotracker3]]"
  - "[[track-on2]]"
  - "[[tap-vid-dataset]]"
  - "[[synthetic-to-real-gap]]"
created: 2026-06-26
updated: 2026-06-26
---

# TAPIR

Two-stage **[[point-tracking]]** method that fuses **TAP-Net's global
per-frame initialization** with **PIPs' iterative temporal refinement**.
Replaces PIPs' fixed-length MLP-Mixer with a **depthwise-conv-over-time**
refinement net (any-length sequences, parallelizable across frames) and
adds a **self-supervised position-uncertainty** head. The reference TAP
implementation since ICCV 2023; direct ancestor of BootsTAPIR /
[[tapnext]] in the DeepMind TAP-line.

## One-line summary

Per-frame cost-volume init + ConvNet heatmap → soft-argmax + occlusion +
uncertainty for every frame independently; then 4 iterations of
local-pyramid score maps + depthwise-conv-over-time refinement that
updates position, occlusion, uncertainty, and query feature.

## Inputs / outputs

- **In:** RGB video `[T, H, W, 3]` + query `(t_q, x_q, y_q)`. Trained
  at 256² with 24-frame clips; deployable at any T (depthwise conv
  removes PIPs' fixed-length constraint).
- **Out:** per-frame `(x_t, y_t)` + occlusion `o_t` + position
  uncertainty `u_t`. Output visibility = `(1 − o_t)(1 − u_t) > 0.5`.

## How it works

### Stage 1 — Track initialization (global, per-frame, parallel)

1. **Backbone** (TSM-ResNet-18 paper / PreResNet-18 + InstanceNorm
   open-source). Feature map `F ∈ R^{T × H/8 × W/8 × C}`.
2. **Query feature:** `F_q = bilinear(F[t_q], x_q, y_q)`.
3. **Cost volume:** dot product `F · F_q` → `(T, H/8, W/8, 1)`.
4. **Per-frame ConvNet** on each frame's cost-volume slice → heatmap
   `(H/8, W/8, 1)` + occlusion logit (avg-pool + project) + position
   uncertainty logit (second avg-pool + project).
5. **Spatial soft-argmax:** softmax over space → set heatmap cells far
   from the argmax to zero → weighted average of positions. The
   thresholding prevents the model averaging across multiple matches.

### Stage 2 — Iterative refinement (depthwise conv over time)

1. **Multi-scale local score maps:** for each frame, extract a 7×7 patch
   of dot products between `F_q` and a spatial pyramid of feature maps
   at strides `[4, 8, 16, 32, ...]` centered at current `p_t`. Shape
   `(T, 7·7·L)` per query.
2. **Refinement input:** `(T, C + K + 4)` = query feat + score maps + `(p, o, u)`.
3. **12-block refinement net.** Each block = `1×1 conv` (cross-channel,
   PIPs Mixer analog) + **depthwise conv** (along time, replaces PIPs'
   within-channel layer; runs at any T).
4. **Update:** `(Δp, Δo, Δu, ΔF_q)` per iteration. Iterate 4 times
   (best). `F_q` evolves across iterations and frames → "slightly
   different query features" per frame after the first iteration.

### Loss (every refinement step + initialization)

```
L = Huber(p̂, p) · (1 − ô)
    + BCE(ô, o)
    + BCE(û, u) · (1 − ô)
with u_t* = 1{‖p̂ − p‖ > δ}   (self-supervised from model output)
```

### High-resolution inference

K log-spaced image resolutions (256² → original). Run global init at
256², then iterate refinement at each level. Re-initialize occlusion
and uncertainty at each level (the model becomes overconfident
otherwise). Average outputs.

### Training data

- **Kubric MOVi-E** (default).
- **Panning MOVi-E** (new): modify the "look-at" point of the Kubric
  camera to follow a random linear trajectory (≤ 1.5 units off-ground,
  radius ≤ 4, passes within 1 unit of center). Restores panning
  variability that vanilla MOVi-E lacks; carried forward to all
  DeepMind TAP successors.

## Where it's been applied

- **TAP-Vid (DAVIS, Kinetics, RGB-Stacking, Kubric)** — the standard
  benchmark; see [[doersch-2023-tapir]] for full numbers.
- **High-resolution TAP-Vid** (DAVIS 1080p, Kinetics 720p) — gains
  ~4 AJ from spatial detail.
- **Animating still images** (§7) — diffusion model trained on TAPIR
  trajectories generates plausible motion from a single frame.
- Inherited as substrate by **BootsTAPIR** (Doersch et al. 2024;
  semi-supervised real-data fine-tune; cited but not yet ingested) →
  [[tapnext]] (recasts as masked-token decoding, drops cost volumes).

## Known limitations

- **No cross-track context.** Per-point inference; inherits PIPs'
  independent-tracks limitation. [[cotracker]] later fills this with
  cross-track attention.
- **Non-causal refinement.** Refinement uses bidirectional temporal
  context; not online. [[tapnext]] / [[track-on2]] later achieve
  causal SOTA.
- **Iteration over-smoothing on textureless objects.** More than 4–5
  iterations hurts RGB-Stacking; the model can drift past correct
  positions. Open mechanism.
- **Synthetic-only training in the original paper.** BootsTAPIR's
  semi-supervised pipeline is needed for best real-video numbers; the
  fix migrates entirely to data, not architecture.
- **Occlusion + uncertainty independently predicted** from cost-volume
  features → heuristic late fusion `(1−o)(1−u) > 0.5`.
- **Engineering-heavy high-resolution inference** (TPU partitioning,
  pyramid re-inits) — replaced by simpler pipelines in later trackers.
- **No 3D.** 2D pixel tracking only.

## Related methods

- **Direct predecessors:**
  - **PIPs** ([[pips]] / [[harley-2022-pips]]) — refinement substrate
    (12-block iterative net + multi-scale local correlations +
    visibility head).
  - **TAP-Net** (Doersch et al., NeurIPS 2022 — not yet ingested) —
    global per-frame init via cost volume + ConvNet heatmap +
    occlusion logit. TAPIR drops the chunking of PIPs but keeps the
    TAP-Net front end.
- **Direct successor in DeepMind line:** **BootsTAPIR** (semi-
  supervised real-data fine-tune; cited 4+ times across this wiki) →
  [[tapnext]] (TRecViT + masked-token decoding; argues the depthwise-
  conv refinement and cost-volume init *re-emerge* as attention
  patterns when the inductive biases are dropped).
- **Cross-line response:** [[karaev-2024-cotracker]] adds cross-track
  attention and proxy tokens to a PIPs-like substrate; positions
  itself relative to TAPIR for joint-tracking gains.
- **Uncertainty head inherits to:** [[track-on2]] (classification-first
  coordinate head with implicit uncertainty), [[tapnext]] (256-way
  discretized coordinate classifier with expectation read-out).
- **Open-source reference:** github.com/deepmind/tapnet — used by
  most downstream evals in this wiki's TAP sources.
- **Optical-flow ancestor:** RAFT (Teed & Deng, ECCV 2020) — same
  iterative-update philosophy on a different output space.
