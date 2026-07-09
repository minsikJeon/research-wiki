---
type: method
title: STRIDE
status: growing
tags: [4d-reconstruction, driving-scenes, multi-modal, lidar, point-transformer, gaussian-splatting, scene-flow, instance-decomposition, self-supervised, feed-forward]
sources:
  - "[[anon-2026-stride]]"
related:
  - "[[4d-reconstruction]]"
  - "[[any4d]]"
  - "[[v-dpm]]"
  - "[[fsm]]"
  - "[[cut3r]]"
  - "[[vggt]]"
  - "[[pointworld]]"
  - "[[feed-forward-3d-reconstruction]]"
created: 2026-06-26
updated: 2026-07-09
---

# STRIDE

**S**elf-supervised, generalizable, **d**ecomposable 4D **r**econstruction
of multi-modal **d**riving sc**e**nes. The first feed-forward 4D model for
driving scenes that (a) reasons in 3D point space over fused camera +
LiDAR observations via a **Point Transformer v3** backbone, and (b)
emits **learned dynamic instance assignments** alongside per-Gaussian
geometry, appearance, and velocity — all without human annotations.

## One-line summary

Lift camera features into 3D via LiDAR-anchored depth → PTv3 reasoning
backbone → per-point 3DGS with velocity → instance-token decomposition
via differentiable similarity matrix.

## Inputs / outputs

**Inputs (per training/inference call):**
- Multi-view camera images `I^(t,v) ∈ R^{H×W×3}` across timestamps `t`
  and views `v`.
- LiDAR point clouds `P^(t) ∈ R^{N_t × 3}`.
- Calibration: camera intrinsics + extrinsics relative to LiDAR.

**Outputs:**
- A set of 3D Gaussian primitives `{g_i}`, each
  `(µ_i, σ_i, q_i, z_i, α_i, v_i)` = position, scale, quaternion, color,
  opacity, **velocity**.
- A partition of Gaussians into a static set `G_static` and `M`
  dynamic instance sets `{G_dyn^m}_{m=1}^M`.
- Renderable via standard 3DGS rasterization at any target timestamp
  (using linear motion model with per-Gaussian velocity).

## How it works

Three sequential stages.

### Stage 1: Multi-modal feature extraction

For each `(t, v)`:
1. Project LiDAR `P^(t)` onto image plane to derive sparse depth
   `D_lidar^(t,v) ∈ R^{H×W}` + availability mask
   `M_lidar^(t,v) ∈ {0,1}^{H×W}`.
2. Concatenate `(I^(t,v), D_lidar^(t,v), M_lidar^(t,v))` → **5-channel
   input**.
3. ViT encoder → MLP heads → per-pixel dense depth `D^(t,v)`, sky
   mask `S^(t,v)`, feature map `F^(t,v) ∈ R^{H×W×C}`.

Sky mask is used to *decouple* sky and foreground processing later.
Dense depth is supervised by `D_lidar` (where available) plus
photometric reconstruction.

### Stage 2: 4D scene reasoning with PTv3

**Foreground branch:**
1. For each foreground (non-sky) pixel feature in `F^(t,v)`,
   back-project to 3D using predicted `D^(t,v)` + calibration.
2. Augment per-point feature with embeddings of: 3D xyz, time `t`,
   view `v`, pixel ray direction (Plücker coordinates).
3. Aggregate across all `(t, v)` → unified point cloud `F_fg` (3D
   feature points, color-coded by time in Fig 2).
4. **PTv3 backbone** refines `F_fg → F_fg'` via multi-scale patch
   attention with serialized neighborhoods.
5. MLP decoders predict per-point Gaussian parameters.

**Sky branch (parallel, lightweight):**
- Initialize a 400m-radius Fibonacci sphere of points.
- Filter to those projecting into any image's sky region.
- Each point's input feature = (xyz, projected rgb, scale = average
  distance to 3 nearest neighbors) — 9-channel input.
- Separate small PTv3 decodes `N_sky` sky Gaussians.

### Stage 3: Dynamic object decomposition

**Token initialization:**
1. Velocity-threshold to isolate dynamic Gaussians:
   `F_dyn = { f'_fg,i | ‖v_i‖₂ > τ }`, count `N_dyn`.
2. Align dynamic Gaussians to a common reference time using their
   predicted velocities.
3. DBSCAN-cluster the aligned dynamic points → cluster centroids
   `{c_j}_{j=1}^M`.
4. Each centroid → one instance token `t_j` with (3D position +
   learnable embedding).

**Token-point association:**
1. Concatenate dynamic point features + instance tokens; process
   jointly through a second **"Instance PT"** transformer →
   updated `{f'_dyn,i}` and `{t_j}`.
2. Compute differentiable similarity matrix:

```
d_feat(i,j)    = 0.5 · ( (H_p(f'_dyn,i) · H_t(t_j))
                       / (‖H_p(f'_dyn,i)‖ · ‖H_t(t_j)‖)  + 1 )  ∈ [0,1]
d_spatial(i,j) = ‖ µ_i − c_j ‖₂
ζ_j            = sigmoid( MLP(t_j) )                            # existence
s_{i,j}        = ζ_j · ( d_feat(i,j) − min(1, λ_spat · d_spatial(i,j)) )
```

`H_p`, `H_t` = projection MLPs into shared latent space.
`λ_spat` controls how aggressively the spatial bias punishes
far-away assignments. The `min(1, ·)` clamp handles large-extent
instances.

3. Hard assignment: `G_dyn^m = { g_i | argmax_j s_{i,j} = m }`.
   Empty assignments (`ζ_j → 0`) collapse to "no instance."

**Velocity decoding from tokens (Eq for back-prop):**

```
v_tok,j = MLP_token-vel(t_j)                                    # ∈ R³
u_i = Σ_j w_{i,j} v_tok,j + tanh( MLP_residual(f'_dyn,i) )
w_i = one_hot( argmax(s_i) )                                    # STE on softmax
```

The first term is the token-shared rigid-body velocity; the second
is a small per-Gaussian residual to handle non-rigid deformation.
Reconstruction loss flows through `u_i` → through `s_i` (via STE) →
back to token features → encouraging consistent groupings.

## Training

### Stage 1 (backbone): photometric reconstruction

```
L_recon = L_photo + λ_v-reg · L_v-reg + λ_sky · L_sky + λ_mde · L_mde
```

- `L_photo` = L2 rendered RGB + perceptual loss + L1 rendered depth.
- `L_v-reg = Σ_i ‖v_i‖₁` — velocity sparsity (prefer static
  hypothesis until evidence).
- `L_sky` = BCE on predicted sky mask vs off-the-shelf sky
  segmentation.
- `L_mde` = L1 on predicted depth vs sparse LiDAR depth (where
  available).

Linear motion model is used to render at any target time:
`µ_i(t_target) = µ_i + (t_target − t_ref) · v_i`.

### Stage 2 (instance decomposition): same photo loss + smoothness

Freeze ViT + PTv3 backbones; only instance module trains.

```
L_decomp = L_photo + λ_v-reg · L_v-reg + λ_smooth · L_smooth
```

Photometric loss now flows through token-decoded `u_i` instead of
raw `v_i`, supervising the assignment.

**Smoothness prior** (Eq 1):

```
ŷ_i = Mode{ argmax(s_k) : ‖µ_i − µ_k‖₂ < τ_dist  ∧  ‖v_i − v_k‖₂ < τ_vel }
L_smooth = Σ_i CE( softmax(s_i), one_hot(ŷ_i) )
```

Pull each Gaussian's assignment toward the modal assignment among
its spatial-and-kinematic neighbors. Suppresses over-decomposition;
without it, excess tokens fragment single objects across multiple
ids.

## Where it's been applied

- **Waymo Open Dataset (WOD)** — STORM's sparse-input setup:
  2-second clips with frames {0, 5, 10, 15} as input, evaluate on
  the remaining 16 frames.
- **PandaSet** — Flux4D's dense-input setup: 1.5-second clips with
  6 input frames, evaluate on 5 interpolated + 5 extrapolated
  frames.
- **Scene-editing demos (Fig 5):** actor removal, manipulation,
  injection enabled by the learned instance decomposition.

See [[anon-2026-stride]] for numeric results.

## Known limitations

- **Constant-velocity motion.** Per-Gaussian linear motion cannot
  capture non-rigid or curved-trajectory motion. Pedestrians,
  articulated bicyclists, and turning cars are the obvious failure
  cases the paper acknowledges (§5).
- **Decomposition relies on motion cues.** Static-but-distinct
  objects (parked cars, traffic poles, separate buildings) cannot
  be decomposed because the supervision signal is motion-derived.
- **Memory-bounded input window.** PTv3 attends across all input
  observations simultaneously → memory scales with `Σ_t,v H·W`.
  Long-horizon driving sequences (10s+) are not supported.
- **Instance gain over DBSCAN is modest** (mPrec 51.0 → 53.4).
  Initialization is doing most of the work; the learnable component
  refines, but doesn't transform.
- **No comparison vs per-scene optimization baselines** (DIAL-GS,
  IDSplat, UniRe). The "first feed-forward learnable decomposition"
  claim is scope-conditioned.

## Related methods

- **[[any4d]]** — closest cousin: feed-forward multi-modal 4D for
  driving-like scenes. Different design choice — Any4D factors into
  ego (depth+ray) + allo (pose+scene flow) + metric scale, with
  full attention over multi-modal inputs. STRIDE uses 3D point
  reasoning natively + decomposition module. No published head-to-head.
- **[[v-dpm]]** — image-only feed-forward 4D via Dynamic Point Maps.
  Same downstream goal (4D scene + motion), no LiDAR, no
  decomposition.
- **[[fsm]] / [[lacet]]** — chunked elastic TTT 4D. Image-only NVS
  rather than driving + Gaussian output. STRIDE could in principle
  swap its PTv3 backbone for LaCET blocks to unlock long-horizon
  inference.
- **[[cut3r]]** — recurrent online 3D reconstruction with persistent
  state. The streaming analogue STRIDE lacks; a STRIDE+CUT3R-style
  recurrence would address Limitation #3.
- **[[streamvggt]]** — KV-cached streaming distillation of VGGT.
  The "cached spatial memory" alternative to CUT3R; STRIDE's
  per-point PTv3 features could be cached similarly.
- **STORM** (not yet ingested) — STRIDE's primary camera-only
  baseline. STRIDE re-implements a multi-modal variant (STORM+) for
  fair comparison.
- **Flux4D** (not yet ingested) — LiDAR-centric feed-forward 4D
  baseline. STRIDE's PTv3 backbone is positioned as a superior
  replacement for Flux4D's sparse-conv UNet.
- **DIAL-GS / IDSplat / UniRe** (not yet ingested) — concurrent
  self-supervised instance-decomposition methods that require
  per-scene optimization. STRIDE is the feed-forward alternative.
