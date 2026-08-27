---
type: source
source_type: paper
title: "ObjectForesight: Predicting Future 3D Object Trajectories from Human Videos"
authors: [Soraki, Rustin; Bharadhwaj, Homanga; Farhadi, Ali; Mottaghi, Roozbeh]
year: 2026
venue: "arXiv preprint (2026)"
url: https://objectforesight.github.io
raw_path: papers/2601.05237v2.pdf
status: ingested
tags: [manipulation, 6dof-pose, future-prediction, diffusion, human-video, object-centric, egocentric, world-model, se3]
related:
  - "[[objectforesight]]"
  - "[[motionforesight]]"
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[cmp-point-track-manipulation]]"
  - "[[homanga-bharadhwaj]]"
  - "[[rustin-soraki]]"
  - "[[ali-farhadi]]"
  - "[[roozbeh-mottaghi]]"
  - "[[university-of-washington]]"
  - "[[spatialtracker-v2]]"
  - "[[epic-kitchens-dataset]]"
  - "[[diffusion-policy]]"
created: 2026-08-27
updated: 2026-08-27
---

# ObjectForesight: Predicting Future 3D Object Trajectories from Human Videos

## TL;DR

Predicts **future 6-DoF object pose trajectories** (SE(3)) of a
manipulated rigid object from a short egocentric human-video prefix.
A geometry-aware **Diffusion Transformer** (PTv3 point encoder → DiT over
depth-normalized 9D pose tokens) samples diverse, physically coherent
future poses. Trained on **2M+ pseudo-GT 3D trajectories** auto-curated
from EPIC-Kitchens. Plan/forecast-only — no policy, no robot. Same author
line as [[motionforesight]] ([[homanga-bharadhwaj]]); this is the
**rigid-pose** sibling that [[motionforesight]] later argues is a special
case of dense scene flow.

## Why it matters

Second forecast-only member of the "predict future object motion from
passive human video" branch (thread **E**), and the **pose-based**
counterpart to [[motionforesight]]'s scene-flow branch. Three axes of
contrast that make the pair a clean natural experiment:

- **Representation:** rigid 6-DoF SE(3) pose (ObjectForesight) vs dense
  reference-anchored scene flow ([[motionforesight]]). MotionForesight
  argues flow strictly generalizes pose (articulated / deformable / local
  nonrigid); ObjectForesight argues explicit object-centric SE(3) is more
  physically grounded and directly usable by manipulation frameworks.
- **Model:** trained-from-scratch **diffusion** DiT (ObjectForesight) vs
  **repurposed frozen video-DiT + rank-32 LoRA** ([[motionforesight]]).
- **Uncertainty:** ObjectForesight's diffusion is explicitly **multimodal**
  (one-to-many futures) — it directly targets [[motionforesight]]'s flagged
  open weakness (deterministic, motion-magnitude-timid single-future). The
  price is pose-only, rigid-only, short-horizon.

Both **canonicalize to anchor-frame camera coordinates to disentangle
ego-motion from object motion** — the shared load-bearing trick of the
forecast-from-egocentric-video branch.

**For the user's real-time 3D streaming-tracker project:** ObjectForesight
is a *consumer* of trackers/reconstructors, not a competitor — its curation
pipeline stacks [[spatialtracker-v2|SpaTrackerV2]] (metric depth + camera) +
FoundationPose + TRELLIS + SAM2 offline. Confirms the recurring pattern:
downstream forecasting/manipulation work needs metric, ego-disentangled 3D
tracks, and currently assembles them from a slow offline stack — the gap a
real-time 3D streaming tracker fills.

## Key claims

- **Introduces + formalizes the task** of 3D object-dynamics prediction
  (future 6-DoF trajectories) from passive human video, without action
  supervision (§1, contributions).
- **Diffusion beats autoregressive and video-generation** on the same
  object-centric representation (§4.4, Tab 1): on EPIC-Kitchens,
  ObjectForesight-DiT ADE **0.016 m** / ARE **2.30°**, vs AR variant
  0.067 m / 9.48° (AR *underperforms even constant-velocity*), and cuts
  ADE 41% / FDE 45% over constant velocity. On HOT3D-Clips: 0.021 m /
  8.92° ADE/ARE.
- **Direct SE(3) beats generate-then-recover** (§4.4): vs Luma Ray3
  video-gen → pose-recovery, 0.029 m vs 0.084 m ADE, 7.29° vs 12.86° ARE
  (on a 20-video subset — Ray3 too expensive to run at scale). Same
  "predict the geometric variable directly, skip render-then-reconstruct"
  argument as [[motionforesight]].
- **2M+ pseudo-GT 3D trajectory dataset** auto-curated from EPIC-Kitchens
  with zero manual annotation — "first at this scale, fidelity, and
  semantic diversity" for object-centric 3D trajectories (§1, §3.2).
- **PTv3 is the right scene encoder** (§4.4, Tab 2a): PointTransformerV3
  beats DGCNN, PointNet++, SparseConv, No-Encoder. Notably a *poorly
  suited* geometric backbone (SparseConv) is no better than no scene
  encoding at all (2.700° vs 2.690° ARE) — scene conditioning only helps
  with a good encoder.
- **Scales with capacity** (Tab 2b): DiT 6L-384D → 12L-768D drops ARE
  4.242° → 2.299°, ADE 0.0193 → 0.0165 m.

## Methods

### Task (§3.1)

Observe `C` frames (default **C=3**) → predict next **H=8** future 6-DoF
poses. Each pose `p_t = [x,y,z, r_6D]` (translation + continuous 6D
rotation). All poses expressed in **anchor-frame (first future frame)
camera coordinates** to isolate object motion from ego-motion. Anchor-frame
depth back-projected to a point cloud `X`; normalized bboxes give coarse
spatial cues.

### Data curation pipeline (§3.2, Fig 2) — offline, 8 stages

EPIC-Kitchens action segments → prefilter (≤10 s) → **EgoHOS** hand-object
segmentation → **SAM2** object masks with temporal-consensus prompts →
**InternVL3** VLM gating (is object actually moved by hand? view quality) →
**TRELLIS** mesh from clean views (template only, not a training input) →
**SpaTrackerV2** ([[spatialtracker-v2]]) metric depth + camera geometry +
**DiffusionVAS** amodal mask completion → **FoundationPose** 6-DoF init +
bidirectional tracking + re-registration (3 egocentric mods: metric-scale
estimation from depth vs mesh radii, multi-view init, IoU-triggered
re-registration) → sliding (C+H) windows, anchor-frame canonicalized.
**Yield: 3.06M raw → 2.07M filtered trajectories** (each a 0.13 s window).

### Model (§3.3, Fig 3)

1. **Context encoding:** each conditioning frame `[P_k B_k] ∈ R^13` → 64D;
   anchor token attends over conditioning tokens (+ sinusoidal
   relative-time) → context vector `ctx ∈ R^64`.
2. **Geometry encoder:** anchor point cloud `X` → **PointTransformerV3**;
   points carry anchor-camera + anchor-object-frame coords (object-centric);
   FiLM-conditioned on `ctx`; object-centric attention pool → scene
   embedding `z_geom ∈ R^512`.
3. **Pose tokens:** depth-normalized `y_t = [u=x/z, v=y/z, s=log z, r_6D]`
   (9D), channel-standardized; context poses prepended as prefix.
4. **DiT:** denoises future 9D tokens; cosine β-schedule, **v-parameterized**,
   T=1000; `z_geom` + timestep injected via **AdaLN-Zero**; token-type
   (context vs future) + signed anchor-relative-time embeddings.
5. **Sampling:** deterministic **DDIM**, S=50 steps → future trajectory in
   anchor frame; de-normalize + invert log-depth to recover metric SE(3).

### Losses (§3.3)

`L_total = L_v + L_aux + L_zmin + λ_vel·L_vel + λ_acc·L_acc`. `L_v` =
v-prediction MSE with P2 SNR-weighting (+ horizon-aware weight 1→3).
`L_aux` = decoded-SE(3) geodesic rotation + translation error (weighted by
ᾱ_τ). `L_vel/L_acc` = SE(3) velocity/acceleration smoothness on
increments. `L_zmin` = depth-floor penalty. λ_R=2.0, λ_trans=20.0,
λ_vel=0.5, λ_acc=0.1.

## Results (headline)

**EPIC-Kitchens** (Tab 1): ObjectForesight-DiT **ADE 0.016 m, FDE 0.029 m,
ARE 2.30°, FRE 4.82°** — best on all six metrics. AR variant 0.067 m /
9.48° (worse than constant-velocity 0.027 m / 2.47° on translation).
**HOT3D-Clips:** 0.021 m / 8.92°; constant-velocity gap much larger
(0.136 m / 38.70°) — harder motions. **vs Luma Ray3** (20-vid subset):
0.029 vs 0.084 m ADE.

Ablations (Tab 2): PTv3 best encoder; 12L-768D best DiT scale.

## Limitations / open questions

- **Rigid objects only.** No articulated / deformable — the exact
  generality [[motionforesight]] claims scene flow captures and pose
  cannot. Authors flag articulated kinematic models / deformation fields
  as future work (§5).
- **Very short horizon.** C=3 → H=8, ~0.13 s windows (EPIC) / 1.33 s
  (HOT3D @ 6 fps). Not long-horizon planning.
- **Pseudo-GT ceiling.** Entire supervision is auto-curated (SpaTrackerV2 +
  FoundationPose + TRELLIS + SAM2); inherits every stage's error. No
  human-annotated 3D ground truth on EPIC (HOT3D has cleaner GT but lab
  setting).
- **Video-gen baseline on 20 videos only** — Ray3 comparison is
  suggestive, not at scale (compute-limited).
- **No downstream policy / robot.** Plan-only; integration with
  manipulation is future work (§5). Same open end as [[motionforesight]].
- **Single object per clip.** No multi-object interaction.

## Connections

- **Sibling — [[motionforesight]]** (same author [[homanga-bharadhwaj]]):
  the scene-flow branch. Pose-vs-flow, diffusion-from-scratch-vs-repurposed-
  LoRA, multimodal-vs-deterministic. MotionForesight's related work cites
  this and argues flow ⊃ pose; conversely ObjectForesight's **diffusion**
  directly answers MotionForesight's deterministic-single-future weakness.
  Chronology: ObjectForesight is the earlier arXiv (2601, Jan 2026 v1;
  this is v2, Mar 2026).
- **Thread (E) — [[point-tracks-as-manipulation-interface]]:** a
  *contrast* member, not a point-tracks method. The concept's "object
  meshes / 6-DoF pose — need clean object models, hard to estimate" bullet
  is exactly this design point; ObjectForesight shows the pose branch is
  viable *if* you auto-curate pose at scale. Both it and MotionForesight
  populate only the plan half (no control).
- **Perception substrate — [[spatialtracker-v2]]:** used offline for metric
  depth + camera in curation. Reinforces "downstream work needs metric
  ego-disentangled 3D from a slow offline stack."
- **Modeling kin — [[diffusion-policy]] / [[diffusion-forcing]]:** same
  DiT-denoising-over-a-horizon recipe, here over SE(3) pose tokens rather
  than action chunks. v-parameterization + AdaLN-Zero + DDIM.
- **Tools introduced (not yet promoted):** FoundationPose, TRELLIS, SAM2,
  EgoHOS, InternVL3, DiffusionVAS, **PointTransformerV3** (now in 3 wiki
  sources — [[stride]], [[pointworld]], this — promotion candidate).
- **Authors:** [[rustin-soraki]] (UW lead), [[homanga-bharadhwaj]] (3rd
  wiki source; his 3rd distinct research node — 2D-tracks→policy,
  3D-scene-flow-forecast, now 6-DoF-pose-forecast), [[ali-farhadi]] +
  [[roozbeh-mottaghi]] (UW seniors — new to wiki).

## Citation

```
Soraki, R., Bharadhwaj, H., Farhadi, A., & Mottaghi, R. (2026).
ObjectForesight: Predicting Future 3D Object Trajectories from Human
Videos. arXiv:2601.05237v2 [cs.CV], 22 Mar 2026. University of Washington
+ CMU Robotics Institute. Project: https://objectforesight.github.io
```
