---
type: source
source_type: paper
title: "Particle Video Revisited: Tracking Through Occlusions Using Point Trajectories"
authors:
  - Harley, Adam W.
  - Fang, Zhaoyuan
  - Fragkiadaki, Katerina
year: 2022
venue: ECCV 2022
url: https://arxiv.org/abs/2204.04153
raw_path: papers/2204.04153v2.pdf
status: ingested
tags: [point-tracking, video-understanding, optical-flow, occlusion, synthetic-training, foundation-paper]
sources: []
related:
  - "[[pips]]"
  - "[[point-tracking]]"
  - "[[joint-point-tracking]]"
  - "[[online-vs-offline-tracking]]"
  - "[[synthetic-to-real-gap]]"
  - "[[trajectory-chaining]]"
  - "[[adam-w-harley]]"
  - "[[katerina-fragkiadaki]]"
  - "[[cmu-ri]]"
created: 2026-06-26
updated: 2026-06-26
---

# Particle Video Revisited: Tracking Through Occlusions Using Point Trajectories

## TL;DR

PIPs revives Sand & Teller's 2006 "particle video" idea with a modern
recipe: a CNN feature extractor, multi-scale correlation pyramids, and
a 12-block **MLP-Mixer** that iteratively refines an 8-frame trajectory
of `(x, y)` positions + per-frame appearance features for one target
pixel. The defining design choice is **extreme temporal awareness at the
cost of spatial / cross-track context**: every target is tracked
independently. Trained on a custom **FlyingThings++** dataset with
synthetic multi-frame occlusions, the model beats RAFT (chained) and
DINO (matched) on FlyingThings++, KITTI, CroHD, and BADJA — especially
on occluded points and through occluder swap-outs.

## Why it matters

This is the **canonical predecessor** of every modern deep TAP method in
the wiki. [[karaev-2024-cotracker]], [[cotracker3]], [[tapnext]],
[[track-on2]], [[tapnext-plus-plus]], [[tapip3d]],
[[xiao-2025-spatialtracker-v2]] all explicitly cite or extend PIPs.
Three core ideas survive everywhere:

1. **Iterative refinement** of a trajectory inside a fixed temporal
   window (the lever that TAPIR replaces with depthwise conv,
   CoTracker replaces with a transformer, TAPNext replaces with an SSM
   scan, Track-On replaces with persistent memory).
2. **Multi-scale correlation pyramids** between query feature and per-
   frame features (kept in TAPIR and CoTracker; dropped in TAPNext).
3. **Visibility prediction** as a separate head trained with cross
   entropy — the ancestor of "occlusion-aware" TAP loss design.

PIPs also crystallized the **synthetic-training-with-injected-
occlusions** template that became standard (Kubric panning, occluder
paste, etc.) — see [[synthetic-to-real-gap]].

## Key claims

- **8-frame chunks beat 2-frame flow chaining** for tracking through
  multi-frame occlusions (§3.7, §5.3). RAFT-chained trajectories
  follow the occluder; PIPs picks the target back up on re-emergence.
- **Iterative MLP-Mixer refinement** over (displacement, feature,
  correlation pyramid) tokens for K=6 iterations is the best
  refinement substrate among MLP-Mixer / conv-over-time /
  attention-over-time variants the authors tested (§5.7).
- **Appearance constancy + zero-velocity initialization** — initial
  feature and position trajectories are just `tile(F_1)` and
  `tile(p_1)` across time. The network learns to fix this from the
  correlation maps. Translation invariance comes from subtracting
  `p_1` from positions before the Mixer.
- **Visibility-thresholded chaining** at test time (§3.7) extends
  beyond 8 frames: re-initialize from the latest high-visibility
  timestep, threshold starting at 0.99, decrementing until a valid
  pivot is found. **Re-use the original `F^0`** across re-inits to
  prevent identity drift onto occluders.
- **FlyingThings++ training generalizes to KITTI / CroHD / BADJA** —
  *one* model, no per-domain fine-tuning. The injected-occluder data
  augmentation is the load-bearing trick.
- **Independent per-point tracking is surprisingly competitive** with
  flow-based methods despite discarding spatial smoothness. The
  authors flag this as surprising and identify joint tracking as the
  obvious next step (the gap that [[cotracker]] later fills).

## Methods

- **Backbone:** RAFT's BasicEncoder CNN at stride 8, `C=256`. No
  temporal convs.
- **Initialization:** `F^0_t = F_1` (tile query feature), `X^0_t = p_1`
  (tile query position) for all `t ∈ [1..8]`.
- **Correlation pyramid `C^k`:** dot product between current feature
  `F^k_t` and per-frame feature map → 7×7 crop at `X^k_t` → 4 pyramid
  levels (radius 3 each). Shape `T × P²L = 8 × 49 × 4`.
- **Refinement:** concatenate `(emb(X^k − p_1), F^k, C^k)` per frame
  → 12-block **MLP-Mixer** → output `(ΔX, ΔF)` for all 8 frames at
  once. Iterate K=6 times.
- **Visibility head:** linear layer on `F^K` + sigmoid → `V^K ∈ [0,1]^8`.
- **Losses:** L1 on positions across all iterations with `γ=0.8`
  weighting (Eq. 1, RAFT-style); BCE on visibility (Eq. 2); softmax CE
  on score maps where visible (Eq. 3) to accelerate convergence.
- **Training:** 100K steps, FlyingThings++ at 368×512, batch 4 × 4
  GPUs (RTX 2080), AdamW lr=3e-4 (1-cycle). ~2 days.
- **Trajectory chaining:** see §3.7 (an early form of
  [[trajectory-chaining]] for 2D point tracking).

### FlyingThings++ (the load-bearing dataset)

- Chain FlyingThings forward flow → forward-backward consistency filter
  → instance-mask consistency filter → 8-frame trajectory library
  covering ~30% of pixels.
- **At batch time, paste a random object from another video on top**
  to synthesize multi-frame occlusions and update GT trajectories.
- Augmentations: color/brightness, random scale, drifting crops,
  Gaussian blur, H/V flips.
- 13,085 train videos / 2,542 test videos.

## Results (headline)

Average trajectory error (lower is better), 8-frame, split by visibility:

| Dataset | RAFT (chained) Vis / Occ | DINO Vis / Occ | **PIPs Vis / Occ** |
|---|---|---|---|
| FlyingThings++ | 24.32 / 46.73 | 40.68 / 77.76 | **15.54 / 36.67** |
| KITTI | 4.03 / 6.79 | 13.33 / 13.45 | 4.40 / **5.56** |
| CroHD | 7.91 / 13.04 | 22.50 / 26.06 | **5.16 / 7.56** |

BADJA keypoint propagation (PCK-T %, averaged over 7 videos):

| Method | Avg |
|---|---|
| RAFT (chained) | 45.6 |
| DINO | 50.5 |
| **PIPs** | **62.3** |

**Speed:** PIPs is faster than RAFT for small N (200 ms vs. 2000 ms at
480×1024 for 256 targets) but slower than RAFT at very large N because
memory scales with `T · N` (the MLP-Mixer consumes a T-length feature
sequence per particle).

## Limitations / open questions

- **Independent per-point inference.** No cross-track context. Authors
  call this the most important next step. [[cotracker]] (joint
  attention) directly fills this gap.
- **8-frame MLP-Mixer is non-recurrent and fixed-length.** Longer
  trajectories require **chaining** (§3.7), which can lose targets if
  occlusion exceeds the window. TAPIR's depthwise conv replaces the
  Mixer to handle arbitrary lengths.
- **No global / per-frame initialization.** Position is initialized at
  `p_1` across time, so the model relies entirely on iterative
  refinement + chaining. TAPIR adds a TAP-Net-style global
  initialization step.
- **No uncertainty estimate** — only binary visibility. TAPIR adds
  self-supervised position uncertainty.
- **Synthetic-only.** No real-data fine-tuning. The
  [[synthetic-to-real-gap]] story starts here; BootsTAP /
  [[cotracker3]] later attack this directly.
- **No 3D.** Pure 2D pixel tracking. [[tapip3d]] +
  [[xiao-2025-spatialtracker-v2]] lift to 3D much later.

## Connections

- Method page: [[pips]].
- **Direct successor (DeepMind line):** [[doersch-2023-tapir]] — fuses
  PIPs refinement with TAP-Net initialization + depthwise conv + uncertainty.
- **Joint-tracking response (Meta/Oxford line):**
  [[karaev-2024-cotracker]] adds cross-track attention; explicitly
  positions itself as filling PIPs' "independent tracks" limitation.
- **MLP-Mixer abandonment:** every TAP method after 2023 replaces the
  Mixer (TAPIR: depthwise conv; CoTracker: transformer; TAPNext:
  SSM+ViT; Track-On2: causal memory + transformer).
- **3D extensions:** [[tapip3d]] uses Harley as co-author; the world-
  coordinate N2N attention is a direct architectural descendant.
- **Trajectory chaining:** §3.7 is the 2D-pixel-query ancestor of the
  long-sequence chaining strategies analyzed in [[trajectory-chaining]]
  (where 3D-coordinate queries fix the occlusion-reprojection failure
  mode for 4D reconstruction).
- **Emergent vs hand-designed components:** [[zholus-2025-tapnext]]
  argues PIPs' cost-volume + iterative refinement *re-emerge* as
  attention patterns when the inductive biases are dropped — see
  [[q-emergent-tracking-heuristics]].
- People: [[adam-w-harley]] (first author; the modern TAP sub-field
  effectively starts here), [[katerina-fragkiadaki]] (senior author,
  user's institution faculty).
- Org: [[cmu-ri]] (lead institution — the wiki's CMU TAP origin point).
- Dataset families introduced: FlyingThings++ (training; not yet a
  dedicated dataset page — single source so far).
- Comparable era predecessors: RAFT (Teed & Deng, ECCV 2020) — flow
  ancestor; TAP-Net (Doersch et al., NeurIPS 2022) — the contemporary
  "global per-frame init" predecessor that TAPIR fuses with PIPs;
  COTR, MAST, DINO — pairwise-correspondence baselines beaten here.

## Citation

Harley, A. W., Fang, Z., & Fragkiadaki, K. (2022). *Particle Video
Revisited: Tracking Through Occlusions Using Point Trajectories.*
ECCV 2022. arXiv:2204.04153v2 [cs.CV].
https://arxiv.org/abs/2204.04153
