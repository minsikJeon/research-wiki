---
type: method
title: FSM (Fast Spatial Memory)
status: growing
tags: [4d-reconstruction, novel-view-synthesis, test-time-training, fast-weights, 4dgs, feed-forward, long-context]
sources:
  - "[[ma-2026-fsm]]"
related:
  - "[[lacet]]"
  - "[[lact]]"
  - "[[4d-reconstruction]]"
  - "[[v-dpm]]"
  - "[[4rc]]"
  - "[[any4d]]"
  - "[[point4d]]"
  - "[[vggt]]"
created: 2026-06-24
updated: 2026-06-24
---

# FSM (Fast Spatial Memory)

An end-to-end feed-forward 4D reconstruction model that learns
spatiotemporal representations from long sequences of posed images and
renders novel view-time combinations. Built on [[lacet]] blocks for
O(n) scaling. The first large-scale 4D NVS model powered by
[[test-time-training]].

## One-line summary

Patchify posed images + Plücker ray maps + timestamps → stack of LaCET
blocks → decode to novel view-time via either direct pixel prediction
(LVSM-style) or 4D Gaussian Splatting (LRM-style).

## Inputs / outputs

- **In:** V posed images `{I_j}` with camera intrinsics/extrinsics +
  timestamps.
- **Out:** rendered image at any target (viewpoint, time) combination.

## How it works

### Image tokenization

Each input view j is concatenated along channels: RGB image `I_j`
(3ch) + Plücker ray map `P_j` (6ch) + timestamp map `T_j` (1ch)
→ 10-channel feature map. Patchified into non-overlapping `p×p`
patches, flattened to `10p²`-dim vectors, linearly projected to
D-dimensional token embeddings.

### Backbone

N stacked [[lacet]] blocks. Fast weights carry state across chunks,
enabling the model to process arbitrarily long sequences in a single
forward pass with bounded memory.

### Two decoder variants

**FSM-LVSM** (direct pixel prediction):
- Target view-time query = empty image-token map (zero appearance
  channels) + target camera/time metadata.
- Query tokens concatenated with input tokens, processed jointly
  through backbone.
- Output: LayerNorm → linear projection → sigmoid → RGB patches.
- No explicit scene representation. Target views synthesized
  independently (no cross-target interaction).

**FSM-LRM** (4D Gaussian Splatting):
- Input views copied as virtual views → processed through backbone.
- Output tokens → linear decoder → per-pixel 4D Gaussian primitives
  `g ∈ R²⁰` (position, time, color, scale, rotation, opacity).
- Rendered via tile-based rasterization with deferred backpropagation.
- Explicit 4D scene representation.

### Training

Photometric supervision only: `L = (1/U) Σ [ℓ₂ + 0.5·LPIPS]`.
Pretrained on 7 datasets (~233M frames): RealEstate10K, DL3DV,
PointOdyssey, Spring, Multi-Cam Video, DynamicReplica, Stereo4D.
Long-context curriculum: resolution 128→256, temporal span 128→256,
dynamic input view count.

## Headline results

### 4D NVS — Stereo4D (256×256)

| | PSNR↑ | LPIPS↓ | SSIM↑ |
|---|---|---|---|
| FSM-LVSM | **32.16** | **0.043** | **0.931** |
| FSM-LRM | 27.19 | 0.147 | 0.876 |
| MoVieS | 27.29 | 0.114 | 0.888 |
| 4DGT | 24.62 | 0.102 | 0.785 |

### 3D NVS — DL3DV (256×256, static scenes)

FSM-LVSM: 26.69 / 0.091 / 0.846 — competitive with dedicated static
methods (tttLVSM 26.90, tttLRM 25.07) despite being a dynamic model.

## Known limitations

- Assumes posed inputs — no joint camera estimation.
- 256×256 resolution only in current experiments.
- Rendering-only supervision — no geometric constraints (depth,
  correspondence, flow).
- Not tested on very long sequences (>136 frames) despite
  architectural support.

## Related methods

- **[[lacet]] / [[lact]]** — the backbone block. FSM is the first
  downstream model to use LaCET.
- **[[v-dpm]]** — another 4D NVS method; fine-tunes VGGT with full
  attention (O(n²)). FSM achieves O(n) via TTT.
- **[[4rc]]** — encode-once query-anywhere 4D; uses full attention.
- **[[point4d]]** — long-range 4D via chunk chaining; focuses on 3D
  tracking rather than NVS.
- **4D-LRM (not ingested)** — FSM-LRM follows its Gaussian
  parameterization.
- **LVSM (not ingested)** — FSM-LVSM follows its direct-pixel decoding
  design.
- **tttLRM / tttLVSM** from [[zhang-2025-lact]] — direct ancestors of
  FSM-LRM and FSM-LVSM; FSM adds temporal conditioning + elastic
  consolidation + multi-dataset 4D pretraining.
