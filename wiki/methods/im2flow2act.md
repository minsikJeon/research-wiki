---
type: method
title: Im2Flow2Act
status: growing
tags: [manipulation, flow, object-centric, sim-to-real, cross-embodiment, diffusion-policy, video-diffusion]
sources:
  - "[[xu-2024-im2flow2act]]"
related:
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[track2act]]"
  - "[[3d-flow-action]]"
  - "[[dex4d]]"
  - "[[pri4r]]"
  - "[[mu0]]"
  - "[[pointworld]]"
  - "[[diffusion-policy]]"
  - "[[tapir]]"
created: 2026-06-27
updated: 2026-07-09
---

# Im2Flow2Act

## One-line summary

Two-network system: (i) **Im2Flow** — AnimateDiff-based flow generator
trained on cross-embodiment human/sphere videos that outputs a
*complete object-only* task flow; (ii) **Flow2Act** — diffusion policy
trained on sim-only random play that turns object flow → robot actions
via a temporal alignment module.

## Inputs / outputs

- **Flow generator inputs:** initial RGB frame, language task
  description.
- **Flow generator output:** object flow `F ∈ R^{3×T×H×W}` (u, v,
  visibility) for the whole task.
- **Policy inputs:** task flow F, current keypoint locations f_t (from
  online tracker), proprioception ρ_t.
- **Policy output:** L-step action chunk (6-DoF EE + gripper).

## How it works

### Flow generator (Im2Flow)

1. **Grounding-DINO** to detect object bbox; uniform grid-sample
   points inside.
2. Run [[tapir]] on demo videos to get ground-truth flow F.
3. **Compress flow** with frozen SD VAE encoder (decoder fine-tuned)
   → latent space `x_{1:T} ∈ R^{C×T×Hx×Wx}` (8× downsampled).
4. **AnimateDiff** = SD-UNet inflated with motion modules along time.
   LoRA on SD weights; motion modules trained from scratch.
5. Generate latent flow → decode → motion-filter (keep
   object-relevant points) → **complete task flow** at task start.

### Flow-conditioned policy (Flow2Act)

```
p(a_t | F_{0:T}, s_t, ρ_t)
```

Three blocks:
- **State encoder** φ(f_t, x₀): transformer over N current keypoints
  with 2D sinusoidal PE + linear-projected initial 3D coords +
  CLS-token summary.
- **Temporal alignment** ψ(F_{0:T}, s_t, ρ_t): predicts latent z_t of
  *remaining* task flow, supervised by encoder ξ on
  ground-truth remaining flow with L2 (detached on target side).
- **Diffusion action head**: same family as [[diffusion-policy]];
  conditions on `z_t` at inference (not raw F).

### Training data

- **Flow generator:** real human videos + real cross-embodiment robot
  videos.
- **Policy:** sim-only UR5e *play* data from predefined random
  primitive actions on rigid + articulated + deformable objects. No
  task-specific simulation.

## Key design decisions

- **Object-only flow, not grid flow.** Grid flow captures embodiment
  motion (human arm in train, robot arm at test) → OOD at deployment.
  Ablation: GridFlow drops sim SR by ~50 points vs. object flow.
- **Complete task flow once, not per-step.** Per-step flow generation
  with the embodiment present during training is OOD when the robot
  arm appears at test.
- **Temporal alignment in latent space.** Avoids predicting raw
  remaining flow; smoother training.

## Where it's been applied

[[xu-2024-im2flow2act]] — UR5e on rigid (pick-and-place, pouring),
articulated (drawer), deformable (cloth folding). Real-world 81%
average SR with **no real-world robot data**.

## Known limitations

- **2D only** — out-of-plane rotations (screwing) and z-axis precision
  fail; pouring is a frequent fail mode.
- **Sim-to-real gap on deformables** (cloth folding 90% sim → 70%
  real).
- **Open-loop task flow** — re-generation requires the alignment
  module to re-localize on a stale flow; no full flow re-generation
  mid-task.

## Related methods

- **[[track2act]]** (concurrent, ECCV 2024): full point trajectories
  + rigid transform + *residual real-robot* policy. Im2Flow2Act trades
  full trajectories for object-only flow + zero real-robot data.
- **[[3d-flow-action]]** (2025): direct 3D successor — same
  AnimateDiff backbone, but 4-channel flow (adds depth) + GPT-4o
  closed-loop verifier + optimization (not learned) action policy.
- **[[dex4d]]** (2026): same "generated video → object tracks →
  policy" decomposition but ports to dexterous + sim-to-real RL.
- **[[pri4r]]** (2026): inverted use-case — point tracks as
  supervision rather than condition.
- **[[diffusion-policy]]:** the action-head family used here.
- **[[tapir]]:** the tracker used to mine flow from human videos.
