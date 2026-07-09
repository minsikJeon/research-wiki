---
type: method
title: Pri4R
status: growing
tags: [vla, manipulation, point-tracking, privileged-information, auxiliary-supervision, imitation-learning]
sources:
  - "[[kim-2026-pri4r]]"
related:
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[vla]]"
  - "[[track2act]]"
  - "[[dex4d]]"
  - "[[im2flow2act]]"
  - "[[3d-flow-action]]"
  - "[[mu0]]"
  - "[[pointworld]]"
created: 2026-06-27
updated: 2026-07-09
---

# Pri4R

## One-line summary

Adds a lightweight **3D point-track head** to a VLA backbone during
fine-tuning; the auxiliary L1 loss on per-step displacements forces the
backbone to encode 4D dynamics. Head is **discarded at inference** →
zero overhead, identical architecture, +13.2% RoboCasa for OpenVLA-OFT.

## Inputs / outputs

- **Train inputs:** standard VLA inputs (multi-view images, language,
  proprioception) + offline-precomputed 3D point tracks `{P_τ}` for
  every demo trajectory (Np = 1024 points).
- **Train outputs:** standard action chunk + per-step 3D displacements
  `ΔP̂_{t:t+H} ∈ R^{H×Np×3}` from the head (training only).
- **Inference inputs/outputs:** unchanged from base VLA.

## How it works

### Architecture (Fig. 2)

- **Point MLP:** P_t ∈ R^{Np×3} → e_t ∈ R^{Np×d}.
- **Fusion MLP:** broadcast `z_t ⊕ e_t` over (H, Np) → output
  `ΔP̂_{t:t+H} ∈ R^{H×Np×3}`.
- **z_t (backbone embedding sequence):**
  - **OpenVLA-OFT:** final-layer hidden states of action-query tokens
    (the ones already mapped to actions by the action head).
  - **π series (π₀, π₀.₅):** a transformer **embedding module** with
    learnable query tokens that cross-attends to the final-layer
    image+language tokens, producing `z_t ∈ R^{H×d}`. Ablation (Table
    V) shows this beats point-expert / learnable-query-only variants.

### Loss

```
L = L_act + ω_pt · ‖ΔP̂_{t:t+H} − ΔP_{t:t+H}‖_1
```

with `ω_pt = 1`, `N_p = 1024`. Per-step **displacements**, not
absolute positions.

### Why 3D point tracks (§IV-B; ablation Table III)

| Supervision | RoboCasa Δ |
|---|---|
| None (baseline) | 0 |
| Goal point set (Pt+H only) | +0.7 |
| 2D point track | +3.9 |
| Depth maps (VAE-latents) | +8.3 |
| **3D point track** | **+13.2** |

Need both **temporal density** (full horizon) and **metric 3D
geometry** for the gain. Also need to track **both robot and scene
points** (+13.2 vs. +10.7 robot-only vs. +2.1 scene-only).

### Data construction

- **Simulation:** init Np query points at frame 1 from mesh-cropped
  cube; store face indices + barycentrics; rollout to get `{P_τ}^T`
  via mesh queries (deterministic ground truth).
- **Real world:** off-the-shelf 3D point tracker (ref [61]) +
  segmentation-biased foreground sampling.

## Where it's been applied

[[kim-2026-pri4r]] — augments OpenVLA-OFT, π₀, π₀.₅. Benchmarks:
LIBERO (4 suites × 500 trials), RoboCasa (24 tasks × 50 trials),
OMY-F3M real-world (4 tasks).

## Known limitations

- Needs ground-truth or pseudo-labeled 3D tracks at training time
  (annotation pipeline).
- Embedding module design is backbone-specific for π series.
- Pseudo-label quality bottleneck for real-world training.
- No ablation on point density (Np).

## Related methods

- **[[dex4d]]:** uses identical 3D point tracks but as *policy
  conditioning input* rather than auxiliary supervision. Dex4D needs
  a video-gen + tracking stack at deployment; Pri4R needs neither.
- **[[track2act]]:** ancestor in spirit (point tracks → manipulation
  policy); Pri4R repackages the same signal as privileged
  supervision.
- **Visual Trace Prompting** (Wen 2025, ref [68]): 2D version of the
  supervision idea.
- **[[vla]]:** base architectures (OpenVLA-OFT, π series).
- **LUPI (Vapnik):** the Learning Using Privileged Information
  framing; Dex4D's teacher policy is the same philosophical move at
  the policy level.
