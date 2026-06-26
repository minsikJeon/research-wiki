---
type: source
source_type: paper
title: "TAPIR: Tracking Any Point with per-frame Initialization and temporal Refinement"
authors:
  - Doersch, Carl
  - Yang, Yi
  - Vecerik, Mel
  - Gokay, Dilara
  - Gupta, Ankush
  - Aytar, Yusuf
  - Carreira, Joao
  - Zisserman, Andrew
year: 2023
venue: ICCV 2023
url: https://arxiv.org/abs/2306.08637
raw_path: papers/2306.08637v2.pdf
status: ingested
tags: [point-tracking, video-understanding, occlusion, uncertainty-estimation, two-stage, foundation-paper]
sources: []
related:
  - "[[tapir]]"
  - "[[point-tracking]]"
  - "[[online-vs-offline-tracking]]"
  - "[[synthetic-to-real-gap]]"
  - "[[harley-2022-pips]]"
  - "[[pips]]"
  - "[[carl-doersch]]"
  - "[[ignacio-rocco]]"
  - "[[google-deepmind]]"
  - "[[oxford-vgg]]"
  - "[[tap-vid-dataset]]"
  - "[[kubric-dataset]]"
created: 2026-06-26
updated: 2026-06-26
---

# TAPIR: Tracking Any Point with per-frame Initialization and temporal Refinement

## TL;DR

TAPIR is the explicit fusion of the two then-leading deep TAP
architectures: **TAP-Net's global per-frame initialization** (a cost
volume + ConvNet that picks an occlusion-robust match independently in
every frame) plus **PIPs' iterative temporal refinement** (multi-scale
local score maps + iterative position updates). Two key engineering
moves make it the canonical TAP recipe of 2023: (1) replace PIPs'
fixed-length MLP-Mixer with a 12-block **depthwise convolution** network
so the refinement runs on any sequence length, and (2) add a
**self-supervised position-uncertainty** head that lets the model
suppress confident-but-wrong predictions. On TAP-Vid, TAPIR improves AJ
by ~20% absolute over PIPs/TAP-Net on DAVIS and ~10% on Kinetics,
running at ~150 FPS (vs PIPs' ~3 FPS due to chunked-MLP serial
inference). Open-sourced as the reference TAP implementation.

## Why it matters

This is the **DeepMind TAP-line's canonical step 2**, between TAP-Vid /
TAP-Net (2022) and BootsTAP (2024) / [[tapnext]] (2025) /
[[tapnext-plus-plus]] (2026). It is the source most other TAP papers
benchmark against and the **predecessor explicitly named in
[[tapnext]]'s thesis** ("none of these tracking-specific components are
necessary"). Three TAPIR ideas show up everywhere afterwards:

1. **Two-stage = global init + temporal refine** as the dominant TAP
   template (BootsTAPIR, BootsTAPNext, LocoTrack all keep this).
2. **Depthwise conv over time as the any-length, parallel-friendly
   refinement substrate** (used as-is by BootsTAPIR; replaced but
   functionally equivalent in [[cotracker]]'s sliding-window
   transformer and [[tapnext]]'s SSM-over-time).
3. **Self-supervised position uncertainty** (extended by [[track-on2]]
   into classification-first coordinate heads and by [[tapnext]] into
   discretized 256-way coordinate classifiers).

For [[synthetic-to-real-gap]] history, TAPIR also introduces the
**Panning MOVi-E** training-data fix — modifying the Kubric "look-at"
point so the camera pans, which the authors found crucial for handling
real-world camera motion. The Panning-MOVi-E recipe is inherited by
every DeepMind TAP follow-up.

## Key claims

- **Combining TAP-Net + PIPs is the load-bearing architectural idea.**
  The "simple combination" (TAP-Net init + PIPs chunked Mixer) beats
  either alone on DAVIS (50.0 AJ vs PIPs 43.3, TAP-Net 41.1).
  TAPIR's additional contributions push DAVIS AJ further to 61.3.
  (Table 5.)
- **Depthwise conv replaces PIPs' MLP-Mixer at parity of params** but
  runs on sequences of any length — the *single largest* architectural
  ablation for DAVIS (53.8 → 61.3 AJ, Table 6, "no depthwise conv").
- **Self-supervised position uncertainty** (predict whether the
  position estimate is within δ of GT; supervise from model output)
  adds +2.7 AJ on DAVIS for free (Table 6).
- **Fully-convolutional in time → parallel inference.** Runtime drops
  from PIPs' ~1 month per Kinetics-Vid GPU run (sequential chunk
  chaining) to ~150 FPS (parallel across frames). On
  horsejump-high@256: 0.3 s vs PIPs 34.5 s for 50 queries.
- **Higher-resolution multi-scale features matter.** Training at
  stride-4 (not just at test time as PIPs did) closes the
  train/test gap (+7.3 DAVIS AJ, Table 6).
- **Panning MOVi-E** training data — modifying Kubric's static "look-
  at" point to follow a random linear trajectory — is needed for the
  model to handle panning real-world cameras. Improves DAVIS but not
  RGB-Stacking (static cameras).
- **Uncertainty enables high-precision downstream use.** Combined
  visibility-and-confident gate: `(1 − u_t) · (1 − o_t) > 0.5`. Useful
  for SfM-style consumers that prefer rejecting low-quality matches.
- **TAPIR refinement converges in 4–5 iterations** (vs PIPs' 6), since
  TAP-Net's global init is already strong. More iterations actively
  hurt RGB-Stacking — possible over-smoothing failure (Table 7).

## Methods

### Stage 1 — Track Initialization (TAP-Net-style)

- **Feature backbone:** TSM-ResNet-18 (paper version) or PreResNet-18
  with instance norm (open-source version), output stride 8, channels
  `C`. Bilinear-sample the query feature `F_q`.
- **Cost volume:** dot product between `F_q` and every per-frame feature
  → `(T, H/8, W/8, 1)`.
- **Per-frame ConvNet** on each frame's cost volume slice → heatmap
  `(H/8, W/8, 1)` + scalar occlusion logit.
- **Spatial soft-argmax with thresholding:** softmax over space, zero
  out cells far from the argmax, take weighted mean of positions.
  Differentiable, but the threshold suppresses averaging across
  multiple matches.
- **Position uncertainty head:** a second logit predicts whether the
  predicted position is far from GT (`> δ`). Target is computed from
  the model's own output (self-supervised).

### Stage 2 — Iterative Refinement (PIPs-style, but depthwise)

- **Local multi-scale score maps:** for each frame, dot-product `F_q`
  (or its refined version) against the per-frame feature pyramid (`L`
  levels by stride-doubling pool), extract 7×7 patches centered at
  the current track. Shape `(T, 7·7·L)`.
- **Refinement network:** 12 blocks, each a `1×1` conv + a depthwise
  conv. Identical-parameter analog of PIPs' MLP-Mixer (cross-channel
  → 1×1, within-channel → depthwise) but **convolutional along time**,
  so it runs at any T. Input shape `T × (C + K + 4)` where
  `K = 7·7·L`. Output `(Δp, Δo, Δu, ΔF_q)` per iteration.
- **Loss (per refinement step + initialization):**
  ```
  L = Huber(p̂, p) · (1 − ô)
      + BCE(ô, o)
      + BCE(û, u) · (1 − ô)
  with u_t* = 1 if ‖p̂ − p‖ > δ else 0   (self-supervised target)
  ```
- **Iterate 4 refinement steps** (paper) / 4 (open-source).

### High-resolution inference

Resize the video into K log-spaced resolutions (256² → original).
Initialize at 256², iterate refinement at every level, average outputs.
Avoids false positives from running the global init at huge feature
grids.

### Panning MOVi-E

Modify Kubric MOVi-E so the camera "look-at" point follows a random
linear trajectory of length ≤ 1.5 units off-ground, within radius 4 of
the workspace center, passing within 1 unit of center. Maintains scene
visibility while inducing panning.

## Results

**TAP-Vid (strided, 256², AJ / δ_avg / OA):**

| Method | Kinetics | Kubric | DAVIS | RGB-Stacking |
|---|---|---|---|---|
| COTR | 19.0 / 38.8 / 57.4 | 40.1 / 60.7 / 78.6 | 35.4 / 51.3 / 80.2 | 6.8 / 13.5 / 79.1 |
| RAFT | 34.5 / 52.5 / 79.7 | 41.2 / 58.2 / 86.4 | 30.0 / 46.3 / 79.6 | 44.0 / 58.6 / 90.4 |
| **PIPs** | 35.3 / 54.8 / 77.4 | 59.1 / 74.8 / 88.6 | 42.0 / 59.4 / 82.1 | 37.3 / 51.0 / 91.6 |
| **TAP-Net** | 46.6 / 60.9 / 85.0 | 65.4 / 77.7 / 93.0 | 38.4 / 53.1 / 82.3 | 59.9 / 72.8 / 90.4 |
| **TAPIR (MOVi-E)** | 57.1 / 70.0 / 87.6 | **84.3 / 91.8 / 95.8** | 59.8 / 72.3 / 87.6 | **66.2 / 77.4 / 93.3** |
| **TAPIR (Panning MOVi-E)** | **57.2 / 70.1 / 87.8** | 84.7 / 92.1 / 95.8 | **61.3 / 73.6 / 88.8** | 62.7 / 74.6 / 91.6 |

Headline gains: +19.3 AJ DAVIS vs PIPs; +10.6 AJ Kinetics vs TAP-Net.

**TAPIR high-resolution** (DAVIS 1080p, Kinetics 720p): DAVIS AJ
61.3 → 65.7; Kinetics 57.2 → 60.0 — most of the gain from higher
spatial detail in refinement.

**Ablations (Table 6, DAVIS AJ):**

| Ablation | DAVIS AJ |
|---|---|
| Full TAPIR | **61.3** |
| − Depthwise Conv (use MLP-Mixer + chunking) | 53.8 |
| − Higher Res Feature (no stride-4 input) | 54.0 |
| − TAP-Net Initialization (constant init) | 59.3 |
| − Uncertainty Estimate | 58.6 |
| − Iterative Refinement (init only) | 41.6 |

The init-vs-refinement contribution split: refinement is much more
important than the global initialization, but each contributes.

## Limitations / open questions

- **Synthetic-only training** (Kubric MOVi-E variants). The
  [[synthetic-to-real-gap]] is directly addressed in BootsTAPIR
  (semi-supervised real-data fine-tune; cited but not ingested).
- **Independent per-point tracks.** Inherits PIPs' limitation; no
  cross-track context. [[cotracker]] later fixes this.
- **Refinement is the bottleneck on RGB-Stacking** (textureless
  objects); more iterations actively *hurt* there — open question on
  why and how to fix (§5.3 discussion).
- **No causal/online operation.** Refinement uses bidirectional context
  across the whole video. [[tapnext]] / [[track-on2]] later achieve
  causal SOTA.
- **Occlusion and uncertainty are still independently predicted** from
  the cost volume — combining them via `(1−u_t)·(1−o_t) > 0.5` is a
  heuristic, not a learned consistent head.
- **High-resolution inference is engineering-heavy** (TPU partitioning,
  image pyramid, re-init occlusion at each level). [[xiao-2025-spatialtracker-v2]]
  and CoTracker3 later run at high res more naturally.
- **No 3D.** Pure 2D pixel tracking; the lifting to world coordinates
  happens later in [[tapip3d]] / [[xiao-2025-spatialtracker-v2]].

## Connections

- Method page: [[tapir]].
- **Direct predecessors:** [[harley-2022-pips]] (refinement substrate
  + correlation pyramid) and TAP-Net (Doersch et al., NeurIPS 2022;
  global init). Figure 3 of the paper is the canonical visualization
  of the PIPs → "simple combination" → TAPIR axis.
- **Direct successor in DeepMind line:** BootsTAPIR (Doersch et al.,
  2024; semi-supervised real-data fine-tune — cited 4+ times in this
  wiki, no primary page yet) → [[tapnext]] (recasts as masked-token
  decoding, drops most engineered components).
- **Cross-line predecessor of:** [[karaev-2024-cotracker]] (Karaev
  et al. position CoTracker explicitly relative to PIPs and TAPIR;
  CoTracker takes the iterative cost-volume idea and adds cross-track
  attention).
- **Emergent-vs-engineered debate origin:** [[tapnext]] argues TAPIR's
  cost-volume + depthwise-conv-over-time + uncertainty re-emerge as
  attention patterns when the inductive biases are dropped — see
  [[q-emergent-tracking-heuristics]].
- **Panning MOVi-E** carried forward to BootsTAP, BootsTAPNext,
  TAPNext++ Kubric-1024 — same dataset lineage.
- **Uncertainty estimation** evolves into [[track-on2]]'s
  classification-first head and [[tapnext]]'s 256-way discretized
  coordinate classifier.
- **Trajectory diffusion proof-of-concept** (§7) — diffusion model
  generates plausible trajectories from a still image. Distant
  ancestor of motion-conditioned video generation work but not
  followed up in this wiki yet.
- People: [[carl-doersch]] (first author; the through-line of the
  DeepMind TAP-line), [[ignacio-rocco]] (co-author; recurs in
  [[karaev-2024-cotracker]], [[zholus-2025-tapnext]], D4RT), Andrew
  Zisserman (senior, [[oxford-vgg]]).
- Orgs: [[google-deepmind]] + [[oxford-vgg]] (joint affiliation
  pattern that recurs in TAPNext, D4RT).
- Dataset: [[tap-vid-dataset]] (eval), [[kubric-dataset]] +
  Panning MOVi-E variant (training).

## Citation

Doersch, C., Yang, Y., Vecerik, M., Gokay, D., Gupta, A., Aytar, Y.,
Carreira, J., & Zisserman, A. (2023). *TAPIR: Tracking Any Point with
per-frame Initialization and temporal Refinement.* ICCV 2023.
arXiv:2306.08637v2 [cs.CV]. https://arxiv.org/abs/2306.08637
