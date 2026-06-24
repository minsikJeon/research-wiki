---
type: source
source_type: paper
title: "Fast Spatial Memory with Elastic Test-Time Training"
authors:
  - Ma, Ziqiao
  - Yu, Xueyang
  - Zhen, Haoyu
  - Yang, Yuncong
  - Chai, Joyce
  - Gan, Chuang
year: 2026
venue: arXiv (cs.CV)
url: https://arxiv.org/abs/2604.07350
raw_path: papers/2604.07350v1.pdf
status: ingested
tags: [test-time-training, fast-weights, elastic-weight-consolidation, 4d-reconstruction, novel-view-synthesis, long-context, 4dgs, chunk-wise]
related:
  - "[[zhang-2025-lact]]"
  - "[[test-time-training]]"
  - "[[lact]]"
  - "[[lacet]]"
  - "[[fsm]]"
  - "[[4d-reconstruction]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[cmp-3d-4d-reconstruction]]"
created: 2026-06-24
updated: 2026-06-24
---

# Fast Spatial Memory with Elastic Test-Time Training

## TL;DR

[[lact|LaCT]] enables efficient large-chunk test-time training, but its
fully plastic fast-weight updates drift when processing multiple chunks,
causing overfitting and catastrophic forgetting. This paper introduces
**LaCET** (Large-Chunk Elastic Test-Time Training), which stabilizes
LaCT with an Elastic Weight Consolidation (EWC) penalty — maintaining
dual parameter sets (stable *anchor weights* + adapted *fast weights*)
with a Fisher-weighted spring pulling critical parameters back. Built
on LaCET, **Fast Spatial Memory (FSM)** is the first large-scale
feed-forward 4D reconstruction model that ingests long sequences of
posed images and renders novel view-time combinations, with two decoder
variants (LVSM-style direct pixels, LRM-style 4D Gaussian Splatting).

## Why it matters

This paper solves a specific failure mode of [[lact]] that prevented it
from scaling to long 4D sequences: multi-chunk fast-weight drift. The
fix — elastic consolidation — is borrowed from continual learning
(Kirkpatrick et al. 2017 EWC) and adapted to the streaming TTT regime.
For the wiki's [[test-time-training]] design space, this adds a fifth
pattern: **elastic chunked TTT with consolidation** (Pattern E), which
is the first to explicitly balance plasticity and stability across
chunks. For [[4d-reconstruction]], FSM is the first TTT-based 4D NVS
model at scale, achieving SOTA among feed-forward methods on Stereo4D.

## Key claims

- **Multi-chunk LaCT is worse than single-chunk LaCT** — fully plastic
  fast-weight updates drift uncontrollably, leading to temporal ghosting
  artifacts (Table 1, Fig 4). This is the key motivating finding. (§2.3)
- **EWC consolidation fixes the drift.** Adding a Fisher-weighted
  quadratic penalty after each chunk's update step stabilizes the
  fast-weight trajectory without killing plasticity. (§2.3, Eq 5)
- **Streaming-EMA anchoring is the best policy.** Three anchor update
  policies tested (global / streaming / streaming-EMA); streaming-EMA
  gives genuinely elastic behavior — a low-pass filter over the
  fast-weight trajectory. (§4.1)
- **SI-style Fisher estimator works best empirically** — the
  synaptic-intelligence-like variant (squared update) slightly outperforms
  EWC (squared gradient) and MAS (absolute gradient) in this setting.
  (Table 1)
- **LaCET consistently dominates LaCT under sparse inputs** — when input
  views are sparse in time and space (the practical regime), advantages
  are large and systematic across PSNR/SSIM/LPIPS. (§4.2, Fig 5)
- **LaCET mitigates camera-interpolation shortcuts** — LaCT learns to
  interpolate nearby context frames rather than building true
  spatiotemporal representations; LaCET suppresses this. (§4.2)
- **FSM-LVSM achieves SOTA among feed-forward 4D methods** on Stereo4D:
  32.16 PSNR / 0.043 LPIPS / 0.931 SSIM at 256×256 (Table 3). On
  NVIDIA benchmark: 23.90 / 0.105 / 0.747. (§5.2)
- **FSM-LVSM approaches optimization-based methods** (SoM, MoSca) that
  take 10–45 min/scene, while FSM is feed-forward. (Table 3)

## Methods

### LaCET block (Fig 2, right)

Same structure as [[lact]] — window attention + TTT layer + FFN — with
one addition: a **consolidate** operation after the TTT **update**.

**Update** (same as LaCT, Eq 3):
```
θ'_{c+1} = θ_c − ∇_θ Σᵢ ηᵢ L(f_θ(kᵢ), vᵢ)
```

**Consolidate** (new, Eq 5):
```
θ_{c+1} = θ'_c − λ F_c ⊙ (θ'_c − θ*_c)
```
where `F_c` is a per-parameter Fisher-style importance estimate
maintained as an EMA (Eq 6), `θ*_c` are anchor weights, and `λ` controls
the elastic prior strength.

**Importance estimators** (three variants tested):
- MAS: `|θ'_c − θ_c|` (magnitude of update)
- EWC: `(θ'_c − θ_c)²` (squared gradient)
- SI: `(θ'_c − θ_c) ⊙ (θ'_c − θ*_c)` (update × drift from anchor)

**Anchor update policies:**
- Global: fixed to initialization (too rigid)
- Streaming: reset to current fast weights at each chunk boundary
- Streaming-EMA: `θ* ← βθ* + (1−β)θ` — low-pass filter (best)

### FSM architecture (Fig 2, left; Fig 3)

**Image tokenization:** Input = V posed images. Each image is
concatenated with its Plücker ray map + timestamp map along the channel
dimension → 10-channel per-view feature map → patchified + linearly
projected to D-dimensional tokens.

**Backbone:** N stacked LaCET blocks.

**Two decoder variants:**

1. **FSM-LVSM** (Fig 3a): target view-time query tokens (zero appearance
   + target camera/time metadata) concatenated with input tokens,
   processed jointly. Output tokens → LayerNorm → linear → sigmoid →
   RGB patches. No explicit scene representation.

2. **FSM-LRM** (Fig 3b): input views copied as virtual views → LaCET
   backbone → 4D GS decoder. Per-pixel Gaussian primitives
   `g ∈ R²⁰` (xyz, t, rgb, scale_xyz, scale_t, rotation_left,
   rotation_right, opacity). Tile-based rasterization.

**Training loss** (Eq 8): `L = (1/U) Σ [ℓ₂ + 0.5·LPIPS]`

### Pretraining data (Table 2)

7 datasets: RealEstate10K (80K scenes), DL3DV (80K), PointOdyssey
(131 synthetic dynamic), Spring (37 synthetic dynamic), Multi-Cam Video
(13.6K synthetic dynamic), DynamicReplica (484 real dynamic), Stereo4D
(80K real dynamic). Total ~233M frames. Curriculum: resolution
128→256, temporal span 128→256, dynamic input view count.

## Results

### 4D NVS (Table 3)

| Model | Stereo4D PSNR↑ | LPIPS↓ | SSIM↑ | NVIDIA PSNR↑ | LPIPS↓ | SSIM↑ |
|---|---|---|---|---|---|---|
| SoM (opt, ~10min) | — | — | — | 15.30 | 0.509 | 0.317 |
| MoSca (opt, ~45min) | — | — | — | 21.45 | 0.265 | 0.712 |
| 4DGT | 24.62 | 0.102 | 0.785 | 14.13 | 0.640 | 0.131 |
| MoVieS | 27.29 | 0.114 | 0.888 | 19.16 | 0.315 | 0.514 |
| **FSM-LRM** | 27.19 | 0.147 | 0.876 | 20.17 | 0.337 | 0.567 |
| **FSM-LVSM** | **32.16** | **0.043** | **0.931** | **23.90** | **0.105** | **0.747** |

### 3D NVS on DL3DV (Table 4)

FSM-LVSM (256×256): 26.69 PSNR / 0.091 LPIPS / 0.846 SSIM. Competitive
with static models like tttLRM (25.07) and tttLVSM (26.90) at similar
resolution, despite being designed for 4D.

### Test-time scaling (Fig 5)

LaCET (4 chunks) with streaming-EMA consistently outperforms LaCT
(1 chunk and 4 chunks) under both sparse and continuous view sampling,
with the gap widening as input density increases.

## Limitations / open questions

- **Pose estimation in dynamic scenes** — FSM assumes posed inputs;
  jointly estimating camera intrinsics and poses in dynamic scenes
  remains open. (§7)
- **Geometric faithfulness** — NVS alone doesn't ensure geometric
  consistency; the authors acknowledge rendering-only supervision may
  not be sufficient and suggest adding depth, correspondence, or optical
  flow supervision. (§7)
- **Resolution** — evaluated at 256×256 (lowest among baselines) for
  fair comparison; higher-resolution scaling not yet demonstrated.
- **No long-video 4D eval** — unlike [[anon-2026-point4d]] which tests
  200+ frame sequences, FSM evaluates on 136-frame Stereo4D clips.
  Long-sequence generalization is claimed architecturally but not
  empirically validated at extreme lengths.
- **Compute budget** — training on 8 H100s for ablations; full model
  bootstrapped from DL3DV-pretrained LaCT backbone.

## Connections

- **[[zhang-2025-lact]] / [[lact]]** — FSM is the first downstream model
  to extend LaCT. The finding that multi-chunk LaCT degrades (Table 1)
  is a concrete failure mode the LaCT paper didn't surface (it mostly
  used single-chunk NVS).
- **[[test-time-training]]** — LaCET adds Pattern E (elastic chunked TTT)
  to the wiki's A–D taxonomy. The consolidation idea is orthogonal to
  the chunk-size / optimizer choices in LaCT and could in principle be
  applied to [[loger]] or [[vgg-t3]].
- **[[4d-reconstruction]]** — FSM is the first TTT-based entry in the
  4D reconstruction comparison. Unlike V-DPM/Any4D/4RC (which use full
  attention), FSM achieves O(n) scaling via TTT state.
- **[[anon-2026-point4d]]** — Point4D and FSM address the same long-video
  4D problem from different angles: Point4D chains 3D-query chunks with
  Sim(3) alignment; FSM carries TTT state across chunks. FSM focuses on
  NVS; Point4D focuses on 3D tracking.
- **4D-LRM (not ingested)** — FSM-LRM follows its Gaussian
  parameterization. tttLRM and tttLVSM from [[zhang-2025-lact]] are the
  direct ancestors of FSM-LRM and FSM-LVSM respectively.
- **Continual learning / EWC** — Kirkpatrick et al. (2017) is the
  conceptual origin. The reinterpretation from "task A → task B" to
  "chunk c → chunk c+1" is the paper's main algorithmic insight.

## Citation

Ma, Z., Yu, X., Zhen, H., Yang, Y., Chai, J., & Gan, C. (2026).
*Fast Spatial Memory with Elastic Test-Time Training.*
arXiv:2604.07350. https://arxiv.org/abs/2604.07350
