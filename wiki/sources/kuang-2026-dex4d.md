---
type: source
source_type: paper
title: "Dex4D: Task-Agnostic Point Track Policy for Sim-to-Real Dexterous Manipulation"
authors: [Kuang, Yuxuan; Park, Sungjae; Fragkiadaki, Katerina; Tulsiani, Shubham]
year: 2026
venue: "arXiv preprint (Feb 2026)"
url: https://arxiv.org/abs/2602.15828
raw_path: papers/2602.15828v1.pdf
status: ingested
tags: [point-tracking, manipulation, dexterous, sim-to-real, reinforcement-learning, video-generation, world-model]
sources: []
related:
  - "[[dex4d]]"
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[bharadhwaj-2024-track2act]]"
  - "[[zhi-2025-3dflowaction]]"
  - "[[cotracker3]]"
  - "[[shubham-tulsiani]]"
  - "[[katerina-fragkiadaki]]"
  - "[[yuxuan-kuang]]"
  - "[[sungjae-park]]"
  - "[[cmu-ri]]"
created: 2026-06-27
updated: 2026-06-27
---

# Dex4D

## TL;DR

**Task-agnostic dexterous policy** trained via a new
**Anypose-to-Anypose (AP2AP)** RL formulation: simulate a 22-DoF
xArm6 + LEAP-hand transforming arbitrary objects from any current 3D
pose to any target 3D pose, across 3,200 UniDexGrasp objects, with no
language conditioning and no task-specific rewards. The key
representation innovation is **Paired Point Encoding** — encode
*pairs* of corresponding (current, target) 3D points with PointNet, so
the network sees correspondence directly rather than two separate
clouds. At deployment: a video generation model (Wan2.6) produces a
task video → SAM2 mask + Co-Tracker3 → metric-calibrated 3D point
tracks → policy condition. **From the user's own group
(Fragkiadaki × Tulsiani, CMU)** — and **the user is acknowledged** in
the paper.

## Why it matters

Three things make this stand out vs. the rest of the manipulation-via-
point-tracks line:

1. **First to apply this paradigm to dexterous (22-DoF) hand
   manipulation.** Prior work (Track2Act, Im2Flow2Act, 3DFlowAction,
   NovaFlow) is parallel-gripper only. Dexterous adds contact-rich
   dynamics that break heuristic flow→action pipelines (cf.
   NovaFlow-CL failures in Table III).
2. **Task-agnostic Anypose-to-Anypose RL** — sidesteps the language-
   conditioned RL pain of designing per-task rewards and environments.
   The policy learns "transform object pose A → pose B" as a *single
   skill*; the high-level planner (video gen → 4D recon → point tracks)
   composes that skill across tasks.
3. **Paired Point Encoding** is the key representation choice — naive
   "encode current points + encode target points separately" loses
   correspondence and tanks performance (5.7% vs. 60% SR in ablation,
   Table II). Pair them first, then encode.

This is the direct CMU descendant of [[bharadhwaj-2024-track2act]]
(same Tulsiani group; both factor manipulation into "plan in point-
track space → execute"). Bharadhwaj is in the acknowledged
discussions (Park × Bharadhwaj × Tulsiani is the DemoDiffusion paper
chain).

## Key claims

- **+22.5% SR over NovaFlow-CL** in real-world dexterous tasks
  (19/40 vs. 10/40 across LiftToy/Broccoli2Plate/Meat2Bowl/Pour;
  Table III).
- **+25.5% SR over open-loop NovaFlow / +16.3% over closed-loop
  NovaFlow-CL** in simulation (Table I; 6-task suite).
- **Paired Point Encoding is critical** — student-policy SR drops
  60.0 → 5.7% (MLP encoding) or → 20.3% (decoupled PointNet
  encoding); Table II.
- **Joint world modeling adds ~3% SR** on top of self-attention
  (Table II); validates the world-model-as-auxiliary-loss pattern.
- **Random plane-height masking** during student distillation enables
  policy robustness to severe finger occlusions at deployment.
- **Zero real-robot demonstrations** required — sim-only training on
  3,200 UniDexGrasp objects + diverse pose configs + extensive domain
  randomization → real-world deployment.

## Methods

### Anypose-to-Anypose (§III-A)

Goal-conditioned MDP. Each episode: random initial pose, random target
pose; once stably achieved, next target is randomly resampled.
No language. Trained with PPO + 3-stage curriculum (single object →
many objects → tougher initializations + slower arm).

### Paired Point Encoding (§III-B, Fig. 3)

```
q_t^i = [p_t^i ; p̄_t^i] ∈ R^6     (current ⊕ target per point)
```
N pairs → PointNet with shared MLPs + mean-max mixed pooling → paired
point features. Preserves both correspondence (paired by index) and
permutation-invariance (PointNet pooling).

### Teacher–student distillation (§III-C)

- **Teacher** (privileged RL): full points sampled on whole object +
  privileged states (torques, fingertip distances, etc.); PPO actor &
  critic.
- **Student** (deployment): masked partial points + propriop + last
  action → MLP tokenizers + PointNet tokenizer → transformer encoder.
  Output: action token + next-state tokens (joint angle, velocity).
  Loss: `L_bc + L_wm` (DAgger BC + L1 world-model loss).

### Deployment pipeline (§IV)

1. Wan2.6 video gen from language + initial RGBD.
2. SAM2 segmentation of initial frame.
3. Co-Tracker3 → 2D object tracks.
4. Video Depth Anything → relative depth, then **median-rescaled** to
   match initial depth observation D₀ (Dex4D's smoothness fix over
   3DFlowAction's per-frame DepthAnythingV2).
5. Lift to 3D → target point tracks.
6. Closed-loop online tracking with CoTracker3 → current points →
   paired-point-condition policy → action.

## Results

### Simulation (6 tasks; Success Rate / Task Progress)

| Method | CL | Avg SR | Avg TP |
|---|---|---|---|
| NovaFlow | ✗ | 0.345 | 0.448 |
| NovaFlow-CL | ✓ | 0.437 | 0.608 |
| **Dex4D** | ✓ | **0.600** | **0.712** |

### Real world (4 tasks, all unseen objects, zero real demos)

| Method | LiftToy | Broc2Plate | Meat2Bowl | Pour | Total |
|---|---|---|---|---|---|
| NovaFlow-CL | 4/10 | 3/10 | 3/10 | 0/10 | 10/40 |
| **Dex4D** | **6/10** | **4/10** | **5/10** | **4/10** | **19/40** |

### Ablation (Table II)

| Variant | SR | TP |
|---|---|---|
| MLP Point Encoding | 0.057 | 0.172 |
| Decoupled (separate PointNet) | 0.203 | 0.363 |
| w/o Self-Attention | 0.490 | 0.675 |
| w/o World Modeling | 0.570 | 0.683 |
| **Dex4D** | **0.600** | **0.712** |

## Limitations / open questions

- **Single-object manipulation only** — no articulated multi-part
  objects (explicitly mentioned as future work).
- **No human grasp priors / HOI data** — LEAP hand's 4-finger
  thick-finger morphology is too far from human hands; future hardware
  + thinner hands could unlock more.
- **No tactile sensing** — vision + point tracks only.
- **Online tracking failures** under fast motion / occlusion are the
  major real-world failure mode; faster, more robust trackers would
  help.

## Connections

- **CMU lineage:** Direct descendant of [[bharadhwaj-2024-track2act]]
  (same Tulsiani group) and [[harley-2022-pips]] /
  [[zhang-2025-tapip3d]] (Fragkiadaki TAP line). The video gen → point
  tracks → policy decomposition is shared with Track2Act, but
  Track2Act uses rigid-transform fitting + parallel-jaw + residual
  policy; Dex4D uses sim-to-real RL + dexterous + Paired Point
  Encoding.
- **Closest contemporary:** NovaFlow (Li 2025, cited but not yet
  ingested) — parallel-gripper "actionable flow" with Kabsch-based
  pose estimation + IK; Dex4D adapts NovaFlow as its primary baseline.
- **Concurrent / contemporary:** [[zhi-2025-3dflowaction]] — also
  uses generated videos + 3D point tracks + optimization, but
  parallel-jaw; Dex4D's relative-depth calibration is an explicit
  improvement over 3DFlowAction's per-frame depth.
- **VLA contemporary:** [[kim-2026-pri4r]] — uses point tracks as
  *privileged supervision* for VLA training rather than as a *policy
  condition*. Dual viewpoint on the same representation.
- **Author overlap:** Minsik Jeon (user) is acknowledged for
  "presentation feedback"; Bharadhwaj, Mittal are in adjacent
  acknowledgments. Park × Bharadhwaj × Tulsiani also co-author
  DemoDiffusion (cited).
- **Point tracker:** [[cotracker3]] (online tracking) for both
  pseudo-labeling and deployment.
- **Concept page:** [[point-tracks-as-manipulation-interface]].

## Citation

```
Kuang, Y., Park, S., Fragkiadaki, K., & Tulsiani, S. (2026).
Dex4D: Task-Agnostic Point Track Policy for Sim-to-Real Dexterous
Manipulation. arXiv preprint arXiv:2602.15828.
```
