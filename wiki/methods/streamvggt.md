---
type: method
title: StreamVGGT
status: growing
tags: [3d-reconstruction, streaming, causal-attention, kv-cache, distillation, vggt, online, point-map, depth-estimation, camera-pose]
sources:
  - "[[zhuo-2026-stream-vggt]]"
related:
  - "[[vggt]]"
  - "[[cut3r]]"
  - "[[wang-2025-vggt]]"
  - "[[wang-2025-cut3r]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[online-vs-offline-tracking]]"
created: 2026-06-12
updated: 2026-07-02
---

# StreamVGGT

A **causal-attention + KV-cached** restructuring of [[vggt|VGGT]] that
turns the offline dense-view backbone into a streaming model: each new
frame attends only to itself and predecessors, with past frames' KV
cached in memory. Trained by distillation from VGGT.

## One-line summary

VGGT's L=24 alternating (spatial + global) attention layers → spatial +
**temporal causal** attention with per-frame **cached memory tokens**;
fine-tuned by KD from VGGT to inherit geometric priors without dataset
annotations.

## Inputs / outputs

- **In (per new frame):** RGB image `I_T ∈ R^{3×H×W}`.
- **Out (per frame):**
  - Camera pose `g_T ∈ R^9` (quaternion + translation + FoV).
  - Point map `P_T ∈ R^{3×H×W}` in world (= frame-1) coords.
  - Depth map `D_T ∈ R^{H×W}`.
  - Point-track features `y_T ∈ R^{2×M}`.
- **State:** cached memory tokens `M = {M_t}_{t=1..T-1} ∈ R^{T×N×C}`
  (per-frame KV cache).

## How it works

### Three components (Fig 3)

```
Image encoder  →  Spatio-Temporal Decoder  →  Multi-Task Heads
   DINOv2         L=24 layers, alternating         (camera, geometry, track)
                  spatial + temporal-causal attn
```

### Streaming inference (Eq 6)

```
F_T = ImageEncoder(I_T)
G_T = Decoder(CrossAttn(F_T, {M_t}_{t=1..T-1}))
P_T, D_T, g_T = MultiTaskHead(G_T)
M_T = TokenCachedMemory(G_T)
```

Each new frame:
1. Patchify with DINOv2.
2. Cross-attend to all previous frames' cached KV (causal).
3. Spatial attention within frame.
4. Decode heads.
5. Push current frame's KV into the cache.

### Training: distillation from VGGT (§3.3)

VGGT (frozen teacher) provides:
- Pseudo-GT camera, depth, pointmap labels.
- Soft confidences as uncertainty-aware regularization.

Loss (Eq 7): `L = L_camera (Huber) + L_depth (conf-weighted L1+grad) +
L_pmap (conf-weighted L1+grad)`.

950M params fine-tuned from VGGT init for 10 epochs on 4× NVIDIA A800
in 7 days. Multi-domain training corpus of 13 datasets.

### Scalability strategies (§4.6, Table 8)

Linear KV-cache growth → two fixes:
- **Windowed streaming:** partition into fixed-length windows; align via
  predicted camera extrinsics.
- **K-nearest-frames caching:** attend only to K most recent frames.

Both bound memory at modest accuracy cost.

## Where it's been applied

- 7-Scenes, NRGBD, ETH3D — 3D reconstruction. **Outperforms CUT3R.**
- ScanNet, Sintel, TUM-dynamics — camera pose. **Outperforms CUT3R**,
  matches VGGT.
- Sintel, Bonn, KITTI, NYU-v2 — depth estimation. **Outperforms CUT3R**.
- TUM-dynamics — 4D reconstruction. Comparable to CUT3R.

## Known limitations

- **KV cache grows linearly with N** — unlike CUT3R's fixed-size hidden
  state. Mitigated by window streaming or K-nearest caching at some
  accuracy cost.
- **Distillation is necessary** — from-scratch causal training loses
  ~30% accuracy (Table 9). Inherits VGGT's training distribution and
  biases.
- **No dynamic-scene gain over VGGT.** Same static-scene assumption;
  TUM-dynamics is the only dynamic benchmark and results match CUT3R
  rather than dramatically improving on it.

## Related methods

- **Teacher:** [[vggt]] — StreamVGGT inherits its architecture and is
  trained by distilling its outputs.
- **Streaming competitor:** [[cut3r]] — different mechanism (fixed-size
  token bank). StreamVGGT beats CUT3R on reconstruction, camera pose,
  and depth across multiple benchmarks.
- **Streaming peers:** Spann3R (token-addressable spatial memory),
  [[point3r]] (geometry-aligned pointer memory with 3D RoPE).
- **TTT alternative:** [[lact]] — different streaming strategy using
  test-time-trained fast weights rather than an explicit KV cache.
- **For middle-ground 3D-tracker design:** StreamVGGT's cached memory
  tokens *are* a per-frame spatial grid, so TAPIP3D-style feature
  lifting from history is direct — addressing the leap-of-logic in
  using CUT3R's compressed hidden state for query lifting.
