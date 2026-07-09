---
type: method
title: VideoMimic
status: growing
tags: [humanoid, whole-body-control, real-to-sim-to-real, imitation-learning, reinforcement-learning, smpl, 4d-reconstruction, deepmimic, unitree-g1, monocular-video]
related:
  - "[[4d-reconstruction]]"
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[loger]]"
sources:
  - "[[allshire-2025-videomimic]]"
created: 2026-07-09
updated: 2026-07-09
---

# VideoMimic

Real-to-sim-to-real pipeline turning **monocular RGB video** into a
**contextual whole-body humanoid policy**. Reconstructs 4D
human-scene geometry, retargets human motion to Unitree G1, trains
RL policy in sim over reconstructed terrains, distills to
proprioception + heightmap + root-direction observation set, deploys
onboard at 50 Hz. First real-world context-aware humanoid learned
from in-the-wild video. CoRL 2025 Best Student Paper.

## One-line summary

Video → 4D reconstruction (SMPL + MegaSAM) → retarget to G1 →
DeepMimic-style RL over terrain → DAgger distill → PPO fine-tune →
single unified policy that stair-climbs, sit-stands, walks rough
terrain from context alone.

## Inputs / outputs

**Input (training data):** Monocular smartphone RGB video (any
casually captured clip of humans doing activities). 123 clips used.

**Input (deployment):**
- Proprioception history (5 steps): `q`, `q̇`, `ω`, projected gravity,
  previous action.
- 11×11 LiDAR heightmap around torso (0.1m grid).
- Root-direction command (joystick or high-level planner).

**Output:** Low-level motor actions for 23-DoF Unitree G1 at 50 Hz.

## How it works

### 1) Real-to-sim data acquisition

```
Video → Grounded SAM2 → person detection
     → VIMO → SMPL θ, β, J3D per frame
     → ViTPose → 2D keypoints J2D
     → BSTRO → foot contact
     → MegaSAM / MonST3R → world point cloud + camera pose

Coarse init (SLAHMR-style): focal + limb-length ratio → per-frame similarity

Joint optimization (JAX, Levenberg-Marquardt):
  arg min  w3D · L3D + w2D · L2D + L_smooth
  over  γ (global translation), ϕ (global orientation),
        θ (local pose), α (scene scale)
  Metric anchor: SMPL height prior recovers absolute scale.

GeoCalib → gravity align
NKSR    → point cloud → lightweight mesh
Kinematic retarget → G1 joint angles under joint-limit + contact
                     + collision constraints
```

Optional pre-optimization pass: refit SMPL β to G1 dimensions →
scene rescaled to robot proportions (improves policy training; NOT
applied at real deployment).

Runtime: ~20 ms per 300-frame sequence on A100 after JAX
compilation.

### 2) Policy training (IsaacGym + PPO)

Four-stage curriculum:

- **Stage 1 — MoCap Pre-Training (MPT).** LAFAN → G1 retargeting.
  PPO on flat ground. Conditions: target joint angles + target root
  roll/pitch + root direction. Deployable alone (Fig 9 in source).
- **Stage 2 — Scene-conditioned tracking.** Init from MPT. Add 11×11
  heightmap via residual projection into latent (init weight 0).
  Batched DeepMimic tracking across reconstructed video terrains.
  Reference State Init + motion load balancing (upweight low-SR
  motions).
- **Stage 3 — DAgger distillation.** Distill to policy that observes
  ONLY proprio + heightmap + root direction (drops target joint angles
  + root roll/pitch). Root direction now the only skill signal at
  deployment.
- **Stage 4 — Under-conditioned RL fine-tune.** Another PPO round
  under distilled observations. Recovers behaviors that were
  distillation-suboptimal. Also lets lower-quality reference motions
  contribute without collapsing.

### 3) Rewards

Data-driven tracking only:
- Link + joint position tracking.
- Joint velocity tracking.
- Foot-contact matching (from BSTRO).
- Action-rate penalty to suppress sim-physics exploitation.

No handcrafted skill-specific rewards.

### 4) Deployment on Unitree G1

- 50 Hz onboard control loop.
- Fast-lio2 LiDAR + probabilistic terrain mapping → 11×11 heightmap
  at 0.1m grid, torso-aligned.
- Joystick for root direction.
- Low joint gain Kp=75 (soft contacts).
- Two critical sim-to-real tunings:
  (a) relaxed episode-termination tolerances vs reference,
  (b) realistic physics perturbations during training.

## Training data

- **LAFAN MoCap** retargeted to G1 → Stage 1 pretraining corpus.
- **123 smartphone videos** curated in-the-wild (sitting, standing,
  stair up/down/backwards, block stepping) → Stages 2–4 corpus.

## Where applied

- SLOPER4D reconstruction eval: **WA-MPJPE 112.13 mm** (vs TRAM
  149.48, WHAM 189.29), **Chamfer 0.75** (vs TRAM 10.66).
- Real Unitree G1: stair ascent + descent (indoor + outdoor), chair /
  bench sit-stand, kerbs, rough terrain, steep earthen slopes,
  vegetation. Single policy. Emergent single-leg-hop recovery under
  slip.
- MPT ablation confirms MoCap pretraining is load-bearing (removing
  it → policy fails to balance across 30k iters).

## Known limitations

- MegaSAM ghost layers under camera drift; poor on low texture.
- NKSR oversmooths fine geometry (stair treads).
- Retargeting solver stuck in local minima under cluttered scenes.
- 11×11 heightmap too coarse for manipulation / precise contact.
- Single rigid-mesh scenes — no articulated / deformable objects.
- 123 clips → occasional jerky recovery motions.
- Root direction human-supplied (no autonomous high-level planner).
- No manipulation — pure locomotion + body-scene interaction.

## Related methods

- **DeepMimic** (Peng et al. SIGGRAPH 2018) — direct ancestor of the
  tracking + RL formulation. Not yet in wiki.
- **SFV** (Peng, Kanazawa, Malik, Abbeel, Levine 2018) — RL of
  physical skills from video; predecessor by same senior authors.
- **Egocentric Loco** (Agarwal, Kumar, Malik, Pathak 2022),
  **ASAP** (He et al. 2025), **H2O / ExBody2** (He/Ji et al. 2024/25),
  **Humanoid Parkour** (Zhuang et al. 2024) — humanoid-RL competitors
  in Table 1 of source.
- **[[huang-2026-pointworld]]** — different manipulation-focused role;
  PointWorld learns dynamics, VideoMimic uses simulator as
  ground-truth dynamics. Fusion (PointWorld as learned dynamics inside
  VideoMimic's Stage 2 loop) is unexplored.
- **[[loger]]** — same lead author cluster (Junyi Zhang, Trevor
  Darrell lab). LoGeR = long-context 3D recon; VideoMimic = video →
  humanoid. Two different threads from the same Berkeley group.
- **Substrate models:** MegaSAM, MonST3R, SLAHMR, VIMO, ViTPose,
  BSTRO, SMPL, GeoCalib, NKSR — all off-the-shelf. Isaac Gym for RL
  sim.

## Open directions

- **RGB conditioning at deployment** instead of heightmap. Ego-view
  RGB-D rendering is already a pipeline byproduct.
- **Autonomous high-level planner** (VLM waypoints, task graph) to
  replace joystick root command.
- **Loco-manipulation** — extend to whole-body policies that both
  navigate *and* manipulate objects. Merge with wiki (E) thread.
- **Articulated / deformable scenes** — chair-with-pivot-seat, doors,
  drawers.
- **Larger video corpora** to reduce jerky recoveries.
- **Real-world fine-tuning** as a post-deployment step.
- **Learned dynamics WM instead of Isaac Gym** — [[pointworld]]-style
  point-flow dynamics as the sim rollout backend.
