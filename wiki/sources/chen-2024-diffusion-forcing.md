---
type: source
source_type: paper
title: "Diffusion Forcing: Next-token Prediction Meets Full-Sequence Diffusion"
authors:
  - Chen, Boyuan
  - Marti Monso, Diego
  - Du, Yilun
  - Simchowitz, Max
  - Tedrake, Russ
  - Sitzmann, Vincent
year: 2024
venue: NeurIPS 2024
url: https://arxiv.org/abs/2407.01392
raw_path: papers/2407.01392v4 (1).pdf
status: ingested
tags: [sequence-modeling, diffusion, generative-model, video, planning, autoregressive, training-paradigm, mit-csail]
sources: []
related:
  - "[[diffusion-forcing]]"
  - "[[anon-2026-pi-r-squared]]"
  - "[[action-chunking]]"
  - "[[flow-matching]]"
  - "[[russ-tedrake]]"
  - "[[mit-csail]]"
created: 2026-06-08
updated: 2026-06-08
---

# Diffusion Forcing: Next-token Prediction Meets Full-Sequence Diffusion

## TL;DR

A training paradigm that **assigns each sequence token an independent
random noise level** during training and samples them via arbitrary
per-token noise schedules at inference. Bridges teacher-forced
next-token prediction (variable-length, causal) and full-sequence
diffusion (guidable, multi-step). Causal Diffusion Forcing (CDF) is the
causal-architecture instantiation that powers all subsequent work
([[anon-2026-pi-r-squared]] is the first robotics deployment).

## Why it matters

This is the **mechanism page** for diffusion forcing. Two reasons it's
load-bearing in this wiki:

- **Underlying paradigm for πR².** Per-position noise schedules in
  action chunks (πR²'s staircase) are direct instances of CDF sampling.
  Without this paper there's no πR².
- **Bridges training paradigms.** Maps cleanly onto the
  [[train-inference-mismatch]] cross-cutting concept: standard
  full-sequence diffusion *requires* all tokens to share one noise level
  at inference (mismatch with streaming use); CDF removes that
  constraint at training time.

## Key claims

- **Noising = partial masking, along two axes.** Standard masking
  removes tokens along the *time* axis (teacher forcing); standard
  full-sequence diffusion masks along the *noise* axis with the same
  level for all tokens. Diffusion Forcing **unifies both**: each token
  has its own noise level `k_t`, ranging from 0 (clean) to K (pure
  noise). The model learns to "unmask" arbitrary subsets.
- **Training:** for each token in the sequence, sample
  `k_t ∼ Unif{0,...,K}` independently; noise the token to level `k_t`;
  predict noise via MSE — standard diffusion loss applied per token.
  No teacher forcing, no shared-level constraint.
- **Sampling:** prescribe an `M × T` 2D grid of noise levels `K_{m,t}`
  — columns are timesteps, rows index a denoising trajectory. Iterate
  rows top→bottom; at each row, denoise tokens column-by-column to the
  row's prescribed level. **The grid `K` is the inference-time degree
  of freedom** — change `K` to get different behaviors with the same
  trained model.
- **Causal Diffusion Forcing (CDF)** — instantiate with a causal
  architecture (RNN/causal transformer). `x_t^{k_t}` depends only on
  past tokens. RNN latent `z_t ∼ p(z_t | z_{t-1}, x_t^{k_t}, k_t)`
  unifies Bayesian-filtering latents (zero noise = posterior; full
  noise = prior) and diffusion.
- **Theorem (informal):** training Diffusion Forcing maximizes a lower
  bound on log-likelihoods of **all subsequences** of tokens
  observed at training time, under appropriate conditions.
- **Stabilizes long autoregressive rollouts.** For high-dim continuous
  sequences (video), standard AR diverges past training horizon. CDF
  stably rolls out by using *slightly noised* past tokens as anchors,
  not fully clean ones — small noise level acts as regularization.
- **Long-horizon guidance via Monte Carlo Guidance (MCG).** Because
  future tokens can be diffused without fully diffusing the past,
  guidance gradients can propagate backward in time while respecting
  causality.

## Methods

### Setup

Sequence `x_{1:T}`, noise levels `k_{1:T} ∈ [0, K]^T` (one per token).
Latent states `z_t` for the recurrent variant.

### Training (Algorithm 1)

```
loop:
    sample trajectory x_{1:T}
    for t = 1, ..., T:
        sample k_t ∼ Unif{0, ..., K}
        x_t^{k_t} ← ForwardDiffuse(x_t, k_t)
        z_t ∼ p_θ(z_t | z_{t-1}, x_t^{k_t}, k_t)
        ε̂_t = ε_θ(z_{t-1}, x_t^{k_t}, k_t)
    L = MSE(ε̂_{1:T}, ε_{1:T})
    backprop
```

### Sampling (Algorithm 2)

Given a 2D noise schedule `K ∈ [K]^{M × T}` (one column per timestep,
rows index denoising substeps):

```
init x_{1:T} ∼ N(0, I)   # row m = M, all pure noise
for m = M − 1 down to 0:
    for t = 1, ..., T:
        z_t^new ∼ p_θ(z_t | z_{t-1}, x_t, K_{m+1,t})
        x_t^new ← reverse step toward K_{m,t}
    add long-horizon guidance to x_{1:H}
return x_{1:T}
```

### Theorem 3.1 (informal)

Optimizing the training objective at uniform `k_t` sampling provably
maximizes a lower bound on the joint likelihood `p_θ(x_{1:T})` for
**all** sequences of noise levels simultaneously, under standard
diffusion-training conditions. The model learns one parameter set that
serves the full family of sampling schedules.

## Results (headline)

The paper evaluates on four domains:

- **Video prediction (Minecraft, DMLab, real video).** CDF stably rolls
  out past training horizon; teacher-forced + sliding-window baselines
  diverge.
- **Maze planning (model-based RL).** MCG-guided CDF beats guided
  full-sequence diffusion on long-horizon planning tasks.
- **Visual imitation learning** (small-scale): CDF policies more
  robust to observation noise than teacher-forced.
- **Time-series prediction:** competitive with specialized baselines.

(Detailed numbers in Sec. 4 of paper, not summarized here.)

## Limitations / open questions

- **Compute overhead vs. teacher forcing.** Training samples noise per
  token; sampling iterates a 2D grid. Roughly K× full-sequence
  diffusion's training cost, depending on K.
- **Scaling.** Original experiments are small-scale; large-LLM-scale
  CDF was an open question at publication and remains so.
- **Choice of sampling grid `K`** is heuristic — different grids give
  different behaviors (stable rollout, planning, etc.) but a principled
  selector is missing.
- **Continuous-only.** Diffusion lives in continuous space; mapping to
  discrete-token domains (language) is non-trivial. AR-Diffusion is
  the closest discrete analogue.

## Connections

- **Concept page:** [[diffusion-forcing]] (created in this ingest).
- **Closest non-Diffusion-Forcing related work:** AR-Diffusion (text
  diffusion with linearly-dependent per-token noise) — Diffusion
  Forcing generalizes the noise dependency.
- **First robotics deployment:** [[anon-2026-pi-r-squared]] —
  πR²'s staircase schedule is exactly a CDF sampling grid.
- **Streaming Diffusion (cited as concurrent):** another sampling-grid
  instantiation, linearly increasing τ.
- **Authors / affiliations:** [[mit-csail]] (Chen, Du, Simchowitz,
  Tedrake, Sitzmann), TU Munich (Marti Monso). [[russ-tedrake]] (senior;
  recurring with [[chi-2024-diffusion-policy]]).
- **Concept cross-link:** [[train-inference-mismatch]] — Diffusion
  Forcing is a *training-paradigm* fix for the standard diffusion's
  rigid noise-schedule constraint.

## Citation

Chen, B., Marti Monso, D., Du, Y., Simchowitz, M., Tedrake, R., &
Sitzmann, V. (2024). *Diffusion Forcing: Next-token Prediction Meets
Full-Sequence Diffusion.* NeurIPS 2024. arXiv:2407.01392v4.
https://arxiv.org/abs/2407.01392
