---
type: method
title: Dex4D
status: growing
tags: [manipulation, dexterous, sim-to-real, reinforcement-learning, point-tracking, world-model, paired-point-encoding]
sources:
  - "[[kuang-2026-dex4d]]"
related:
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[track2act]]"
  - "[[3d-flow-action]]"
  - "[[im2flow2act]]"
  - "[[pri4r]]"
  - "[[mu0]]"
  - "[[pointworld]]"
  - "[[cotracker3]]"
  - "[[shubham-tulsiani]]"
  - "[[katerina-fragkiadaki]]"
created: 2026-06-27
updated: 2026-07-09
---

# Dex4D

## One-line summary

Task-agnostic **Anypose-to-Anypose (AP2AP)** sim-to-real RL policy
for 22-DoF xArm6 + LEAP hand, conditioned on object-centric 3D point
tracks via **Paired Point Encoding**. High-level planner: video gen
(Wan2.6) → SAM2 + CoTracker3 → relative-depth-rescaled 3D point
tracks → policy condition. Closed-loop online tracking at deployment.

## Inputs / outputs

- **Training:** 3,200 UniDexGrasp objects in Isaac Gym with random
  pose pairs; PPO + 3-stage curriculum.
- **Inference inputs:** language instruction + initial RGBD.
- **Inference outputs:** 22-dim joint action (6-DoF arm delta + 16-DoF
  LEAP absolute) at each control step.

## How it works

### Anypose-to-Anypose (§III-A)

Goal-conditioned MDP: each episode starts at a random object pose;
target poses re-sampled randomly on success. No language, no
task-specific reward. PPO + 3-stage curriculum (single object → many
objects → tougher init + slower arm).

### Paired Point Encoding (§III-B)

```
q_t^i = [p_t^i ; p̄_t^i] ∈ R^6     (current ⊕ target per point)
```

N pairs → PointNet (shared MLPs + mean-max mixed pooling) → paired
point features. Preserves both correspondence (paired by index) and
permutation-invariance (PointNet pooling). Ablation shows this is
critical: 60.0% SR vs. 5.7% (MLP) vs. 20.3% (decoupled PointNet).

### Teacher–student distillation (§III-C)

- **Teacher** (privileged RL): full points on whole object + privileged
  state (torques, fingertip distances) → PPO actor + critic.
- **Student** (deployment): masked partial points (random
  plane-height masking for occlusion robustness) + propriop + last
  action → MLP/PointNet tokenizers → transformer encoder → action
  token + next-state tokens.
- Loss: `L = L_bc + L_wm` (DAgger BC L1 + L1 next-state).

### Reward shaping

```
r = r_goal + r_f,o + r_h,o + r_bonus + r_curl + r_table + r_action
```

Point-distance-based rather than 6D-pose-based for smoother landscape.

### Deployment pipeline (§IV)

1. **Wan2.6** generates task video from language + initial RGBD.
2. **SAM2** segments object in frame 1.
3. **CoTracker3** → 2D object tracks across generated frames.
4. **Video Depth Anything** → relative depth, **median-rescaled** to
   match initial D₀ → metric depth (Dex4D's smoothness fix over
   3DFlowAction).
5. Back-project to 3D → target point tracks.
6. **Closed-loop online tracking** with CoTracker3 → current points →
   Paired Point Encoding → policy action.

## Where it's been applied

[[kuang-2026-dex4d]] — xArm6 + LEAP hand on 4 real-world tasks
(LiftToy, Broccoli2Plate, Meat2Bowl, Pour). Zero real demos. 19/40
SR vs. 10/40 NovaFlow-CL baseline.

## Known limitations

- Single-object manipulation only.
- No human-grasp priors / HOI data — LEAP hand morphology too far
  from human.
- No tactile sensing — vision + point tracks only.
- Online tracking failures under fast motion / occlusion = main
  real-world failure mode.

## Related methods

- **[[track2act]]:** Same CMU Tulsiani group, same "predict tracks →
  use them as plan" decomposition, but parallel-jaw + residual
  policy. Dex4D is the dexterous + RL descendant.
- **[[3d-flow-action]]:** Closest contemporary — also uses generated
  video → 3D point tracks; Dex4D uses learned dexterous RL policy
  vs. 3DFlowAction's optimization on parallel-jaw.
- **[[im2flow2act]]:** Concept ancestor — same flow-as-interface but
  2D + sim-trained learned policy + parallel-jaw.
- **NovaFlow** (Li 2025 — primary Dex4D baseline, not yet ingested):
  3D actionable flow + Kabsch + IK.
- **[[pri4r]]:** Inverse philosophy — same 3D point tracks but as
  *privileged training-time supervision* rather than as conditioning.
- **[[cotracker3]]:** Online tracker used at deployment.
