---
type: concept
title: History Guidance (HG)
status: growing
tags: [classifier-free-guidance, diffusion, video-diffusion, generative-model, inference-time-guidance, diffusion-forcing]
sources:
  - "[[song-2025-history-guided-video-diffusion]]"
related:
  - "[[diffusion-forcing]]"
  - "[[dfot]]"
  - "[[chen-2024-diffusion-forcing]]"
  - "[[pi-gdm]]"
  - "[[train-inference-mismatch]]"
created: 2026-06-11
updated: 2026-06-11
---

# History Guidance (HG)

A family of **classifier-free-guidance-style methods** that condition
diffusion sampling on a *variable-length, possibly noisy* history of
past frames. Enabled by [[dfot]]'s noise-as-masking framework. Generalizes
CFG along two axes — **time** (which history windows to compose) and
**frequency** (at what noise level to corrupt history before guiding).

## Definition

For a score function `s_θ(x_G | x_H^{k_H})` that conditions generation
`G` on history `H` corrupted at noise level `k_H`:

- **History Guidance Score (Eq 5):**
  ```
  ∇log p(x_G) + Σ_i ω_i [∇log p(x_G | x_{H_i}^{k_{H_i}}) − ∇log p(x_G)]
  ```
  where each term composes a score conditioned on a different history
  subsequence at a different noise level. The mix `(H_i, k_{H_i}, ω_i)`
  is the deployment-time DoF.

## The four named variants

| Variant | Mechanism | Use case |
|---|---|---|
| **HG-v (Vanilla)** | one history window at `k_H = 0`; CFG with full history vs no history | improve quality + consistency |
| **HG-t (Temporal)** | compose scores from *long* + *short* history windows | balance long-range memory vs short-range reactivity; OOD-robust |
| **HG-f (Fractional)** | guide with history at `k_H ∈ (0, 1)` | low-pass filter on history → preserves dynamics, fixes static-video collapse |
| **HG-tf (Time + Frequency)** | composition along both axes | the comprehensive instance |

## Why each variant

### HG-v: simplest, but trade-off

Higher guidance scale `ω` → higher quality + consistency, but generates
**static videos** at `ω ≥ 3` (model copies the most recent history
frame). Equivalent to over-conditioning.

### HG-f: fix the static-video failure

Guiding with **only low-frequency components** of history (high `k_H`)
preserves consistency without forcing the model to copy details:

```
∇log p(x_G | x_H) + ω · [∇log p(x_G | x_H^{k_H}) − ∇log p(x_G)]
```

High-frequency details (textures, fast motion) are unconstrained → more
dynamic videos. Achieves best FVD on Kinetics-600 (170.4 vs 247.5 SD).

### HG-t: out-of-distribution history

Long histories tend to be OOD (curse of dimensionality). HG-t splits the
long history into shorter, in-distribution subsequences and composes
their scores. Enables 862-frame rollouts and OOD camera-rotation
generation (Fig 7 of source).

### HG-tf: the general framework

Sum scores conditioned on **multiple history subsequences at different
noise levels**. Each component handles a different aspect: short-window
for local consistency, long-window for global memory, low-frequency
for dynamics, etc.

## Why HG is a generalization of CFG

CFG (Ho & Salimans 2022) trades off conditional vs unconditional scores:
```
∇log p(x | c) + ω · [∇log p(x | c) − ∇log p(x)]
```

In HG: the conditioning variable is the *history sequence* itself, and
the "unconditional" score is recovered by setting `k_H = 1` (full noise
on history). CFG is the special case:
- conditional = `k_H = 0` (clean history)
- unconditional = `k_H = 1` (fully noised history)
- binary mask, no compositional mixing

## What's load-bearing

HG works **because of DFoT's flexibility**. Standard video diffusion
models trained with fixed-length history can't compute the scores at
arbitrary `(H_i, k_{H_i})` — the model would be OOD. DFoT's per-frame
`k_t ∼ Unif[0,1]` training is what makes any conditioning pattern
in-distribution at inference.

## Relevance to other guidance mechanisms in this wiki

| Method | Lives at | Closest analogue |
|---|---|---|
| CFG | inference, binary mask | HG-v (as special case) |
| [[pi-gdm]] | inference, linear-measurement guidance | conceptually similar inference-time correction but solves a different problem (inpainting) |
| HG-v | inference, binary mask via noise | strict generalization of CFG |
| HG-f | inference, "low-pass filter" | new — no CFG analogue |
| HG-t | inference, multi-window composition | related to compositional diffusion (Du et al., Liu et al.) |

## Open questions

- **Adaptive `(ω, k_H)` selection.** All variants tune by hand. A
  principled selector based on task-specific quality/dynamics targets
  is open.
- **Causal variant.** Streaming applications need the causal version of
  DFoT for HG to apply online. App A.5 of source sketches it but doesn't
  evaluate.
- **HG for chunked perception/control.** πR²'s staircase schedule is
  essentially HG with a particular `(k_T, ω)` family. A unified
  framework treating πR², TT-RTC, and HG as instances of "noise-as-mask
  guidance" hasn't been written.

## See also

- [[dfot]] — the architecture/training framework that makes HG possible.
- [[song-2025-history-guided-video-diffusion]] — the source paper.
- [[diffusion-forcing]] — the broader noise-as-masking paradigm.
- [[pi-r-squared]] — the chunked-control analogue of HG.
- [[pi-gdm]] — a different style of inference-time guidance.
