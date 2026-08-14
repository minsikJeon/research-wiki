---
type: source
source_type: paper
title: "ZipMap: Linear-Time Stateful 3D Reconstruction via Test-Time Training"
authors: [Jin, Haian; Wu, Rundi; Zhang, Tianyuan; Gao, Ruiqi; Barron, Jonathan T.; Snavely, Noah; Hołyński, Aleksander]
year: 2026
venue: "arXiv preprint (2026)"
url: https://haian-jin.github.io/ZipMap
raw_path: papers/2603.04385v3.pdf
status: ingested
tags: [3d-reconstruction, feed-forward, test-time-training, linear-complexity, streaming, stateful, scalability, novel-view-synthesis, vggt, fast-weights]
related:
  - "[[zipmap]]"
  - "[[test-time-training]]"
  - "[[lact]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[vggt]]"
  - "[[cut3r]]"
  - "[[streamvggt]]"
  - "[[point3r]]"
  - "[[vgg-t3]]"
  - "[[loger]]"
  - "[[cmp-3d-4d-reconstruction]]"
  - "[[haian-jin]]"
  - "[[aleksander-holynski]]"
  - "[[google-deepmind]]"
created: 2026-08-14
updated: 2026-08-14
---

# ZipMap: Linear-Time Stateful 3D Reconstruction via Test-Time Training

## TL;DR

Feed-forward 3D reconstruction (VGGT/π³ family) is quadratic in the number
of input views because it uses global attention. ZipMap replaces that
global attention with a **large-chunk test-time-training (TTT / [[lact|LaCT]])
layer** that compresses the entire image collection into the fast-weights
of a SwiGLU-MLP in a single forward pass → **O(N) bidirectional**
reconstruction that *matches or beats* quadratic VGGT/π³ quality. Bonus:
the fitted fast-weights are a **queryable scene state** — novel-view
point/depth at ~100 FPS independent of N — and the same update rule runs
**streaming** (one view at a time). 700+ frames in <10 s on one H100
(75 FPS), >20× faster than VGGT.

## Why it matters

This is the paper that **merges the two O(N)-3D lines** in this wiki:

1. **TTT-for-3D** ([[vgg-t3]] offline-global, [[loger]] chunk-streaming,
   [[lact]]/[[fsm]] 4D-NVS) — linear cost via fast-weight compression.
2. **Streaming-3D** ([[cut3r]] fixed state, [[streamvggt]] KV-cache,
   [[point3r]] pointer memory) — online per-frame updates.

Prior linear-time methods bought speed by **sacrificing quality** (CUT3R,
TTT3R degrade sharply as N grows — Fig 4). ZipMap is the first to hit
quadratic-quality **and** linear-time **and** a queryable state **and**
a streaming mode, all from one LaCT recipe on a VGGT-style backbone.

**For the user's real-time 3D streaming-tracker project:** ZipMap is the
closest existing system in the design space — linear-time, streaming,
stateful, queryable — and it directly closes the "no streaming eval" gap
flagged earlier against [[v-dpm]]/[[4rc]]. **But it reconstructs geometry,
it does not track**: no temporal correspondence / dynamic scene-flow
output. The LaCT-stateful-backbone recipe is the substrate to build a 3D
streaming *tracker* on; the tracking head + dynamic-correspondence eval is
the open piece ZipMap does not cover. See [[zipmap]] for the design notes.

## Key claims

- **Global attention → large-chunk TTT gives O(N) with no quality loss**
  (Abstract; §3.2). The TTT layer aggregates global info by one gradient
  step over *all* image tokens, so cost is linear, not quadratic in views.
- **Matches/beats quadratic SOTA.** Camera pose (Tab 1–2), point maps
  (Tab 3–4), video/mono depth (Tab 5) — comparable to VGGT [68] and π³
  [76], while *substantially* beating the O(N) baselines CUT3R and TTT3R.
- **>20× faster than VGGT** on long sequences: 700+ frames in <10 s vs
  VGGT's >200 s on one H100; ~3× faster than CUT3R/TTT3R because those
  reconstruct sequentially (low GPU utilization) while ZipMap does one
  parallel forward pass (§4.2, Fig 1).
- **Linear-time methods degrade with N; ZipMap does not** (Fig 4, DL3DV):
  CUT3R/TTT3R error grows sharply as N grows (both scene-scale and
  view-density protocols); ZipMap tracks the quadratic baselines.
- **The fast-weights are a queryable implicit scene state** (§4.4):
  queried at novel ray-map cameras at ~100 FPS *independent of N*, it
  produces pixel-aligned point/depth. Point cloud from state-queries alone
  ≈ point cloud reconstructed from the input images (Fig 5).
- **It infers unseen structure** — extrapolates walls/floors/ground beyond
  observed views (deterministic, so no hallucination of unseen objects),
  evidence the state encodes basic 3D scene priors (§4.4, Fig 5).
- **Extends to streaming** by online per-view fast-weight update (Eq 8,
  §3.3; evaluated Appendix D.5). Main paper focuses on bidirectional.
- **Ablations** (Tab 6): Newton–Schulz orthonormalization and the gated
  output unit are both crucial; dynamic per-token learning rate beats
  fixed global TTT LR (0.1 / 1.0). Removing the reference view (final
  training stage) doesn't clearly help standard benchmarks but improves
  long-sequence accuracy (§4.3, Appendix D.4).

## Methods

### Architecture (§3.1–3.2)

VGGT-style backbone, but **linear**:

- **Tokenize:** frozen DINOv2 patch tokens per image; +1 camera token +4
  register tokens per image. A separate **ray-map** input `T ∈ R^{H×W×9}`
  (ray origin, direction, `r_o × r_d`) with a **query token** lets the
  model be queried at arbitrary target cameras.
- **Backbone:** L=24 identical blocks, each =
  1. **Local window attention** — self-attention *within each view's
     tokens only* (RoPE), captures intra-view spatial structure.
  2. **Global large-chunk TTT layer** ([[lact|LaCT]]) — the linear-scaling
     global-mixing step (below).

### The TTT global layer (§3.2)

Fast-weight function is a SwiGLU-MLP `f_W(x) = W₂[SiLU(W₁x) ∘ (W₃x)]`,
`W = {W₁,W₂,W₃}`. One gradient step over **all** input-view tokens fits it:

- Virtual key–value objective `L(f_W(k_i), v_i) = −f_W(k_i)ᵀv_i`
  (dot-product / associative-memory loss; unrelated to the 3D loss).
- Gradient `g = ∇_W Σ η_i L(f_W(k_i), v_i)`, per-token LR `η_i` from a
  linear layer.
- **Muon-style update:** `Δ = NewtonSchulz(g)`; `Ŵ = ∥W∥·(W−Δ)/∥W−Δ∥`
  (spectral-norm-conditioned step + L2 normalization for stability).
- **Apply:** `o′_i = f_Ŵ(q_i)` — linear-cost analogue of self-attention.
  Same `Ŵ` applies to **target ray-map** query tokens → cross-attention
  analogue at *constant* per-query cost, independent of N.
- Gated output `o_i = RMSNorm(o′_i)·SiLU(W_g o′_i)`.

### Streaming (§3.3)

For a stream `{I₁,I₂,…}`, update fast weights online one view at a time:
`W^(t) ← TTTUpdate(W^(t−1); {k_{t,i}, v_{t,i}})` using the same virtual
KV objective on the current view's tokens only.

### Heads (§3.4)

Four DPT/VGGT-style heads: **camera** (VGGT design, `c_i ∈ R^9` = quat +
translation + 2 intrinsics), **point** (local per-view pointmap `P_i`,
π³-style), **depth** (`D_i` + confidence `Σ_i`; smoother than pointmap,
Σ filters noisy pixels), **query** (target-view RGB `Iᵗ` + depth `Dᵗ` +
`Σᵗ` at a ray-map camera, no explicit scene rep).

### Training (§3.5)

- Losses: `L = L_point + L_depth + 5·L_cam + L_tcolor + L_tdepth`.
  `L_point` scale-invariant local pointmap (ROE solver for scale ŝ);
  `L_depth` conf-weighted L1 + `−α log Σ` (α=0.2, Laplacian NLL);
  `L_cam` L1, affine-invariant after reference-view removal (π³-style);
  `L_tcolor = 10·(MSE + LPIPS)`, `L_tdepth` — **query losses, finetune
  only**; + normal + depth-gradient smoothness.
- Init from VGGT: reuse DINOv2 encoder, init local window attn from VGGT
  frame-wise attn, init a subset of TTT params from VGGT global attn.
- `d=1024`, fast-weight MLP intermediate 2048, state `6d²`/layer;
  **~1.40B params**.
- **64 H100, 3 stages:** 80K iters static + reference view (~5 d) → 40K
  iters +dynamic finetune (~2.5 d) → 60K iters **remove reference view**.
  29 public datasets.

## Results (headline; cross-paper — not a leaderboard)

**Camera pose** (Tab 1–2). Ours matches quadratic, beats linear:

| | Re10K AUC@5↑ | Co3Dv2 AUC@5↑ | Sintel ATE↓ | TUM ATE↓ | ScanNet ATE↓ |
|---|---|---|---|---|---|
| VGGT (O(N²)) | 38.71 | 67.84 | 0.172 | 0.012 | 0.035 |
| π³ (O(N²)) | 63.10 | 57.12 | 0.073 | 0.014 | 0.030 |
| CUT3R (O(N)) | 46.92 | 24.88 | 0.216 | 0.042 | 0.096 |
| TTT3R (O(N)) | 46.37 | 22.61 | 0.204 | 0.028 | 0.065 |
| **ZipMap (O(N))** | **53.34** | **62.46** | **0.132** | **0.012** | **0.034** |

**Point maps** (Tab 3–4): matches VGGT/π³ on DTU/ETH3D/7-Scenes/NRGBD;
beats CUT3R/TTT3R by ~3–4× (e.g. DTU Acc. 1.228 vs CUT3R 5.045).

**Depth** (Tab 5, video): Sintel AbsRel 0.248 / Bonn 0.059 / KITTI 0.057
— beats every O(N) baseline, generally ≥ VGGT (Sintel 0.298). Best O(N)
method; competitive with π³. Monocular: outperforms all on NYU-v2.

**Efficiency** (Fig 1): 700+ frames <10 s (75 FPS) on one H100, VGGT >200 s
(>20×); ~3× faster than CUT3R/TTT3R; **state query ~100 FPS**, independent
of N.

## Limitations / open questions

- **Reconstruction, not tracking.** Outputs geometry (cameras/depth/
  pointmaps) + novel-view synthesis. No temporal point correspondence /
  dynamic scene-flow head — the thing the user's streaming-tracker project
  targets. ZipMap gives the O(N) stateful backbone; the tracking head +
  correspondence eval are not addressed.
- **Deterministic state → no hallucination of unseen content.** Infers
  low-frequency common structure (walls/floors) but cannot invent unseen
  objects (missing sofa, Fig 5). Not a generative scene model.
- **Dynamic scenes handled in recon (Fig 3) but no 4D-tracking eval.**
  Dynamic datasets used in stage-2 finetune; no dense-4D-tracking numbers
  vs [[v-dpm]]/[[4rc]]/[[point4d]].
- **Streaming relegated to appendix** (D.5) — main claims are bidirectional
  batch; streaming quality-vs-latency trade-off under-reported.
- **Chunk / state-size still empirical** (inherited [[lact]] limitation):
  `6d²` state, chunk = all views; no principled selector.
- **Heavy training** — 64 H100 for ~8 GPU-days-equiv across 3 stages, 29
  datasets. Not the "12% of VGGT" cheap-TTT-retrofit that [[vgg-t3]] is;
  ZipMap is a full backbone trained from a VGGT init.

## Connections

- **Builds directly on [[lact]]** ([[zhang-2025-lact]]) — the large-chunk
  TTT block is the load-bearing component. ZipMap = "LaCT applied to
  feed-forward 3D reconstruction." Adds a queryable ray-map path + streaming
  update + the 3D head zoo.
- **Linear-time TTT-for-3D peers:** [[vgg-t3]] (offline-global, cheap VGGT
  retrofit — ZipMap is a full-backbone trained sibling that *also*
  streams), [[loger]] (chunk-streaming, π³ backbone), [[fsm]]/[[lacet]]
  (4D-NVS, elastic consolidation). ZipMap is the [[test-time-training]]
  Pattern-F entry (see that page).
- **Streaming-3D peers it beats:** [[cut3r]] (fixed implicit state),
  [[point3r]] (pointer memory), [[streamvggt]] (KV-cache — grows with N;
  ZipMap's fast-weight state is fixed-size). All are O(N) but ZipMap holds
  quadratic quality while they degrade with N.
- **Quadratic baselines it matches:** [[vggt]] (init + encoder reused),
  π³ (loss + head design borrowed). Answers VGGT's scaling limitation
  head-on.
- **Same TTT substrate as** [[sun-2024-ttt]] (original), one level up in
  chunk size via [[lact]].
- **Authors:** [[haian-jin]] (lead; also LVSM / RayZer / LVSM view-synthesis
  line), [[aleksander-holynski]] (also senior on [[cut3r]] — same author
  now on both the fixed-state and the TTT-state streaming approaches).
  [[google-deepmind]] × Cornell × MIT.
- Adds the **linear-time axis** to [[cmp-3d-4d-reconstruction]] alongside
  [[fsm]].

## Citation

Haian Jin, Rundi Wu, Tianyuan Zhang, Ruiqi Gao, Jonathan T. Barron, Noah
Snavely, Aleksander Hołyński. "ZipMap: Linear-Time Stateful 3D
Reconstruction via Test-Time Training." arXiv:2603.04385v3 [cs.CV], 8 Apr
2026. Google DeepMind, Cornell University, MIT.
Project: https://haian-jin.github.io/ZipMap
