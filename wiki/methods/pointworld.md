---
type: method
title: PointWorld
status: growing
tags: [manipulation, world-model, 3d-point-tracking, dense-tracking, point-transformer, mpc, dinov3, foundation-model, cross-embodiment]
related:
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[mu0]]"
  - "[[track2act]]"
  - "[[im2flow2act]]"
  - "[[3d-flow-action]]"
  - "[[dex4d]]"
  - "[[pri4r]]"
  - "[[stride]]"
  - "[[vggt]]"
  - "[[cotracker3]]"
  - "[[cmp-point-track-manipulation]]"
sources:
  - "[[huang-2026-pointworld]]"
created: 2026-07-09
updated: 2026-07-09
---

# PointWorld

Large pretrained 3D dynamics model. Predicts **future 3D scene point
flows** given a **current point cloud + robot action point flows**.
Deployed via MPPI for zero-shot manipulation.

## One-line summary

PTv3 backbone + DINOv3 scene features + URDF-forward-kinematics robot
action points → chunked 10-step 3D flow prediction @ 0.1 s. MPPI
searches actions on top. Real-Franka zero-shot on rigid / deformable /
articulated / tool-use from single RGB-D.

## Inputs / outputs

**Inputs:**
- Calibrated RGB-D (one or few views).
- Robot description file (URDF).
- Sequence of robot joint configs `{q_{t+k}}_{k=0}^H` (candidate action).

**Outputs:**
- Per-point 3D displacements for scene point cloud at each of
  `H = 10` future timesteps (0.1 s each = 1 s horizon).

## How it works

Two shared-modality streams merged into one point cloud:

### State stream
1. RGB-D → back-project (mask robot pixels via URDF-FK).
2. Scene point set `s_t = {(p_{t,i}, f_i^S)}`.
3. Frozen **DINOv3** ViT-L multi-layer features projected from 2D
   via calibrated cameras attach to each point.

### Action stream
1. URDF-defined robot mesh → sample gripper surface points once
   (300–500 pts). Whole-body flows dilute signal; gripper-only wins.
2. Propagate points via forward kinematics from joint sequence
   `{q_{t+k}}` → time-varying robot 3D positions.
3. Time embedding attaches to each robot point.

### Backbone
Concatenate scene ∪ robot points → **PTv3** (Point Transformer v3,
U-Net hierarchy, 50M–1B params) → per-point features → shared MLP
head → per-step per-point 3D displacement over 10-step chunk.

### Training loss

```
L = Σ_{k,i} w_{k,i} · ρ_δ(P̂ - P) · e^{-s} + s
```
- Huber loss ρ_δ on 3D residual.
- Movement weight `w = m/Σm`, `m = σ(κ(δ - τ))` — focuses on moving
  points (only 1–5% of scene moves per step).
- Aleatoric uncertainty `s = log-variance` per point per step —
  regularizes over noisy pseudo-labels. Emerges to capture
  action-conditioned uncertainty at deformable boundaries.
- Points marked invisible by CoTracker3 excluded from supervision.

### Chunked-not-autoregressive

Single forward pass predicts all 10 steps jointly. Beats teacher-
forcing / self-feeding autoregressive (Figure 12): lower drift,
10× faster inference.

## Data + 3D annotation pipeline

**Datasets:**
- **DROID** (real single-arm Franka, in-the-wild): ~200 hours after
  filtering (60% recovery).
- **BEHAVIOR-1K** (sim bimanual / humanoid / mobile / whole-body):
  ~300 hours after filtering to contact-active nonzero-motion.

Total ~2M trajectories / 500 hours.

**3D annotation** (for real DROID):
1. Sensor depth → replaced with **FoundationStereo** depth.
2. VGGT-initialized camera extrinsics → refined via optimization
   aligning robot depth to URDF mesh. 1.8 cm / 1.9° median error.
3. **CoTracker3** 2D → lift to 3D via refined depth + extrinsics →
   visibility labels propagated.

## Deployment: MPPI-based action inference

```
Loop:
  Given RGB-D → s_0
  Given task-relevant scene points I_task with targets {g_i} (from GUI or VLM)
  Sample K action perturbations via cubic-spline correlated noise
  For each ℓ = 1..K:
    Construct robot action a^{(ℓ)}_{1:T} via URDF-FK
    Roll out: s^{(ℓ)}_{1:T} = F_θ^T(s_0, a^{(ℓ)}_{1:T})
    Cost J^{(ℓ)} = Σ_k [c_task(s_k) + c_ctrl(E_k)]
  Weighted-average update: nominal ← Σ ω_ℓ · perturbed, ω_ℓ ∝ exp(-J^{(ℓ)}/β)
  Execute first step; replan
```

Task cost `c_task = mean of ||p_{k,i} - g_i||²` over task-relevant
points. Rigid / deformable / articulated all use the same pointwise
goal formulation.

## Training

- Init: PTv3 from scratch; DINOv3 frozen; no other pretraining.
- Batch: joint DROID + BEHAVIOR-1K.
- Scale sweep: 50M / 132M / 411M / 1B — all log-linear improvement.
- Random-camera-count training generalizes best across 1/2/3-cam eval.

## Where applied

- **Prediction quality:** DROID test ℓ2 mover 0.0312 (best), 20%
  reduction from GBND baseline.
- **In-the-wild real Franka manipulation** with MPPI: rigid pushing
  (tissue box 100%, book 70%), deformable (scarf 80%, pillow 40%),
  articulated (drawer 90%, microwave 30%), tool use (duster 60%,
  broom 60%). No demos, no fine-tuning.
- **Cross-domain fine-tune with 20× fewer updates** beats scratch on
  held-out real environments.

## Known limitations

- Zero-shot cross-domain (D↔B) still poor without finetune.
- Sim-only pretrain underperforms scratch on real.
- Needs calibrated RGB-D + accurate depth (FoundationStereo used).
- MPC needs task-relevant point set + goal positions (GUI or VLM);
  not fully autonomous.
- URDF-only embodiments — no direct human-video pretraining.
- 10-step (1s) chunk horizon.
- No head-to-head against language-conditioned trace generators
  ([[mu0]] / Track2Act / 3DFlowAction).

## Related methods

- **Distinct role vs. same-family trace generators:** [[track2act]] /
  [[im2flow2act]] / [[3d-flow-action]] / [[dex4d]] / [[mu0]] answer
  *what motion should happen* given task; PointWorld answers *what
  happens* given candidate action. Task → action recovered by MPPI
  wrapper, not by the model.
- **Complementary to [[mu0]]:** µ0 predicts task-conditioned traces;
  PointWorld predicts action-conditioned dynamics. Combining them =
  µ0-trace as MPPI cost target rolled out through PointWorld
  simulator. Neither paper attempts.
- **Backbone: [[stride]]** shares PTv3. STRIDE flagged aggregate-at-
  once as a streaming cap; PointWorld inherits same trade-off.
- **Depends on:** [[wang-2025-vggt]] (camera-init for annotation),
  [[karaev-2024-cotracker3]] (2D tracks → 3D lift).
- **Not-yet-ingested baselines:** GBND (Ai et al. Sci. Robotics 2025 —
  prior SOTA), FoundationStereo (depth), MPPI (Williams 2017), VoxPoser
  / ReKep (Huang's prior VLM-based waypoint / constraint work).

## Open directions

- Language-conditioned trace generator + PointWorld dynamics for
  fully autonomous MPC (skip GUI task specification).
- Streaming PointWorld — replace PTv3 with a linear-time backbone
  (see thread D: [[loger]] / [[streamvggt]] / [[point3r]]).
- Cross-embodiment finetune with human video (no URDF) — the µ0 axis
  PointWorld cannot currently cover.
- Longer chunk horizons (currently 10 steps @ 0.1 s = 1 s).
- Direct benchmark against [[mu0]] on shared tasks.
