---
type: source
source_type: paper
title: "Towards Self-Supervised, Generalizable and Decomposable 4D Driving Scene Reconstruction"
authors:
  - Anonymous
year: 2026
venue: NeurIPS 2026 (under review)
url: "(anonymous submission)"
raw_path: papers/7307_Towards_Self_Supervised_G.pdf
status: ingested
tags: [4d-reconstruction, driving-scenes, multi-modal, lidar, point-transformer, gaussian-splatting, scene-flow, instance-decomposition, self-supervised, feed-forward]
related:
  - "[[stride]]"
  - "[[4d-reconstruction]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[karhade-2025-any4d]]"
  - "[[sucar-2026-v-dpm]]"
  - "[[luo-2026-4rc]]"
  - "[[ma-2026-fsm]]"
  - "[[anon-2026-trace-anything]]"
  - "[[anon-2026-point4d]]"
created: 2026-06-26
updated: 2026-06-26
---

# Towards Self-Supervised, Generalizable and Decomposable 4D Driving Scene Reconstruction

## TL;DR

**STRIDE** is the first feed-forward 4D driving-scene reconstruction model
that simultaneously (a) fuses **camera + LiDAR** in a unified 3D point
representation processed by a **Point Transformer v3** backbone, and (b)
learns **dynamic instance decomposition** without any human annotations.
Outputs are 3D Gaussian primitives with per-Gaussian velocity and an
instance-token assignment, all from a single forward pass. Achieves SOTA
on Waymo Open Dataset and PandaSet over STORM (camera-only) and Flux4D
(LiDAR-centric) baselines, with the largest gains on scene-flow metrics.

## Why it matters

STRIDE is the first wiki entry in the **driving-scene** flavor of the
2025-26 feed-forward 4D wave ([[4d-reconstruction]]). Two contributions
make it distinct from the existing image-only batch ([[v-dpm]],
[[any4d]], [[4rc]], [[trace-anything]], [[fsm]]):

1. **Camera + LiDAR multi-modal architecture.** Most 4D entries are
   image-only; [[any4d]] supports RGB-D/IMU/Doppler as auxiliary
   inputs but doesn't reason natively in 3D point space. STRIDE lifts
   image features into 3D using LiDAR-anchored depth and runs the
   reasoning backbone (**PTv3**) over the unified 3D point cloud.
2. **Learnable, self-supervised instance decomposition.** None of the
   existing 4D wiki entries decompose the scene into dynamic instances.
   STRIDE introduces **latent dynamic instance tokens** that emerge
   from the same motion supervision as scene flow — no 3D tracklets,
   no segmentation masks.

The paper's stated **limitation #3** ("backbone operates on all input
observations aggregated across time, preventing seamless long-horizon
reconstruction due to memory constraints") is exactly the
**streaming-4D gap** in [[overview]]'s known-gaps list. STRIDE adds a
4th anchor to the "no streaming 4D" claim alongside Any4D, V-DPM, 4RC.

## Key claims

- **Reasoning in 3D beats reasoning in 2D image space for scene flow.**
  Ablation (Table 4): replacing PTv3 with STORM's 2D cross-time ViT
  drops EPE3D from 0.180 → 0.207 and roughly **doubles flow angle
  error** (0.382 → 0.593). Confirms the paper's motivation that
  perspective distortion in 2D handicaps motion modeling. (§4.3)
- **Point Transformer v3 outperforms sparse-conv UNet** on the same
  3D-lifted features. Same ablation: SparseConvUNet → PTv3 reduces
  EPE3D from 0.385 → 0.180 (almost **2× improvement**), validating
  the claim that limited receptive fields of sparse conv cannot
  capture long-range motion correspondences. (§4.3)
- **Multi-modal input is necessary, not just helpful.** Image-only
  ablation: EPE3D 0.313 → 0.180 when adding LiDAR depth+availability
  maps (Table 5). Adding LiDAR points as an additional PTv3 input
  gives only marginal gain at higher compute — depth-map injection is
  the load-bearing modality, not raw LiDAR tokens. (§4.3)
- **Instance decomposition emerges from motion supervision alone.**
  M3 in Table 6: training the instance module with **only** the 4D
  reconstruction loss (no smoothness prior, DBSCAN-init only) already
  beats the pure-clustering baseline (+0.83 mPrec, +1.34 mRec).
  Adding the smoothness loss on top gives the best combination
  (+2.37 mPrec, +2.80 mRec). (§4.3)
- **Naive smoothness alone is degenerate.** M2 in Table 6: training
  with only the smoothness loss without the reconstruction signal
  collapses to under-decomposition (everything spatially close →
  one instance). Smoothness is a useful *regularizer* but cannot
  drive the assignments by itself. (§4.3)
- **Initialization matters.** M4 in Table 6: replacing DBSCAN-based
  token initialization with Farthest Point Sampling tanks precision
  by 27.5 points. The spatial-clustering prior in initialization is
  load-bearing, not just a warm-start convenience. (§4.3)
- **SOTA on Waymo Open Dataset and PandaSet.** Beats STORM, STORM+
  (multi-modal STORM re-impl), and Flux4D-ff on EPE3D, flow angle,
  dynamic-region PSNR/SSIM/D-RMSE across both datasets. STORM+ has
  marginally better *full-image* PSNR on WOD but is significantly
  worse in dynamic regions — STRIDE's improvements are
  *dynamic-aware*. (Tables 1-2)
- **Learnable instances beat DBSCAN at all IoU thresholds.** mPrec
  51.0 → 53.4, mRec 71.4 → 74.2 on WOD; gap widens at lower IoU
  thresholds, confirming the smoothness prior helps with coherent
  instance boundaries. (Table 3)
- **Decomposition enables scene editing.** Actor removal, injection,
  and placement demonstrated (Fig 5) — concrete downstream benefit
  of the instance representation that the optimization-baseline
  family (3DGS variants requiring per-scene tuning) cannot match
  in a feed-forward setting.

## Methods

See [[stride]] for the full method writeup. Three-stage architecture:

### Stage 1 — Multi-modal feature extraction (ViT)

For each camera image `I^(t,v)` and aligned LiDAR depth map
`D_lidar^(t,v)` + availability mask `M_lidar^(t,v)`, concatenate to
form a **5-channel input** processed by a ViT encoder. Outputs:
per-pixel dense depth `D^(t,v)`, sky mask `S^(t,v)`, and feature
maps `F^(t,v) ∈ R^{H×W×C}`.

### Stage 2 — 4D scene reasoning (PTv3 over lifted 3D features)

Back-project each foreground (non-sky) pixel feature to 3D using
predicted depth + camera calibration. Augment with embeddings of
3D xyz, time `t`, view index `v`, and pixel ray direction (Plücker).
Aggregate across all `(t, v)` into a unified 3D point cloud `F_fg`.
A **Point Transformer v3** backbone refines `F_fg → F_fg'`. MLP
decoders predict per-point Gaussian parameters
`(µ_i, σ_i, q_i, z_i, α_i, v_i)` — position, scale, quaternion,
color, opacity, **and per-Gaussian velocity**. A separate
lightweight PTv3 processes a 400m-radius Fibonacci sky-dome point
cloud for sky modeling.

### Stage 3 — Dynamic object decomposition (instance tokens)

1. **Initialize** dynamic-instance tokens by velocity-thresholding
   Gaussians (`‖v_i‖ > τ`), aligning to a common reference time, and
   DBSCAN-clustering the dynamic points → cluster centroids
   `{c_j}_{j=1}^M` → `M` learnable tokens `{t_j}` with centroid
   position + learnable embedding.
2. **Associate** Gaussians to tokens via a differentiable similarity
   matrix:

   ```
   d_feat(i, j)    = 0.5 · (cosine(H_p(f'_dyn,i), H_t(t_j)) + 1)
   d_spatial(i, j) = ‖µ_i − c_j‖₂
   ζ_j             = sigmoid(MLP(t_j))                      # existence
   s_{i,j}         = ζ_j · (d_feat(i,j) − min(1, λ_spat · d_spatial(i,j)))
   ```

   Hard assignment: `G_dyn^m = { g_i | argmax_j s_{i,j} = m }`.

3. **Back-prop through argmax** via Straight-Through Estimator on
   `softmax(s_i)`. Each Gaussian's velocity becomes
   `u_i = Σ_j w_{i,j} v_tok,j + tanh(MLP(f'_dyn,i))` — token-shared
   rigid velocity plus per-Gaussian residual. `L_recon` flows
   through `u_i`, so reconstruction error supervises which token
   each Gaussian belongs to.

### Two-stage training

- **Stage 1:** Train ViT + PTv3 backbones with
  `L_recon = L_photo + λ_v-reg · L_v-reg + λ_sky · L_sky + λ_mde · L_mde`.
  `L_photo` = L2 RGB + perceptual + L1 depth.
- **Stage 2:** Freeze stage-1 modules; train instance decomposition
  with `L_decomp = L_photo + λ_v-reg · L_v-reg + λ_smooth · L_smooth`.
  `L_smooth` (Eq 1) = cross-entropy against the modal token among each
  Gaussian's spatial-and-kinematic neighbors, suppressing
  over-decomposition.

## Results

### Waymo Open Dataset (sparse-input, 4 input frames out of 20)

| Method | Modality | Dyn PSNR↑ | Dyn D-RMSE↓ | EPE3D↓ | Flow Angle↓ |
|---|---|---|---|---|---|
| STORM | Cam | 26.38 | 5.48 | 0.276 | 0.658 |
| STORM+ (re-impl) | Cam+LiDAR | 27.34 | 2.22 | 0.200 | 0.485 |
| Flux4D-ff | LiDAR | 23.16 | 4.98 | 0.246 | 0.436 |
| **STRIDE** | Cam+LiDAR | **27.21** | **2.14** | **0.161** | **0.307** |

STRIDE's flow angle is **~33% lower** than the next-best (Flux4D-ff)
and **~37% lower** than the multi-modal STORM+. (Table 1)

### PandaSet (dense-input, 6 inputs of 16; interp + extrap)

Interp Dyn: PSNR 22.39 / D-MAE 0.91 / V-RMSE 0.154 — best on all
three metrics. Extrap Dyn: PSNR 21.45 / D-MAE 1.09 / V-RMSE 0.154 —
also best. (Table 2)

### Dynamic instance segmentation on WOD (Table 3)

| Method | mPrec↑ | Prec@0.5↑ | mRec↑ | Rec@0.5↑ |
|---|---|---|---|---|
| DBSCAN baseline | 51.03 | 42.54 | 71.43 | 59.39 |
| **STRIDE (learned)** | **53.40** | **42.84** | **74.23** | **60.13** |

### Ablations (WOD, reduced training schedule)

- **Backbone (Table 4):** Cross-time ViT (STORM-style) and
  SparseConvUNet (Flux4D-style) both underperform PTv3 — primarily
  in dynamic regions and on flow.
- **Modality (Table 5):** Image-only → +LiDAR depth = ~halves EPE3D
  (0.313 → 0.180). Adding raw LiDAR points as PTv3 input is
  marginal.
- **Instance learning (Table 6):** M5 (DBSCAN init + L_smooth +
  L_recon) is the only configuration with positive gains on both
  precision and recall.

## Limitations / open questions

(All three explicitly acknowledged by the authors in §5.)

- **Constant-velocity motion model.** Per-Gaussian linear velocity
  cannot model nonlinear or non-rigid object motion across the input
  window. Pedestrians and articulated motion are obvious failure
  cases.
- **Decomposition is motion-only.** Static-but-distinct objects
  (parked cars, traffic poles, separate buildings) cannot be
  decomposed because there is no motion signal to drive token
  assignment. The instance representation collapses static content
  into the background.
- **No streaming / long-horizon mode.** The PTv3 backbone aggregates
  all input observations across time, capping the input window by
  GPU memory. Combining STRIDE's PTv3 backbone with a streaming
  mechanism ([[streamvggt]] cached KV, [[lact]]/[[loger]] TTT-style
  fast weights, or [[fsm]]/[[lacet]] chunked elastic TTT) is an
  obvious extension.
- **Beats DBSCAN, but not by a wide margin** on instance
  segmentation (53.4 vs 51.0 mPrec). The learnable component
  improves over a strong clustering baseline, but the gap suggests
  most of the structure is being supplied by initialization.
- **No comparison against optimization-based + decomposable
  baselines** (DIAL-GS, IDSplat, UniRe). Paper positions these as
  scope-different (per-scene optimization) but a head-to-head on
  quality would strengthen the case.

## Connections

### Within the wiki

- **[[4d-reconstruction]]** — STRIDE adds the first **driving-scene
  multi-modal** entry to the feed-forward 4D wave. Other entries are
  image-only ([[v-dpm]], [[4rc]], [[trace-anything]], [[fsm]],
  [[any4d]]) or focus on long-sequence handling
  ([[anon-2026-point4d]], [[fsm]]). STRIDE's distinguishing axes:
  3D point-space reasoning (vs 2D-token), instance decomposition
  (none of the others have it), and LiDAR as primary input.
- **[[any4d]]** — closest cousin in spirit (multi-modal feed-forward
  4D with motion output). Any4D supports more modalities (RGB-D, IMU,
  Doppler) but reasons in a factored 2.5D/3D pose space; STRIDE goes
  fully into 3D point space via PTv3 and adds decomposition.
- **[[feed-forward-3d-reconstruction]]** — STRIDE extends the
  feed-forward paradigm to driving scenes, where the LiDAR sensor
  modality changes the architectural calculus (sparse 3D geometry is
  cheap and accurate; dense visual features need anchoring).
- **[[cut3r]]** — both are stateful 3D-aware models for driving-like
  domains. CUT3R is streaming (recurrent state); STRIDE is batch.
  STRIDE's PTv3 backbone is the spatial-grid analog of CUT3R's
  hidden state, similar to how StreamVGGT's cached memory tokens
  resolve the "CUT3R hidden state isn't a spatial grid" ambiguity
  in [[overview]].
- **[[ma-2026-fsm]] / [[fsm]] / [[lacet]]** — both target
  feed-forward 4D, but FSM is image-only NVS over indoor/outdoor
  scenes, while STRIDE is driving-specific with LiDAR. LaCET's
  chunked elastic TTT is the obvious candidate streaming mechanism
  to lift STRIDE's "no long-horizon" limitation.
- **Train-inference mismatch ([[train-inference-mismatch]])** —
  STRIDE trains on 2-second / 20-frame WOD clips; deployment-time
  long-horizon mode is unaddressed. Same problem class as
  Point4D / SpaTrackerV2, no fix in this paper.

### Outside the wiki (not yet ingested as primary)

- **STORM** (camera-only feed-forward 4D for driving) — primary
  baseline; cited 5+ times across this wiki via STRIDE + Flux4D.
  Worth promoting to a method page on next batch.
- **Flux4D** (LiDAR-centric feed-forward 4D) — same baseline status.
- **DrivingRecon** (camera-only DriviSC method) — referenced but not
  benchmarked head-to-head.
- **Point Transformer v3 (Wu et al. CVPR 2024)** — backbone choice.
  Worth a tool page if more 4D-driving papers adopt it.
- **DIAL-GS / IDSplat / UniRe** — three concurrent self-supervised
  instance-decomposition methods that *do* require per-scene
  optimization. STRIDE's "first feed-forward learnable decomposition"
  claim is conditioned on excluding this family.

## Citation

Anonymous (2026). *Towards Self-Supervised, Generalizable and
Decomposable 4D Driving Scene Reconstruction.* NeurIPS 2026
submission (under double-blind review).
