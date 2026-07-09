---
type: source
source_type: paper
title: "Visual Imitation Enables Contextual Humanoid Control (VideoMimic)"
authors: [Allshire, Arthur; Choi, Hongsuk; Zhang, Junyi; McAllister, David; Zhang, Anthony; Kim, Chung Min; Darrell, Trevor; Abbeel, Pieter; Malik, Jitendra; Kanazawa, Angjoo]
year: 2025
venue: "CoRL 2025 (Best Student Paper Award)"
url: https://arxiv.org/abs/2505.03729
raw_path: papers/2505.03729v1.pdf
status: ingested
tags: [humanoid, whole-body-control, real-to-sim-to-real, imitation-learning, reinforcement-learning, smpl, 4d-reconstruction, deepmimic, unitree-g1, monocular-video]
sources:
  - "[[zhang-2026-loger]]"
related:
  - "[[videomimic]]"
  - "[[uc-berkeley]]"
  - "[[junyi-zhang]]"
  - "[[4d-reconstruction]]"
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[cotracker3]]"
created: 2026-07-09
updated: 2026-07-09
---

# VideoMimic

## TL;DR

**Real-to-sim-to-real pipeline** turning monocular smartphone videos
into contextual humanoid whole-body skills. Reconstructs 4D
human-scene geometry from RGB video (SMPL + MegaSAM/MonST3R + joint
optimization for metric scale), retargets to Unitree G1, trains
DeepMimic-style RL policy over reconstructed terrains, distills via
DAgger to a **single policy conditioned on proprioception + 11×11
heightmap + root direction**. Deployed onboard 50 Hz on real 23-DoF
Unitree G1 — stair ascents/descents, chair sit-stand, rough terrain,
grass, kerbs — all one policy, no per-task tuning. **CoRL 2025 Best
Student Paper**. 123 smartphone videos as full training set. UC
Berkeley (Allshire × Choi × Zhang × McAllister equal, Darrell +
Abbeel + Malik + Kanazawa senior).

## Why it matters

Opens a new corner of the wiki: **humanoid whole-body control learned
directly from monocular video** (no MoCap, no pre-scanned scenes, no
per-skill reward engineering). Sharp departures from prior wiki
threads:

1. **Whole-body humanoid, not tabletop manipulation.** All prior (E)
   sources (Track2Act / µ0 / PointWorld / …) target gripper or hand
   manipulation. VideoMimic is 23-DoF whole-body locomotion +
   contextual sitting / climbing on a real bipedal humanoid.
2. **Joint 4D human-scene reconstruction** as a first-class pretraining
   substrate. Prior 4D-recon wiki sources ([[trace-anything]] /
   [[point4d]] / [[v-dpm]]) reconstruct scenes; VideoMimic
   additionally recovers metric-scale human motion in the same world
   frame — physically meaningful contact-consistent trajectories,
   simulator-ready meshes.
3. **Real-to-sim-to-real** as a paradigm — different from the (E) line
   which trains from real / sim demos directly. VideoMimic converts
   in-the-wild video → simulator-ready reference → RL policy → real
   robot. Bridge is the joint 4D reconstruction, not action labels.
4. **Single unified policy over all skills.** Not skill-selection
   between separate policies; one PPO-trained controller selects
   behavior from environment context (heightmap + root direction).
   Context replaces task labels.
5. **New author cluster.** UC Berkeley (Kanazawa + Malik + Abbeel +
   Darrell + Allshire + Choi + Junyi Zhang). Junyi Zhang appearing
   here as co-first bumps him to **2 wiki sources** (LoGeR + VM);
   confirms his Darrell-lab affiliation across long-context 3D-recon
   and humanoid-video threads.
6. **CoRL 2025 Best Student Paper** — strong external signal.

## Key claims

### Reconstruction — SLOPER4D subset (Table 2)

| Method | WA-MPJPE ↓ (mm) | W-MPJPE ↓ (mm) | Chamfer ↓ |
|---|---|---|---|
| WHAM | 189.29 | 1148.49 | — (no scene) |
| TRAM | 149.48 | 954.90 | 10.66 |
| **VideoMimic** | **112.13** | **696.62** | **0.75** |

- **~25% W-MPJPE reduction** vs TRAM; **14× Chamfer reduction** —
  scene-aware joint optimization dominates.
- **~20 ms per 300-frame sequence** on A100 after JAX +
  Levenberg-Marquardt compilation. Real-time-viable data engine.

### Real-world Unitree G1 (Figure 5)

Single distilled policy handles:
- Indoor + outdoor stair ascents and descents.
- Chair / bench sit-down and stand-up.
- Kerb / block stepping.
- Steep earthen slopes, rough vegetation.

All from proprioception + LiDAR heightmap + joystick root direction.
**No per-skill reward engineering, no per-skill policy.**

Robustness anecdote: unexpected foot slides while descending stairs →
policy momentarily hops on single leg to recover balance. Emergent
recovery.

### MoCap Pre-Training (MPT) ablation (Figure 6)

Removing MPT → policy fails to balance during training; success
rate stays near 0 across 30k iterations. MPT is load-bearing.

### Data scale

**Only 123 monocular smartphone videos** as the full training corpus
for scene-aware skills — combined with LAFAN MoCap for pretraining.
Sub-1B-parameter policy scale; comparable to prior humanoid work,
notably less than DROID+B1K corpora used by [[huang-2026-pointworld]].

## Methods

### 3.1 Real-to-sim: monocular video → simulator-ready data

Pipeline (Figure 2):

```
Monocular RGB video
    ↓
Grounded SAM2 → person detection + tracking
    ↓
VIMO → per-frame SMPL (θ, β, J3D)
ViTPose → 2D keypoints J_2D
BSTRO → foot contact
MegaSAM (or MonST3R) → per-frame depth + camera pose [R|t] + intrinsics K
    ↓
Coarse global trajectory (SLAHMR-style focal + limb-length ratio)
    ↓
JAX Levenberg-Marquardt joint optimization:
    minimize w3D·L3D + w2D·L2D + L_smooth
    over global translations γ, orientations ϕ, local poses θ, scene scale α
    ↓
GeoCalib → gravity alignment
NKSR → point cloud → lightweight mesh
Kinematic retargeting to Unitree G1 (joint-limit, contact, collision constraints)
    ↓
Simulator-ready motion + mesh pair
```

Metric scale recovered via **SMPL height prior** — human body size is
a known metric anchor; MegaSAM's scale-ambiguous point cloud rescaled
to match.

Optional **scale-adaptation pass**: refit SMPL β to G1 dimensions
before scene optimization → shrinks scene to robot-size proportions.
Improves policy trainability but not used at real-world deployment
(operates directly at metric-scale scene).

### 4. Policy training in simulation (Figure 3)

Four-stage curriculum in IsaacGym + PPO:

**Stage 1 — MoCap Pre-Training (MPT).** LAFAN motion capture retargeted
to G1. Policy learns to track target joint angles + root roll/pitch +
root direction on flat ground. Pretraining alone gives a deployable
policy (Fig 9) but no scene context.

**Stage 2 — Scene-Conditioned Tracking.** Init from MPT checkpoint.
Add heightmap conditioning via residual projection into latent (init
weight 0). DeepMimic-style tracking across reconstructed video
terrains. Batched tracking with Reference State Init + motion load
balancing (upweight low-SR motions).

**Stage 3 — DAgger Distillation.** Distill to policy that observes
only proprioception + heightmap + root direction (drops target joint
angles + root roll/pitch). Root direction can come from joystick or
high-level planner.

**Stage 4 — Under-conditioned RL Fine-tune.** Another PPO round on
the distilled observation set. Removes suboptimality from the
distillation gap. Also permits adding lower-quality reference motions
to the corpus without collapsing.

### Observations

Actor input:
- Proprio history (length 5): joint pos `q`, joint vel `q̇`, angular
  vel `ω`, projected gravity `g`, previous action `a_{t-1}`.
- Target root roll/pitch, root direction (relative x-y + yaw in local
  frame).
- Optional: target joint angles (Stage 1/2), 11×11 heightmap (Stage
  2+).

Critic: same + privileged observations (Table 3).

### Rewards

Data-driven tracking only — link + joint position tracking, joint
velocity tracking, foot-contact matching. Action-rate penalty to
suppress simulator physics exploitation. No handcrafted skill-specific
terms.

### 5.2 Real-world deployment

- **Robot:** Unitree G1, 23-DoF, onboard 50 Hz.
- **Sensing:** Fast-lio2 LiDAR odometry + probabilistic terrain
  mapping → 11×11 heightmap aligned to torso frame.
- **Input:** joystick root direction from human operator.
- **Two critical tuning ingredients** (from iterative sim-to-real):
  (1) relax episode-termination tolerances vs reference; (2) inject
  realistic physics perturbations during training.
- **Kp = 75** (low joint gain) to avoid violent contact when hitting
  chairs / stairs.

## Results

Beyond Tables 2 and Figure 6 above:

- **Real-world qualitative (Fig 5):** sit → stand from chair, upstairs,
  downstairs, kerb + rough terrain. Single policy. Recoveries under
  disturbance.
- **First real-world deployment** of a context-aware humanoid policy
  learned from monocular human videos, jointly demonstrating
  perceptive locomotion + environment-prompted whole-body skills
  (sitting, standing, climbing).
- **Multi-human reconstruction + ego-view RGB-D rendering** (Fig 4):
  pipeline byproducts. Ego-view rendering not used in current policy
  but suggests direction for RGB-conditioned humanoid perception.

## Limitations / open questions

Authors flag five failure modes:

1. **Reconstruction brittleness.** MegaSAM ghost layers under camera
   drift, poor on low-texture surfaces. Dynamic human points sometimes
   fused into static point cloud → "feet buried" artifacts. NKSR
   meshification oversmooths fine geometry (narrow stair treads).
   Videos with lost stair details are discarded.
2. **Retargeting.** Kinematic optimizer assumes all reference poses
   are humanoid-feasible after scale adaptation. In cluttered scenes,
   conflicting foot-contact vs collision-avoidance costs trap solver
   in local minima; RL must "clean up."
3. **Sensing.** 11×11 heightmap sufficient for coarse terrain + chair
   height but too low-res for precise contact, manipulation, or
   overhanging obstacles. Richer perception (RGB-D, learned occupancy)
   is future work.
4. **Simulation fidelity.** Single rigid-mesh scene — no articulated
   / deformable objects. Chair modeled as rigid box, not pivoting
   seat.
5. **Data scale.** 123 clips → occasional jerky recovery motions.
   Larger corpora + real-world fine-tuning needed.

Not flagged but visible from design:
- **Distilled policy dropped target joint angles.** Simplifies
  deployment but sacrifices motion quality for underspecified
  behaviors. Stage 4 RL fine-tune partially compensates.
- **Root direction is human-supplied (joystick).** Not fully
  autonomous. A high-level planner (VLM waypoints, task graph) is
  future work but unaddressed.
- **No comparison against [[huang-2026-pointworld]]-style dynamics
  WMs** — VideoMimic uses simulator as ground-truth dynamics.
  PointWorld would replace the sim rollout with a learned WM. Fusion
  possibility.
- **Manipulation not addressed.** All wiki (E) sources handle object
  manipulation; VideoMimic handles environment-body interaction. The
  two together would be a whole-body loco-manipulation policy.

## Connections

- **[[uc-berkeley]]** — bumps to now **5+ sources** (RTC / RTC-train
  time, CUT3R, LoGeR, VideoMimic). BAIR + Kanazawa + Malik + Abbeel
  + Darrell all present.
- **[[junyi-zhang]]** — **now 2 wiki sources** (LoGeR lead + VideoMimic
  co-first). Confirms Darrell-lab affiliation across long-context
  3D-recon + humanoid-video threads. Bumps `sources:` list.
- **New person cluster** (all held at 1 wiki source for now): Arthur
  Allshire (co-first; also IsaacGym co-author), Hongsuk Choi (co-first),
  David McAllister (co-first), Angjoo Kanazawa (senior; foundational
  monocular human motion — HMR, SLAHMR line), Jitendra Malik (senior;
  co-author on RMA + legged-loco + humanoid-loco line), Pieter Abbeel
  (senior; RL / VLA / dexterous line + Physical Intelligence advisor),
  Trevor Darrell (senior; supervises LoGeR + VideoMimic).
- **[[4d-reconstruction]]** — VideoMimic is a downstream **consumer**
  of 4D reconstruction (MegaSAM + MonST3R), unlike existing wiki 4D
  sources which are producers. Extends the "what is 4D good for?"
  answer beyond tracking + rendering to **robot-skill learning**.
- **MegaSAM / MonST3R** — cited by 5+ wiki sources; VideoMimic uses
  as substrate. MonST3R now cited in **9 wiki sources** counting
  this — well past promotion threshold to primary source (still
  deferred; no PDF in raw/).
- **[[cotracker3]]** — Berkeley + Meta / Karaev line; cited-adjacent
  but not directly used by VideoMimic.
- **Not-yet-ingested baselines / substrates:**
  - **DeepMimic** (Peng, Abbeel, Levine, van de Panne, SIGGRAPH 2018)
    — the tracking + RL blueprint VideoMimic extends.
  - **SFV** (Peng, Kanazawa, Malik, Abbeel, Levine 2018) — direct
    predecessor: RL of physical skills from monocular videos.
    Kanazawa + Malik continuity across 7 years to VideoMimic.
  - **SLAHMR** (Ye et al. 2023) — global human-scene positioning
    method VideoMimic bootstraps from.
  - **HMR / SMPL** — Kanazawa's foundational human body model.
  - **Egocentric Loco** (Agarwal, Kumar, Malik, Pathak 2022),
    **ASAP** (He et al. 2025), **H2O** (He et al. 2024), **ExBody2**
    (Ji et al. 2025), **Humanoid Parkour** (Zhuang et al. 2024) —
    the humanoid-RL competitors in Table 1.
  - **Isaac Gym** (Makoviychuk et al. 2021) — sim substrate; Arthur
    Allshire is a co-author (worth noting).
  - **VIMO, ViTPose, BSTRO, MegaSAM, MonST3R, GeoCalib, NKSR** —
    perception + geometry substrates. Any could earn tool pages if
    ingested.

## Citation

```
Allshire, A., Choi, H., Zhang, J., McAllister, D., Zhang, A., Kim,
C. M., Darrell, T., Abbeel, P., Malik, J., & Kanazawa, A. (2025).
Visual Imitation Enables Contextual Humanoid Control. In Proceedings
of the 9th Conference on Robot Learning (CoRL 2025).
arXiv preprint arXiv:2505.03729.
```
