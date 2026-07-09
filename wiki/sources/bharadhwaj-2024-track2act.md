---
type: source
source_type: paper
title: "Track2Act: Predicting Point Tracks from Internet Videos enables Generalizable Robot Manipulation"
authors: [Bharadhwaj, Homanga; Mottaghi, Roozbeh; Gupta, Abhinav; Tulsiani, Shubham]
year: 2024
venue: ECCV 2024
url: https://arxiv.org/abs/2405.01527
raw_path: papers/2405.01527v2.pdf
status: ingested
tags: [point-tracking, manipulation, imitation-learning, web-video, foundation-paper, cross-embodiment]
sources: []
related:
  - "[[track2act]]"
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[cotracker]]"
  - "[[shubham-tulsiani]]"
  - "[[homanga-bharadhwaj]]"
  - "[[cmu-ri]]"
  - "[[meta-ai]]"
  - "[[vla]]"
created: 2026-06-27
updated: 2026-06-27
---

# Track2Act

## TL;DR

Factorize a manipulation policy into (a) an **embodiment-agnostic
interaction plan** = predicted 2D point tracks from web videos, and (b)
a **residual policy** trained on ~400 robot demos that corrects the
open-loop plan. The track prediction model is a DiT that denoises future
2D tracks conditioned on initial image, goal image, and a random set of
query points; predicted tracks → rigid transforms (PnP) → end-effector
waypoints → corrected per-step by a small closed-loop residual policy.
**This is the temporal ancestor of the
[[point-tracks-as-manipulation-interface]] line** — Im2Flow2Act,
3DFlowAction, Dex4D all cite it.

## Why it matters

This is the first paper to (i) train a generative model that predicts
**full future point tracks** of arbitrary scene points from internet
videos (human + cross-robot), and (ii) show those tracks can be
converted into robot end-effector trajectories via rigid-transform
fitting + a tiny residual policy. It demonstrates that point tracks are
expressive enough to *replace* dense video generation as a planning
interface while being cheaper to compute and easier to ground in 3D.

From the user's own group — **Bharadhwaj × Tulsiani**, the same
Tulsiani lab that produces Point4D and Flow3R. The Dex4D (2026) paper
by Kuang × Park × Fragkiadaki × Tulsiani is the direct descendant of
this line.

## Key claims

- **Track prediction beats flow + video infilling baselines** by a
  large margin on EpicKitchens / SmthSmthv2 / Bridge / RT1: ∆ score
  0.67–0.77 vs. 0.21–0.42 (Flow) / 0.17–0.30 (Video) — §5.1, Table 1.
- **Predicting full trajectories rather than next-step flow is
  expressive**: dense flow is too coarse for large non-linear state
  changes between initial and goal; video infilling produces
  implausible artifacts; track prediction abstracts out appearance.
- **A tiny residual policy (~400 demos, 10 tasks)** closes the gap
  between embodiment-agnostic open-loop tracks and robot-specific
  closed-loop execution. Compositional generalization (CG): 55%
  vs. 25% open-loop; Type generalization (TG): 40% vs. 25%.
- **Embodiment-agnostic plans transfer** — train on EpicKitchens
  (humans) + RT1 + Bridge (robots), deploy on a **Spot** mobile
  manipulator that is never in the training data.
- **3–4 orders of magnitude less in-domain robot data** than direct
  behavior cloning approaches (RT-1, BC-Z, RoboAgent).

## Methods

### Pipeline

1. **Track prediction model** (Diffusion Transformer): denoises future
   2D point trajectories `[P_t]_{t=1}^H` given initial image I₀, goal
   image G, and p random initial points P₀. Variable p, randomized
   positions — at test time, query any set of points. Trained on
   400K 4–5 sec video clips (EpicKitchens + RT-1 + BridgeData) with
   **Co-Tracker** as the ground-truth tracker.
2. **Rigid-transform fitting** (Algorithm 1): filter `τ_obj` to moving
   points, solve `T_t = arg min Σ‖proj(K·T_t·P_0^i) − (x_t^i, y_t^i)‖`
   with PnP + RANSAC. Convert to end-effector poses
   `ā_t = T_t · e_1`. Requires depth image at first frame only.
3. **Residual policy** π_res(I_t, G, τ, [ā_t]): predicts h-step residual
   action chunks `Δa_{t:t+h}` on top of the open-loop plan. Trained via
   BC on ~400 Spot teleop trajectories.

### Architecture

- Track predictor: DiT conditioned on (I₀ embed, G embed, k embed);
  one token per query point; initial P₀ injected un-noised.
- Residual policy: small image+plan transformer producing 6-DoF
  Δposes.

## Results

| Setting | BC | Affordance | Video-Cond | HOMask | Ours-OpenLoop | Ours-action | **Ours-Residual** |
|---|---|---|---|---|---|---|---|
| Mild Gen (MG) | 60% | 65% | 60% | 70% | 35% | 70% | **70%** |
| Standard Gen (G) | 20% | 30% | 25% | 40% | 25% | 45% | **60%** |
| Compositional (CG) | 0% | 10% | 0% | 25% | 30% | 30% | **55%** |
| Type Gen (TG) | 0% | 5% | 0% | 20% | 25% | 30% | **40%** |

The residual variant wins decisively at the harder generalization tiers
(CG + TG) where vanilla BC and even hand-object-mask conditioning
collapse.

## Limitations / open questions

- **Single-object short-horizon tasks** only. Long-horizon multi-object
  manipulation is left as future work.
- **Open-loop rigid-transform fitting** assumes a single rigid moving
  object (`τ_obj` filtered by motion). Articulated or deformable
  objects need a different action-extraction step.
- **Depth at first frame only** — limits 3D reasoning to initialization;
  subsequent end-effector poses come from open-loop chained transforms.
  3DFlowAction and Dex4D both move to per-step 3D point tracks for
  precisely this reason.
- **Real-world residual data on Spot** (~400 trajectories) — still
  some embodiment-specific data needed; not truly zero-shot to the
  target robot.

## Connections

- **Direct successors:** [[xu-2024-im2flow2act]] (object flow instead of
  point tracks, fully sim-conditioned policy); [[zhi-2025-3dflowaction]]
  (lifts to 3D flow + GPT-4o closed loop); [[kuang-2026-dex4d]]
  (dexterous + Anypose-to-Anypose + sim-to-real RL); [[kim-2026-pri4r]]
  (3D point tracks as *privileged supervision* for VLAs).
- **Point tracker dependency:** Uses [[cotracker]] (Karaev 2023) to
  mine ground-truth tracks from web videos. Predates [[tapir]] / TAP-Net
  as the dominant tracker choice.
- **Hand-object mask predecessor:** Bharadhwaj et al. ICRA 2024 (ref
  [4]) used hand+object masks as the interaction plan; Track2Act
  replaces them with full point trajectories.
- **DiT architecture:** [[anon-2026-stride]] and many recent
  trajectory-generation works share the same DiT pattern.
- **Concept page:** [[point-tracks-as-manipulation-interface]] is the
  central thread.
- **Author overlap:** Tulsiani group → [[anon-2026-point4d]] (user's
  paper), [[flow3r]], [[kuang-2026-dex4d]] (direct descendant).

## Citation

```
Bharadhwaj, H., Mottaghi, R., Gupta, A., & Tulsiani, S. (2024).
Track2Act: Predicting Point Tracks from Internet Videos enables
Generalizable Robot Manipulation. ECCV 2024.
arXiv:2405.01527.
```
