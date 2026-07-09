---
type: source
source_type: paper
title: "Pri4R: Learning World Dynamics for Vision-Language-Action Models with Privileged 4D Representation"
authors: [Kim, Jisoo; Cho, Jungbin; Chu, Sanghyeok; Bal, Ananya; Kim, Jinhyung; Lee, Gunhee; Lee, Sihaeng; Kim, Seung Hwan; Han, Bohyung; Lee, Hyunmin; Jeni, Laszlo A.; Kim, Seungryong]
year: 2026
venue: "arXiv preprint (Mar 2026)"
url: https://arxiv.org/abs/2603.01549
raw_path: papers/2603.01549v2.pdf
status: ingested
tags: [vla, manipulation, point-tracking, privileged-information, auxiliary-supervision, imitation-learning]
sources: []
related:
  - "[[pri4r]]"
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[vla]]"
  - "[[kuang-2026-dex4d]]"
  - "[[bharadhwaj-2024-track2act]]"
  - "[[3d-point-tracking]]"
  - "[[laszlo-jeni]]"
  - "[[cmu-ri]]"
created: 2026-06-27
updated: 2026-06-27
---

# Pri4R

## TL;DR

Adds an **auxiliary 3D point-track head** to existing VLA backbones
(OpenVLA-OFT, π₀, π₀.₅) during fine-tuning. The head predicts per-step
3D displacements `ΔP_{t:t+H}` of pre-sampled points; gradients flow
back into the VLM and reshape its shared representation to encode
**4D scene dynamics**. At inference the head is **discarded** — zero
overhead, identical architecture. Treat 3D point tracks as **privileged
information** (Vapnik) available at training-time but not test-time.
Consistent gains across LIBERO (+1–4%), RoboCasa (+3–13%), and 4 real
tasks; +13.2% RoboCasa for OpenVLA-OFT, +9.8% on LIBERO-Long is most
striking.

## Why it matters

Two design positions distinguish Pri4R from the rest of the
manipulation-from-point-tracks family:

1. **Privileged, not conditioning.** Track2Act, Im2Flow2Act,
   3DFlowAction, Dex4D, NovaFlow all *condition the policy* on
   generated point tracks at test time — they need a flow generator
   running at deployment. Pri4R uses tracks **only during training**,
   so deployment is unchanged.
2. **Compared empirically against alternative supervision targets**
   (Table III): goal point set (+0.7), 2D track (+3.9), depth (+8.3),
   3D point track (+13.2). 3D tracks are the clear winner — temporally
   dense + spatially metric + spatially sparse + same metric space as
   actions.

The Laszlo Jeni connection puts this on the CMU radar (he's CMU-RI
faculty); KAIST and LG AI Research lead.

## Key claims

- **+13.2% average SR on RoboCasa** for OpenVLA-OFT (33.1 → 46.3),
  with biggest gains on Doors (+16), Drawers (+21), Knobs (+17),
  Levers (+30.7), Buttons (+23.3) — Table II.
- **+9.8% on LIBERO-Long** for OpenVLA-OFT (85.5 → 95.3) — the
  long-horizon split benefits most.
- **3D point tracks beat all alternative supervision targets**:
  - Goal point set: +0.7
  - 2D point track: +3.9
  - Depth maps (VAE-compressed): +8.3
  - **3D point track: +13.2** (Table III)
- **Track both scene and robot points**, not just one (+13.2 vs.
  +10.7 robot-only vs. +2.1 scene-only).
- **Don't feed P_t as input** — feeding the current point set to the
  VLM backbone hurts Pick-and-Place performance (likely because it
  perturbs the VLM's pretrained token-embedding space); auxiliary-only
  is best (Table IV).
- **Faster convergence** — Pri4R reaches the baseline's peak success
  rate **2.7× faster** in wall-clock training (Fig. 3).

## Methods

### Architecture (Fig. 2)

- **Point Track Head** = two small MLPs:
  - Point MLP: embeds current point set `P_t ∈ R^{Np×3}` → per-point
    features `e_t ∈ R^{Np×d}`.
  - Fusion MLP: takes `z_t ⊕ e_t` (broadcast across H steps and Np
    points) → predicts `ΔP_{t:t+H} ∈ R^{H×Np×3}`.
- **z_t = backbone embedding** sequence over horizon H.
  - **OpenVLA-OFT:** z_t = final-layer hidden states of the
    action-query tokens (the ones that map to action chunks).
  - **π series:** introduce a transformer **embedding module** with
    learnable queries that cross-attends to the backbone's
    final-layer image+language tokens → z_t. (Table V ablates: this
    custom embedding module beats point-expert / learnable-query-only
    alternatives.)

### Training (§IV-D)

- Auxiliary L1 loss on 3D point-track displacements:
  `L = L_act + ω_pt · ‖ΔP̂_{t:t+H} − ΔP_{t:t+H}‖_1`.
- `ω_pt = 1`, `N_p = 1024`.
- **Sim 3D tracks** = simulator mesh + face indices + barycentrics
  (deterministic ground truth).
- **Real 3D tracks** = pseudo-labels from an off-the-shelf 3D point
  tracker (ref [61] — likely SpatialTracker/SpaTrack-style) +
  segmentation-biased foreground sampling.
- Discard head after training.

### Why 3D point tracks? (§IV-B)

Versus goal-only / 2D / depth, 3D point tracks are uniquely:
- **Temporally dense** (full horizon, matching action chunk).
- **Geometric / metric** (3D Euclidean, same space as actions).
- **Spatially sparse** (efficient — N_p = 1024 vs. dense depth/video).
- **Identity-registered** across time (depth maps lack this).

## Results

### LIBERO (Table I; 4 task suites × 500 trials)

| Method | Avg SR |
|---|---|
| Diffusion Policy | 72.4 |
| Octo | 75.1 |
| DiT Policy | 82.4 |
| OpenVLA | 76.5 |
| π₀ | 87.4 |
| **π₀ + Pri4R** | **90.6** (+3.2) |
| π₀.₅ | 92.6 |
| **π₀.₅ + Pri4R** | **94.0** (+1.4) |
| OpenVLA-OFT | 92.7 |
| **OpenVLA-OFT + Pri4R** | **96.3** (+3.6) |

### RoboCasa (Table II; 24 atomic tasks × 50 trials)

| Method | Avg SR |
|---|---|
| π₀ | 38.8 |
| **π₀ + Pri4R** | **42.2** (+3.4) |
| π₀.₅ | 52.9 |
| **π₀.₅ + Pri4R** | **57.0** (+4.1) |
| OpenVLA-OFT | 33.1 |
| **OpenVLA-OFT + Pri4R** | **46.3** (+13.2) |

### Real-world (OMY-F3M arm; Table VII)

Gains across 4 tasks (Height, Spatial, Depth, Tracking), e.g. Height
unseen 83.3 → 96.7 and Tracking OOD 50.0 → 66.7 with OpenVLA-OFT.

## Limitations / open questions

- **Needs ground-truth or pseudo-labeled 3D point tracks at training
  time** — extra annotation pipeline, even if free at inference.
- **Backbone-specific embedding-module design** (π series needs the
  custom cross-attention module; non-trivial to port to a new VLA).
- **Pseudo-label quality bottleneck** — real-world point tracks come
  from an off-the-shelf 3D tracker; failure modes inherit.
- **No ablation on point density** — does N_p = 1024 vs. 256 vs. 4096
  matter?

## Connections

- **Closest sibling:** [[kuang-2026-dex4d]] — uses 3D point tracks as
  **policy conditioning**, not privileged supervision. Pri4R argues the
  privileged form is sufficient and cheaper at deployment.
- **Direct predecessor of the supervision pattern:** Visual Trace
  Prompting (Wen 2025 ref [68], 2D); Pri4R's contribution is moving
  to 3D and to **auxiliary loss** rather than input prompt.
- **VLA backbones used:** [[vla]] page covers π₀, π₀.₅, OpenVLA-OFT.
- **Point tracking dependency:** ref [61] is a recent 3D point
  tracker, likely SpatialTrackerV2 or similar — would be the same
  family as [[xiao-2025-spatialtracker-v2]] / [[zhang-2025-tapip3d]].
- **CMU connection:** [[laszlo-jeni]] is CMU-RI faculty (co-senior).
- **Privileged-information framing:** Cites Vapnik's LUPI (Learning
  Using Privileged Information). Same conceptual move as
  teacher-student RL (Dex4D's teacher policy uses privileged
  observations).
- **Concept page:** [[point-tracks-as-manipulation-interface]] —
  Pri4R is the "training-time-only" branch of that family.

## Citation

```
Kim, J., Cho, J., Chu, S., Bal, A., Kim, J., Lee, G., Lee, S., Kim, S.H.,
Han, B., Lee, H., Jeni, L. A., & Kim, S. (2026). Pri4R: Learning
World Dynamics for Vision-Language-Action Models with Privileged 4D
Representation. arXiv preprint arXiv:2603.01549.
```
