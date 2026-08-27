---
type: method
title: ObjectForesight
status: growing
tags: [manipulation, 6dof-pose, future-prediction, diffusion, se3, object-centric, human-video, egocentric, world-model]
sources:
  - "[[soraki-2026-objectforesight]]"
related:
  - "[[motionforesight]]"
  - "[[spatialtracker-v2]]"
  - "[[diffusion-policy]]"
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[epic-kitchens-dataset]]"
created: 2026-08-27
updated: 2026-08-27
---

# ObjectForesight

**Future 6-DoF object-pose forecaster.** Given a short egocentric video
prefix (C=3 frames + monocular depth + anchor-frame object mask), predicts
a distribution over the next H=8 rigid-object SE(3) poses via a
geometry-aware **Diffusion Transformer**. Plan/forecast-only — no policy,
no robot. The **rigid-pose** sibling of [[motionforesight]]'s dense
scene-flow forecaster (same author line).

## One-line summary

PTv3 point encoder → object-centric scene embedding `z_geom`; DiT denoises
depth-normalized 9D pose tokens (v-param, AdaLN-Zero, DDIM) conditioned on
`z_geom` + a past-pose prefix → diverse, physically coherent future SE(3)
trajectory in anchor-frame coordinates.

**Problem**
	 Forecast the future 6 DoF trajectory of object, given a short RGB video clip. Can be used for planning and acting
**Idea**
	 1) Dataset Preparation: egocentric video clips, with 6DoF object poses
	 2) Model (DiT + PointTransformer) that predicts future object traj given previous frame inputs
	 These makes objectforesight a scalable, object centric 3D forward-dynamics model
**Key Details**
	 - diffusion based traj prediction -> better than autoregressive (which converges to avg traj)
**What's not solved**
	- rigid trajectory only, no deformation, etc
	- short context / short prediction length. are they learning enough motion?
	- Can I use the data as eval dataset?
## Inputs / outputs

- **In:** C=3 RGB context frames + monocular depth (anchor-frame point
  cloud `X`), anchor-frame object mask, past object poses `P_1:C` (SE(3)) +
  normalized bboxes.
- **Out:** H=8 future 6-DoF poses `p_t = [x,y,z, r_6D]`, anchor-frame
  camera coords, metric.
- **Modality:** diffusion → **multimodal** (samples multiple plausible
  futures from Gaussian noise).

## How it works

1. **Anchor canonicalization:** all frames expressed in the first-future-
   frame camera coordinates → isolates object motion from ego-motion.
2. **Context attention:** conditioning poses+bboxes → anchor token queries
   them (+ relative-time embedding) → `ctx ∈ R^64`.
3. **Geometry encoder:** point cloud `X` → **PointTransformerV3**; points
   carry anchor-camera + anchor-object-frame coords; FiLM on `ctx`;
   object-centric attention pool → `z_geom ∈ R^512`.
4. **Pose tokens:** depth-normalized `[u=x/z, v=y/z, s=log z, r_6D]` (9D),
   standardized; context poses prepended as prefix.
5. **DiT denoising:** cosine schedule, **v-parameterization**, T=1000;
   `z_geom` + timestep via **AdaLN-Zero**; deterministic **DDIM** (S=50)
   at inference → de-normalize + invert log-depth → metric SE(3).

**Losses:** v-MSE (P2 SNR-weight + horizon 1→3) + decoded-SE(3) geodesic
rotation/translation aux + SE(3) velocity/acceleration smoothness + depth
floor.

## Data pipeline (offline, 8-stage, → 2.07M trajectories)

EPIC-Kitchens → EgoHOS hand-object → SAM2 (temporal-consensus prompts) →
InternVL3 VLM gating → TRELLIS mesh (template only) → **[[spatialtracker-v2|
SpaTrackerV2]]** metric depth+camera + DiffusionVAS amodal masks →
FoundationPose 6-DoF init + bidirectional tracking + re-registration →
sliding (C+H) windows, anchor-canonicalized. 3.06M raw → 2.07M filtered.

## Where it's been applied

All in [[soraki-2026-objectforesight]]: EPIC-Kitchens (ADE 0.016 m / ARE
2.30°) + HOT3D-Clips (0.021 m / 8.92°). Beats AR variant, constant-velocity,
and Luma Ray3 video-gen-then-recover baselines. Plan-only — no downstream
manipulation deployed.

## Known limitations

- **Rigid objects only** (no articulated / deformable) — the generality
  [[motionforesight]]'s scene flow claims over pose.
- **Very short horizon** (C=3→H=8, ~0.13 s EPIC windows).
- **Pseudo-GT ceiling** — supervision entirely auto-curated; inherits
  SpaTrackerV2 + FoundationPose + TRELLIS + SAM2 errors.
- **No policy / robot** — integration is future work.
- Single object per clip.

## Related methods

- **Sibling — [[motionforesight]]** (same author [[homanga-bharadhwaj]]):
  scene-flow vs pose; repurposed-frozen-video-DiT+LoRA vs
  trained-from-scratch-diffusion; deterministic vs **multimodal**.
  ObjectForesight's diffusion answers MotionForesight's deterministic-
  single-future weakness; MotionForesight's flow answers ObjectForesight's
  rigid-only limitation. Two ends of the same forecast-from-human-video
  branch.
- **Modeling kin — [[diffusion-policy]]:** DiT-denoising-over-a-horizon,
  here on SE(3) pose tokens instead of action chunks.
- **Perception substrate — [[spatialtracker-v2]]:** offline metric depth +
  camera in curation.
- **Concept — [[point-tracks-as-manipulation-interface]]:** the pose-based
  *contrast* to the point-tracks family (not a member); plan-only.
