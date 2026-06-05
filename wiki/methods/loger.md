---
type: method
title: LoGeR (Long-Context Geometric Reconstruction)
status: stub
tags: [3d-reconstruction, long-context, hybrid-memory, sliding-window-attention, test-time-training, chunk-wise, feed-forward, dense-pointmap]
sources:
  - "[[zhang-2026-loger]]"
related:
  - "[[vggt]]"
  - "[[cut3r]]"
  - "[[test-time-training]]"
  - "[[trajectory-chaining]]"
  - "[[feed-forward-3d-reconstruction]]"
created: 2026-05-29
updated: 2026-05-29
---

# LoGeR (Long-Context Geometric Reconstruction)

Chunk-wise feed-forward dense 3D reconstruction with **hybrid memory**:
non-parametric SWA (sliding window attention) across adjacent chunks
for lossless local detail + parametric TTT (test-time training) layer
for compressed global context. Trained on 128-frame sequences,
generalizes to 1k+ inference; `LoGeR*` variant scales to 19k frames
with feedforward SIM(3) alignment.

## One-line summary

π³-style ViT backbone with each block doing **frame-wise self-attn +
(sparse) SWA across `C_{m-1} ∪ C_m` + chunk-wise TTT fast-weight
apply/update + bidirectional within-chunk attn**. Hybrid memory
decouples local fidelity (SWA) from global coordinate-frame integrity
(TTT).

## Inputs / outputs

- **In:** video stream `V`, partitioned into chunks `C_1, ..., C_M`.
  Each chunk is `n` frames (default `n=16`, varying).
- **Out:** per-frame local pointmap `P_i ∈ R^{H×W×3}` and camera pose
  `c_i ∈ SE(3)`.
- **State carried between chunks:** TTT fast weights `W_m` (global
  compressed memory); SWA only needs the previous chunk's frame-attn
  output tokens.

## How it works (per block)

1. **Per-frame self-attn** — spatial features per frame, independent.
2. **Sparse SWA over `C_{m-1} ∪ C_m`** — lossless local memory
   between adjacent chunks. Inserted at only **4 layers** of the
   network (not every block) for compute-boundedness.
3. **Chunk-wise TTT layer:**
   - **Apply:** `H̃_C = H_C + f_{W_m}(LN(H_C))`.
   - **Update:** `W_{m+1} = U(W_m; H_C)` (gradient-based self-
     supervised update on chunk content).
   - Pre-norm inside TTT for streaming stability.
4. **Bidirectional within-chunk attn** — full attention *within*
   `C_m` only. Bounded compute. Preserves multi-frame reasoning power
   inherited from π³.

## `LoGeR*` add-on (long-sequence stabilization)

For sequences > 1k frames where memory alone still drifts:

- Estimate rigid SE(3) alignment `A_m` between current chunk and
  aligned-previous-chunk using overlap frames.
- Apply `A_m` to all frames in `C_m`.
- Done feed-forward in both train and inference.

## Heads + loss

- Pointmap decoder + camera pose decoder (lightweight, π³-style).
- Loss: scale-invariant local pointmap `L_local` + affine-invariant
  pose `L_pose` + global pointmap `L_global` (world-coords
  consistency).
- `L_total = L_local + L_pose + λ_global · L_global`.

## Curriculum (critical)

Three stages:

1. **48 frames, 4 chunks** — small.
2. **48 frames, 12 chunks** — same total, more chunks. Forces
   reliance to shift from SWA to TTT (because each chunk shrinks).
3. **128 frames, 20 chunks** — full scale. Needs H200 GPUs.

`LoGeR*` initializes from stage-1 then runs through 2-3.

## Where it's been applied

- **KITTI 11 sequences** (single-trajectory feed-forward
  reconstruction): ATE 18.65m avg (vs TTT3R 72.86).
- **VBR** (Brizi 2024, repurposed): 8.8k–18.8k frames, 11.5 km. 55.2%
  relative ATE improvement.
- **ScanNet, TUM-Dynamics, Sintel** (short-context cross-checks).
- **7scenes 3D reconstruction** (qualitative).

## Known limitations

- Periodic state reset + `LoGeR*` alignment needed for >1k frames —
  not fully automatic.
- TartanAirV2 (large-scale outdoor) heavily up-weighted; data wall
  binding.
- TTT optimization brittle without curriculum.
- Training cost: 4 days × 32 H100+H200 GPUs.
- Chunked baseline (`Pi3-Chunk`) already captures a meaningful share
  of LoGeR's gains; hybrid memory adds on top but is not the only
  active ingredient.
- Short-context gains modest; LoGeR is for the long regime.

## Related methods

- **Backbone:** π³ [Wang et al. 2026] — successor to [[vggt]].
- **Mechanism shared with [[vgg-t3]]:** [[test-time-training]] —
  but applied to *recurrent chunk-wise* state here (VGG-T3 uses TTT
  for *global offline* compression).
- **TTT3R** [Chen et al. 2026] — main autoregressive baseline.
  Frame-wise vs LoGeR's chunk-wise + hybrid.
- **Chunked / efficient-attention competitors:** **FastVGGT**
  (token merging), **SparseVGGT** (block-sparse), **VGGT-Long**,
  **VGGT-SLAM**, **InfiniteVGGT**, **StreamVGGT**, **Stream3R**.
- **Recurrent competitors:** [[cut3r]], **MUSt3R**, **Point3R**,
  **Long3R**, [[mapanything]].
- **Optimization-based competitors:** DROID-SLAM, DPV-SLAM, DPV-SLAM++.
- **Hybrid memory architectural ancestors:** Longformer, BigBird
  (SWA in LMs); Jamba (SSM + attention hybrid).
- **For the user's planning note:** LoGeR is the closest existing
  instance of **Caricature 3 (slow planner + fast controller)**
  applied to perception — the planner is the bidirectional chunk
  pass + TTT update; the controller is implicit in how downstream
  consumers query the predictions. A natural extension is making the
  controller explicit (per-frame net riding on top of LoGeR's
  chunk-paced state).
