---
type: method
title: DFoT (Diffusion Forcing Transformer)
status: growing
tags: [video-diffusion, diffusion-forcing, transformer, dit, generative-model, training-paradigm, classifier-free-guidance]
sources:
  - "[[song-2025-history-guided-video-diffusion]]"
related:
  - "[[diffusion-forcing]]"
  - "[[diffusion-forcing-method]]"
  - "[[history-guidance]]"
  - "[[chen-2024-diffusion-forcing]]"
  - "[[pi-r-squared]]"
  - "[[training-time-rtc]]"
created: 2026-06-11
updated: 2026-06-11
---

# DFoT (Diffusion Forcing Transformer)

A video diffusion training framework that extends [[diffusion-forcing]]
from causal RNNs to **non-causal DiT transformers**. Trains a single
DiT to denoise sequences with per-frame independent noise levels,
enabling conditioning on any subset of history frames at any noise
level at inference time.

## One-line summary

Train a video DiT with per-frame `k_t ∼ Unif[0, 1]`; at inference, set
`k_t = 0` for any history frames you want to condition on and `k_t > 0`
for frames to generate — one model, any conditioning pattern.

## Inputs / outputs

- **Training in:** clean video `x_{1:T}`. Internal: per-frame noise
  levels `k_T ∼ Unif[0,1]^T`.
- **Training out:** velocity / noise predictor
  `ε_θ(x_T^{k_T}, k_T)` for the full sequence.
- **Inference in:** prescribed mask `(H, G)` (history vs generation
  indices) + history-frame contents (optionally pre-noised) + generation
  noise schedule.
- **Inference out:** generated frames `x_G`.

## How it works

### The "noise as masking" identity (§3)

| `k_t` | Interpretation |
|---|---|
| `0` | clean frame (full conditioning) |
| `(0, 1)` | partially masked — noisy snapshot of original |
| `1` | fully noised = "fully masked" = no information |

This unifies clean-conditioning, dropout-conditioning, and full diffusion
under one parameter. CFG is recovered as a special case (set `k_H = 0`
for conditional, `k_H = 1` for unconditional).

### Training (Eq 4)

```
for each minibatch:
    sample x_{1:T} from data
    for t = 1, ..., T:
        sample k_t ∼ Unif[0, 1]
    noise frames independently: x_t^{k_t} = α_{k_t} x_t + σ_{k_t} ε_t
    forward through DiT with per-frame AdaLN(k_t):
        ε̂_T = ε_θ(x_T^{k_T}, k_T)
    loss = || ε̂_T − ε_T ||²    (computed on ALL T frames)
    backprop
```

Key differences vs Diffusion Forcing (CDF):

| Property | CDF ([[chen-2024-diffusion-forcing]]) | DFoT |
|---|---|---|
| Architecture | causal RNN | non-causal DiT (full attention) |
| Causality | strict | none — any frame attends to any other |
| Use case | streaming video / planning | flexible-history video generation |
| Per-token noise | yes | yes (per-frame) |

### Architecture (Fig 2b)

Standard video DiT block:
- Patchify each frame.
- **AdaLN conditioning per frame** (scale, shift, gate computed from
  embedded `k_t`).
- Full self-attention across all `T × #patches` tokens.
- FFN.

**Same parameter count as standard video DiT.** The only change is
that AdaLN's conditioning `k` becomes a vector `k_T` and is applied
per-frame.

### Sampling

Choose mask `(H, G)`:
- For each step with current noise level `k`:
  - Set `k_t = 0` for `t ∈ H` (history frames stay clean).
  - Set `k_t = k` for `t ∈ G` (generation frames at current step).
- Run DDPM / DDIM update on generation frames only.

Optionally use [[history-guidance]] (HG-v, HG-t, HG-f, HG-tf) for
quality / dynamics / OOD robustness.

### Fine-tuning from existing models

12.5% of scratch-training cost can convert a pretrained Full-Sequence
diffusion model to DFoT. The architectural change (per-frame AdaLN) is
small enough that the pretrained weights largely transfer.

## Why one model serves all conditioning patterns

Theorem 4.1 (informal): training with per-frame `k_t ∼ Unif[0,1]`
optimizes a reweighted ELBO on **all subsequences** simultaneously.
Any inference-time mask pattern is in-distribution.

## Where it's been deployed

All in [[song-2025-history-guided-video-diffusion]]:
- **Kinetics-600** — generic video quality (FVD 4.3, matches industry
  W.A.L.T at far less compute).
- **RealEstate10K** — 862-frame autoregressive rollouts from one image.
- **Minecraft** — long-context generation, OOD-robust.
- **Fruit Swapping** — long-horizon reactive imitation learning
  (combines memory + reactivity scores).

## Known limitations

- **Per-frame independent noise during training** doubles in the per-frame
  AdaLN parameters' workload — minor cost.
- **Guidance scale tuning required.** `ω` and `k_H` are empirically picked
  per task.
- **No discrete-token version.** Continuous-latent only.
- **Causal variant analyzed in App A.5 but not fully evaluated.** For
  streaming applications (online perception / real-time control), the
  causal DFoT variant is the natural pick but evidence is thin.

## Related methods

- **Concept page:** [[diffusion-forcing]] (the cross-source pattern).
- **Foundational paper:** [[chen-2024-diffusion-forcing]] — CDF, causal
  RNN; DFoT is the non-causal DiT extension.
- **History-guidance family:** [[history-guidance]] — the new
  inference-time capability DFoT enables.
- **Cousin in robotics:** [[pi-r-squared]] — same per-position-AdaLN
  recipe applied to action chunks; πR² staircase = special-case `k_T`
  pattern.
- **Cousin in robotics (training-time):** [[training-time-rtc]] —
  per-position AdaLN with `k = 1` for prefix (clean clamp), `k < 1` for
  postfix. DFoT generalizes this to arbitrary per-frame `k_t` for video.
- **Standard CFG:** Ho & Salimans 2022. CFG = DFoT with binary `k_H ∈ {0, 1}`
  and a separate-encoder mask.
