---
type: method
title: LaCT (Large-Chunk Test-Time Training)
status: growing
tags: [test-time-training, fast-weights, long-context, hybrid-architecture, muon-optimizer, sequence-modeling, video-diffusion]
sources:
  - "[[zhang-2025-lact]]"
related:
  - "[[test-time-training]]"
  - "[[sun-2024-ttt]]"
  - "[[loger]]"
  - "[[vgg-t3]]"
  - "[[cut3r]]"
  - "[[diffusion-forcing]]"
  - "[[dfot]]"
created: 2026-06-12
updated: 2026-06-12
---

# LaCT (Large-Chunk Test-Time Training)

A sequence-modeling block that combines local **window attention** with
a **large-chunk TTT layer** (chunk size 2K–1M tokens) whose fast weights
are a SwiGLU MLP. The single design change vs prior [[test-time-training|TTT]]
methods — radically larger chunks — unlocks GPU utilization gains
(5% → 70%), nonlinear fast weights up to 40% of model parameters, and
test-time optimization with Muon.

## One-line summary

Treat the inner TTT optimization as a large-batch SGD-like step (one
update per chunk of thousands of tokens), wrap a window-attention layer
around it for local structure, and put a feed-forward on top.

## Inputs / outputs

- **In:** sequence of tokens `x_1, ..., x_N`, projected to per-token
  `(q_i, k_i, v_i)`.
- **Out:** sequence `o_1, ..., o_N`.
- **State:** fast weights `W = {W_1, W_2, W_3}` of a SwiGLU MLP,
  carried across chunks. Reset semantics depend on task.

## How it works

### LaCT block (Fig 2)

```
Token sequence
  ↓
Window Attention (causal or bidirectional, intra-chunk local)  +  residual
  ↓
Split into large chunks (2K–1M tokens each)
  ↓
For each chunk:
    g = ∇_W Σ_i η_i L(f_W(k_i), v_i)
    W ← L2Normalize(W − Muon(g))         # or vanilla GD / momentum
    o_i = f_W(q_i) for all q_i in chunk
  ↓
Feed-Forward  +  residual
```

### Fast-weight network (Eq 6)

`f_W(x) = W₂ [SiLU(W₁ x) ∘ (W₃ x)]`. SwiGLU MLP, no bias.

### Loss (Eq 7)

`L(f_W(k_i), v_i) = − f_W(k_i)^T v_i` — dot-product attention loss
(maximize inner product between predicted output and value).

### Update orders → equivalent attention masks (Fig 3)

| Apply-then-Update | Mask equivalent |
|---|---|
| Apply only (chunk = full sequence) | Full attention |
| Update + apply alternately, same chunk | Block-wise causal |
| Apply then update | Shifted block-wise causal (no leakage within chunk) |
| Update on subset, apply on all | Strided block-wise causal |

This menu lets a single LaCT layer mimic different causality structures
just by changing the per-chunk update schedule.

### Window attention pairing

The TTT layer treats tokens within a chunk as **a set** (no order). To
recover intra-chunk locality, the block prepends a window attention layer
(quadratic but small) before the TTT layer. Inspired by GAU, BASED,
InfiniAttention.

### Muon for test-time updates (Eq 9)

```
Muon(g) ≃ U V^T  where g = U S V^T   (SVD)
weight-update(W, g) = L2Normalize(W − Muon(g))
```

Spectral-norm-normalized gradient → better-conditioned online steps. In
LaCT's experiments, Muon clearly beats Momentum and vanilla GD.

## Where it's been applied

All in [[zhang-2025-lact]]:

| Task | Data | Result |
|---|---|---|
| Novel view synthesis | Objaverse + DL3DV (1.3B model) | beats 3DGS up to 128 input views @ 960×536 |
| Language modeling | Long-Data-Collections (760M / 3B model) | outperforms GLA-SWA, DeltaNet-SWA at 32K |
| AR video diffusion | Internal video corpus (Wan 2.1 14B fine-tuned) | consistent 56K-token videos |

## Known limitations

- **No rotational invariance** — RoPE-style invariance enjoyed by softmax
  attention doesn't transfer to SwiGLU fast weights.
- **Chunk size is empirical** — no principled selector across modalities.
- **No unposed 3D reconstruction tested** — NVS used input camera poses.
- **No FVD on video diffusion** — only validation denoising loss as scaling
  metric.

## Related methods

- **Direct precursor:** [[sun-2024-ttt]] — original TTT-Linear / TTT-MLP,
  which LaCT extends with larger chunks + nonlinear state + Muon.
- **Same-paradigm 3D applications:** [[loger]] (per-chunk TTT + SWA for 3D
  reconstruction), [[vgg-t3]] (per-scene TTT for offline 3D), TTT3R
  (per-frame TTT).
- **Streaming 3D contrast:** [[cut3r]] — uses a fixed-size **token bank**
  updated by attention rather than fast weights updated by gradient
  descent. LaCT is the test-time-trained analogue of CUT3R's memory.
- **Diffusion-forcing cousin:** LaCT's AR video diffusion uses the same
  noisy/clean chunk interleaving as [[chen-2024-diffusion-forcing]] and
  [[dfot]]; one of the two leading recipes for autoregressive video
  diffusion at scale.
- **Concurrent: InfiniAttention** (delta-rule chunk recurrence) — similar
  structure, less expressive update rule.
- **Concurrent: Titans, TTT-Linear+** — TTT variants with different
  inner models / optimizers.
