---
type: method
title: CoWTracker
status: growing
tags: [point-tracking, dense-tracking, optical-flow, warping, cost-volume-free, transformer, vggt, iterative-refinement]
sources:
  - "[[lai-2026-cowtracker]]"
related:
  - "[[point-tracking]]"
  - "[[joint-point-tracking]]"
  - "[[cotracker]]"
  - "[[cotracker3]]"
  - "[[pips]]"
  - "[[vggt]]"
  - "[[cmp-tap-methods]]"
created: 2026-08-28
updated: 2026-08-28
---

# CoWTracker

**Cost-volume-free dense point tracker.** Replaces the correlation/cost
volume used by every prior TAP method with a **warping** mechanism (borrowed
from optical-flow work WAFT): sample each target frame's features at the
current track estimate, bring them into correspondence with the query frame,
and refine iteratively with a spatial-temporal transformer. Because there is
no quadratic cost volume, it indexes features at high resolution → SOTA
dense point tracking, and the same model transfers zero-shot to optical
flow.

## One-line summary

`u' = Φ(u | F)`: warp target features to the query frame at the current
displacement, concat with query features + hidden state, run a
ViViT-style (spatial ∥ temporal) transformer, predict residual `Δu`; K=5
iterations; [[vggt|VGGT]] backbone + DPT upsampler for ½-stride indexing.

## Inputs / outputs

- **In:** video `{I_t}`, query pixels `P` (from frame 0).
- **Out:** dense displacement field `u_t(p)` → tracks `x_t(p)=p+u_t(p)`,
  visibility `v_t(p)`, confidence `τ_t(p)`. 2D only.
- **As optical flow:** feed a 2-frame "video," same config.

## How it works

1. **Backbone:** VGGT (patch-embed frozen, last blocks finetuned) → joint
   per-frame features `F`.
2. **DPT upsampler** (+ raw-image U-Net) → ½-stride high-res features.
3. **Warping tracker** (K=5 iterations):
   - warp: `G_t(p) = sample(F_t, p + u_t(p))` — the *only* cross-frame
     feature pairing (no cost volume);
   - concat `z_t = G_t ⊕ F_0 ⊕ u_t ⊕ h_t`;
   - **spatial-temporal transformer** — 2 spatial self-attn (over points P)
     per 1 temporal attn (over frames T), ViViT-style;
   - linear head → residual `Δu`; update `u`, hidden state `h`.
   - visibility/confidence = sigmoid readouts on final `h`.

Head cost is **linear** in T × |P| × K — the enabler of high-res indexing.

Trained on **Kubric only**, Huber (visible+occluded) + BCE (vis/conf),
50k iters @ 336×560.

## Where it's been applied

All in [[lai-2026-cowtracker]]:
- **Dense point tracking** — TAP-Vid DAVIS/Kinetics/RGB-Stacking + RoboTAP;
  mean **AJ 71.3 / δavg 81.8 / OA 93.3**, SOTA (beats AllTracker).
- **Zero-shot optical flow** — Sintel 0.78/1.48 EPE, KITTI 1.04/4.87%,
  Spring 0.17/0.75%; beats specialized RAFT/SEA-RAFT/WAFT on several.

## Known limitations

- **Not streaming/causal** — offline; VGGT backbone quadratic in video
  length (chunk long clips). Throughput backbone-dominated.
- **Kubric-only** synthetic training; no real-data/pseudo-label recipe.
- **2D only** (no depth/3D).
- Fails: extreme viewpoint change, long full occlusion, specularities.
- Refinement saturates at K=5–6.

## Related methods

- **Departs from all cost-volume trackers:** [[pips]], [[tapir]],
  [[cotracker]], [[cotracker3]], [[tapnext]], [[spatialtracker-v2]] build
  correlation/cost volumes; CoWTracker warps instead — new matching axis on
  [[point-tracking]].
- **Joint-tracking kin:** [[cotracker]] — CoWTracker frames CoTracker's
  cross-track attention as equivalent to WAFT-style global warp reasoning;
  shares the spatial-temporal-attention pattern ([[joint-point-tracking]]).
- **Backbone:** [[vggt]] — best feature extractor (ablation); also the
  quadratic-length limitation.
- **External flow lineage:** WAFT (warp-based optical flow), RAFT, SEA-RAFT.
- **AllTracker** — prior dense-tracking SOTA it beats (cost-volume based).
