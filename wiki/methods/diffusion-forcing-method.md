---
type: method
title: Diffusion Forcing / Causal Diffusion Forcing (CDF)
status: growing
tags: [sequence-modeling, diffusion, generative-model, video, planning, training-paradigm]
sources:
  - "[[chen-2024-diffusion-forcing]]"
related:
  - "[[diffusion-forcing]]"
  - "[[pi-r-squared]]"
  - "[[action-chunking]]"
  - "[[flow-matching]]"
created: 2026-06-08
updated: 2026-06-08
---

# Diffusion Forcing (DF) / Causal Diffusion Forcing (CDF)

Sequence-modeling training paradigm that **decouples noise level from
time step**. Each token in a sequence gets an independent random noise
level during training; at inference, an arbitrary per-token noise
schedule (a 2D `M × T` grid) is prescribed.

## One-line summary

Train a diffusion model to denoise any combination of variably-noised
tokens; sample with a prescribed 2D noise-level grid that interpolates
between teacher-forced AR generation, full-sequence diffusion, sliding
windows, or anything in between — all with a single trained model.

## Inputs / outputs

- **Training in:** sequence `x_{1:T}`. Internal: per-token noise levels
  `k_t ∼ Unif{0, ..., K}`.
- **Training out:** model `ε_θ(z_{t−1}, x_t^{k_t}, k_t)` (causal RNN
  variant) or `ε_θ(x_{1:T}^{k_{1:T}}, k_{1:T})` (transformer variant)
  predicting per-token noise.
- **Inference in:** prescribed noise grid `K ∈ [0, K]^{M × T}`,
  observation prefix.
- **Inference out:** clean sequence `x_{1:T}` after iterating rows of
  the grid.

## How it works

### Two-axis view of masking (§3.1 of source)

- *Masking along time axis* = teacher-forced next-token prediction.
- *Masking along noise axis* (per-token, same level across time) =
  full-sequence diffusion.
- *Two-axis masking* (different noise level per token) = **Diffusion
  Forcing**.

### Training (Algorithm 1)

```
for each minibatch:
    sample x_{1:T} from data
    for t = 1, ..., T:
        sample k_t ∼ Unif{0, ..., K}
        x_t^{k_t} = ForwardDiffuse(x_t, k_t)
    forward through model (RNN: z_t = f_θ(z_{t-1}, x_t^{k_t}, k_t))
    compute per-token MSE loss between predicted and true noise
    backprop
```

Every token contributes a loss term at its sampled level. No teacher
forcing, no shared-level constraint.

### Sampling (Algorithm 2)

Prescribe a 2D grid `K_{m, t}` with columns = timesteps, rows = denoise
substeps. Initialize all tokens at row `M` to pure noise. Iterate rows
top→bottom:

```
for m = M − 1 down to 0:
    for t = 1, ..., T:
        z_t^new = f_θ(z_{t-1}, x_t, K_{m+1, t})
        x_t^new = ReverseStep(x_t, K_{m, t})  # noise level moves toward target
return x_{1:T}
```

The 2D grid is the deployment-time degree of freedom:

| Grid pattern | Equivalent behavior |
|---|---|
| `K_{m, t}` constant in `t`, monotone in `m` | Full-sequence diffusion |
| `K_{m, t}` step-down: 0 for `t < τ_m`, K otherwise | Teacher-forced AR rollout |
| `K_{m, t}` linear ramp in `t` (clean at front, noise at back), shifting `m` | **Streaming diffusion** |
| `K_{m, t}` staircase: clean prefix + ramp + pure noise tail | **πR² schedule** |

### CDF (causal architecture)

For sequential / video / planning data, use a causal architecture
(RNN, causal transformer). `x_t^{k_t}` depends only on `(z_{t-1}, k_t)`
— enforces temporal causality. RNN latent `z_t` doubles as a Bayesian
filter posterior in the `k_t = 0` limit and as a generative prior in
the `k_t = K` limit.

### Why it scales rollouts beyond training horizon

For high-dim continuous sequences (video), standard AR diverges past
the training horizon because tiny per-step errors compound. CDF stays
stable by re-noising past tokens to a small noise level
`0 < k < K` before continuing — adds a regularizing perturbation that
absorbs accumulated drift.

### Why it admits long-horizon guidance

Guidance gradients can propagate through future tokens (which are still
partially noised) into past tokens (which are also partially noised)
— the causality of CDF means past tokens never lose all uncertainty
prematurely, so the guidance signal can flow backward.

## Where it's been applied

- **Video prediction** (Minecraft, DMLab, real video) — stable
  beyond-horizon rollouts.
- **Maze planning** (model-based RL) — long-horizon guided sampling
  beats full-sequence guidance.
- **Visual imitation learning** — robust to obs noise.
- **Time-series forecasting** — competitive with specialized models.
- **Robotics (πR²)** — first practical large-VLA deployment;
  staircase schedule for action chunks.

## Known limitations

- **Compute overhead vs. teacher forcing.** Each training token costs
  one diffusion sample. Sampling iterates a 2D grid (`M × T` denoising
  steps per chunk in the worst case, though many configurations are
  much cheaper).
- **Grid design is heuristic.** No principled way to choose `K_{m, t}`
  for a given downstream task; different patterns work for different
  problems.
- **LLM-scale CDF unproven.** Original work is small-scale; whether
  the paradigm scales to large LMs is open.
- **Continuous data only.** Discrete-token domains (language) need
  variants like AR-Diffusion.

## Related methods

- **Concept page:** [[diffusion-forcing]] (cross-source pattern; this
  page is the algorithmic recipe).
- **First robotics deployment:** [[pi-r-squared]] /
  [[anon-2026-pi-r-squared]].
- **Closest concurrent:** AR-Diffusion (linearly-dependent per-token
  noise for text), Streaming Diffusion (linearly increasing τ for
  continuous control).
- **Mechanism cousins:** Flow Matching (continuous-time
  interpolation; can also be formulated per-position),
  [[test-time-training]] (per-scene fast-weight model; different
  mechanism, similar "training is dynamic" instinct).
