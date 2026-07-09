---
type: method
title: Point3R
status: growing
tags: [3d-reconstruction, streaming, online, pointmap, spatial-memory, 3d-rope, dust3r-family]
sources:
  - "[[wu-2025-point3r]]"
related:
  - "[[cut3r]]"
  - "[[streamvggt]]"
  - "[[dust3r]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[pointmap-representation]]"
created: 2026-07-02
updated: 2026-07-02
---

# Point3R

Online DUSt3R-lineage 3D reconstruction with an **explicit spatial
pointer memory** — each memory unit is a `(3D position, feature)` pair
in the global coordinate system. Alternative to [[cut3r]]'s fixed
implicit state and Spann3R's per-frame cache.

## One-line summary

ViT encodes each frame → intertwined decoders cross-attend image tokens
with a set of 3D-indexed pointer features under a **3D hierarchical
RoPE** → DPT heads emit self-frame + world-frame pointmaps + pose →
memory encoder builds new pointers → distance-based fusion merges them
into the pointer set.

## Inputs / outputs

- **In (per frame):** RGB image `I_t ∈ R^{H×W×3}`; optional streaming
  or unordered.
- **Out (per frame):**
  - `X̂^self_t, C^self_t` — pointmap + confidence in current-camera
    frame.
  - `X̂^global_t, C^global_t` — pointmap + confidence in global
    (frame-1) coords.
  - `T̂_t` — camera pose (quaternion + translation).
- **State:** growing pointer set `M_t = {(P_i ∈ R^3, m_i ∈ R^{768})}`.
  Fusion mechanism caps growth; on WhiteRoom NRGBD scene, ≈1485
  pointers after 26 frames, ≈0.20 s/frame steady-state.

## How it works

1. **Encode.** ViT-Large (DUSt3R init) → image tokens `F_t`.
2. **Interact.** Two intertwined ViT-Base decoders with **3D
   hierarchical RoPE**:
   - Query = image tokens + learnable pose token `z_t`.
   - Key/value = pointer features `m_i` at positions `P_i`.
   - Image token position for RoPE = patch-averaged **previous-frame**
     global pointmap coordinate.
3. **Decode.** Two DPT heads (self-frame + world-frame pointmaps with
   confidences) + MLP pose head.
4. **Extract new pointers.** `P_new(u,v) = patch-avg(X̂^global_t)`,
   `M_new = MLP(F_t, F'_t) + LightViT(X̂^global_t)`.
5. **Fuse.** Nearest-neighbor in Euclidean distance; below changing
   threshold δ → average positions + features; else add as new pointer.
   Threshold is adjusted to keep pointer distribution spatially uniform.

## 3D Hierarchical RoPE (3DHPE)

Extends [RoPE](https://arxiv.org/abs/2104.09864) from 1D token index to
continuous 3D position `(p^x, p^y, p^z)`:

```
θ_t = 10000^{-t/(d_head/2)}
R(n, 3t)   = e^{iθ_t p^x_n}
R(n, 3t+1) = e^{iθ_t p^y_n}
R(n, 3t+2) = e^{iθ_t p^z_n}
```

Hierarchical variant: use h different base frequencies (not just 10000);
average rotated q/k across h scales — handles varying spatial extents.

## Memory fusion mechanism

Per new pointer p^new_j:
- Find nearest neighbor p_i in memory.
- If `d(p_i, p^new_j) < δ_t`: mark for fusion.
- If a memory pointer is nearest neighbor of K new pointers, update via
  averaging both position and feature.
- Else add new pointer.

Ablation (Table 7): without fusion, 7-scenes Acc improves marginally
(0.124 → 0.118) but memory grows unbounded. Fusion is an
efficiency/quality trade-off.

## Training

- Init: DUSt3R weights for encoder + interaction decoders + DPT heads.
- Loss: L2 pose + confidence-weighted L1 pointmap (self + global) with
  scale normalization; metric mode disables normalization.
- Datasets: 14 mixed (ARKitScenes, ScanNet, ScanNet++, CO3Dv2,
  WildRGBD, HyperSim, BlendedMVS, MegaDepth, Waymo, VirtualKITTI2,
  PointOdyssey, Spring, MVS-Synth, OmniObject3D).
- 3 stages: 5-frame 224² → variable aspect max 512 → freeze encoder
  fine-tune on 8-frame.
- 8× A800, 15 days.

## Where it's been applied

- **3D reconstruction:** 7-scenes, NRGBD.
- **Long-sequence 3D reconstruction:** 500-1000-frame 7-scenes,
  400-900-frame NRGBD. Biggest advantage vs [[cut3r]] here.
- **Monocular depth:** NYU-v2, Sintel, Bonn, KITTI.
- **Video depth:** Sintel, Bonn, KITTI (per-sequence + metric-scale).
- **Camera pose:** ScanNet, Sintel, TUM-dynamics — weakest task.
- **vs. SLAM:** MASt3R-SLAM comparison on 7-Scenes seq-01.

## Known limitations

- **Camera pose lags [[cut3r]] and MonST3R-GA.** Growing pointer set
  may inject spatial interference into pose head; authors flag as
  future work.
- **Runtime 0.20 s/frame** at K=26 pointers ~1485 — not real-time.
- **Metric video depth on Sintel weak** (Abs Rel 1.208 vs
  [[cut3r]] 1.029).
- **No downstream integration** (VLA, tracking heads) demonstrated.
- **No head-to-head with [[streamvggt]] / [[vggt]] / [[mapanything]]**.

## Related methods

- **Direct baselines:** [[cut3r]] (fixed implicit state), Spann3R
  (per-frame cache), [[streamvggt]] (KV cache + causal attention).
  Point3R is the geometry-indexed alternative — memory scales with
  explored space, not time.
- **Batch counterparts:** [[vggt]], [[mapanything]],
  [[depth-anything-3]].
- **Ancestor:** [[dust3r]] — Point3R inherits ViT-L encoder + DPT
  heads init + pointmap parameterization.
- **Dynamic-scene sibling:** MonST3R (DUSt3R fine-tune) — Point3R
  handles dynamic natively via pointer overwrite in fusion step.
- **SLAM contrast:** MASt3R-SLAM — compared on same-scene benchmark
  (better geo, lower mem, worse pose, slower).

## Open directions

- Combine pointer memory with a downstream tracker head (analogous to
  [[tapip3d]] using CUT3R features) — pointers are spatially indexed
  so feature lookup by 3D query is natural.
- Bound-runtime variants for real-time SLAM-like use.
- Feed pointer memory into VLA / policy models — spatially organized
  scene features are a natural robot-conditioning representation.
