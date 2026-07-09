---
type: method
title: Track2Act
status: growing
tags: [manipulation, point-tracking, imitation-learning, web-video, residual-policy, diffusion]
sources:
  - "[[bharadhwaj-2024-track2act]]"
related:
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[im2flow2act]]"
  - "[[3d-flow-action]]"
  - "[[dex4d]]"
  - "[[pri4r]]"
  - "[[mu0]]"
  - "[[cotracker]]"
  - "[[diffusion-policy]]"
created: 2026-06-27
updated: 2026-07-09
---

# Track2Act

## One-line summary

Web-video-trained DiT denoiser predicts future 2D point trajectories
from `(I₀, G, P₀)`; PnP fits per-step rigid transforms; a small
residual policy (~400 robot demos) corrects per-step actions.

## Inputs / outputs

- **Train (track predictor):** 400K 4–5 sec clips from EpicKitchens +
  RT-1 + BridgeData, GT tracks from CoTracker.
- **Train (residual policy):** ~400 Spot teleop trajectories across
  10 tasks.
- **Inference inputs:** initial RGBD frame, goal image, p query
  points.
- **Inference outputs:** sequence of robot end-effector poses.

## How it works

### 1. Track prediction (DiT)

Diffusion Transformer that denoises `[P_t]_{t=1}^H` from Gaussian
noise. Conditioning: image embed of I₀ (z₀), goal embed (zg),
diffusion step (zk). p tokens (one per query point); initial P₀
injected un-noised. Variable p, random positions — at test time can
query any point set.

### 2. Rigid-transform fitting (Algorithm 1)

Filter `τ_obj` = predicted moving points. Solve

```
T_t = arg min Σ_i ‖proj(K · T_t · P_0^i) − (x_t^i, y_t^i)‖
```

with PnP + optional RANSAC for outliers. Need only first-frame depth.
Returns 3×4 transforms `[T_t]_{t=1}^H` of the object.

### 3. Open-loop end-effector trajectory

Heuristic: move e₀ to centroid of `{(x_t^i, y_t^i)}_{t=0}` with same
orientation; grasp; then `ā_t = T_t · e_1`.

### 4. Residual closed-loop policy

```
â_t = ā_t + Δa_t,   Δa_{t:t+h} = π_res(I_t, G, τ, [ā_t]_{t=1}^H)
```

Multi-step Δ prediction mitigates compounding errors (cf. action
chunking). Trained via BC on ~400 trajectories; execute only the
first action of the chunk.

## Losses

- Track predictor: standard ε-prediction L2 diffusion loss.
- Residual policy: L2 on action chunks.

## Where it's been applied

[[bharadhwaj-2024-track2act]] — Boston Dynamics Spot in physical
offices/kitchens. 4-tier generalization protocol (MG/G/CG/TG); strong
gains at CG (55%) and TG (40%) over BC / Affordance-Cond /
Video-Cond / Hand-Object-Mask baselines.

## Known limitations

- 2D-only tracks: 3D recovery via PnP from single-frame depth limits
  precision; out-of-plane rotation is shaky.
- Single rigid object per task — no articulated/deformable handling
  in action extraction.
- Embodiment-specific demos required (~400 Spot trajectories) —
  not zero-shot.
- Open-loop transforms compound errors; residual corrects but cannot
  recover from grasp failures cleanly.

## Related methods

- **[[im2flow2act]]** (concurrent CoRL 2024): object-only flow,
  fully sim-trained policy; no real-robot data at all.
- **[[3d-flow-action]]** (2025): 3D flow extension; optimization
  action policy (no learned residual).
- **[[dex4d]]** (2026): dexterous extension from the same CMU group
  (Fragkiadaki × Tulsiani); RL teacher-student instead of residual.
- **[[pri4r]]** (2026): point tracks as auxiliary supervision rather
  than policy input.
- **[[mu0]]** (2026): direct 2D-trace competitor. µ0 beats Track2Act
  on 2D top5-ADE across all horizons at 2.9× lower latency (0.29s vs
  0.85s), and reuses the same "predict tracks from a language-
  conditioned generative model" idea but replaces dense waypoint DiT
  with B-spline flow matching and adds semantic keypoint sampling.
- **[[pointworld]]** (2026): different role — action-conditioned
  dynamics WM + MPPI planner. Not a Track2Act competitor at
  benchmark level; complementary paradigm.
- **[[cotracker]]:** the 2D tracker used to mine training data.
