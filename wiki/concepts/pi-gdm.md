---
type: concept
title: ΠGDM (Pseudoinverse-Guided Diffusion Models)
status: growing
tags: [diffusion, flow-matching, inference-time-guidance, inpainting, conditional-generation, vla, real-time-control]
sources:
  - "[[black-2025-rtc]]"
  - "[[black-2025-training-time-rtc]]"
  - "[[anon-2026-pi-r-squared]]"
related:
  - "[[rtc]]"
  - "[[training-time-rtc]]"
  - "[[pi-r-squared]]"
  - "[[flow-matching]]"
  - "[[action-chunking]]"
  - "[[asynchronous-control]]"
  - "[[train-inference-mismatch]]"
created: 2026-06-10
updated: 2026-06-10
---

# ΠGDM (Pseudoinverse-Guided Diffusion Models)

## Definition

An **inference-time guidance mechanism** for diffusion / flow-matching
models that conditions sample generation on an arbitrary linear
measurement `y = A x_0` of the clean signal `x_0`. The denoiser's
prediction is treated as a *measurement* of the true clean sample, and
a closed-form correction is derived by **pseudoinverting the measurement
operator** under a **linearization** of the denoiser around the current
noisy state.

Original paper: *Pseudoinverse-Guided Diffusion Models for Inverse
Problems* — Song et al., ICLR 2023.

## Why it matters in this wiki

ΠGDM is the **engine of inference-time [[rtc]]**. RTC treats the
"prefix" of an action chunk (the `d` actions already committed to the
robot) as a linear inpainting observation, and uses ΠGDM to softly
constrain the new chunk's first `d` positions during denoising. The
whole RTC → [[training-time-rtc]] → [[pi-r-squared]] lineage can be
read as **progressively replacing the ΠGDM machinery with cheaper /
exacter mechanisms**.

## The general recipe

For an observation `y = A · x_0`:

```
1. Run denoiser:         x̂_0 = denoise(x_t, τ)
2. Measurement residual: r = y − A · x̂_0
3. LINEARIZE:            treat x̂_0 as locally linear in x_t       ← Approximation
4. PSEUDOINVERSE:        g = A⁺ · r                                ← Closed-form solve
5. Combine:              v_guided = v_θ + β · g
6. Euler step:           x_{t−Δt} = x_t + Δt · v_guided
```

For **inpainting** (which RTC is): `A` is just a mask, `A⁺` is the same
mask, so step 4 is trivial — the algorithmic name "ΠGDM" persists, but
the pseudoinverse part is degenerate.

## Two approximation sources

The cost of ΠGDM's elegance is **two compounding errors**:

1. **Linearization** (step 3).
   The denoiser `x̂_0(x_t)` is a deep nonlinear network. ΠGDM assumes
   it is locally linear in `x_t` so a closed-form gradient correction
   exists. The lie is small for small corrections, large for large
   ones — i.e., **error scales with the magnitude of the guidance**.

2. **Pseudoinverse** (step 4).
   Trivial for inpainting masks, but for general non-orthogonal `A` it
   adds another approximation. Not load-bearing for RTC.

## Why per-step VJP overhead

Computing the guidance gradient `g = A⁺ · r` requires
`∂x̂_0/∂x_t · A⁺ · r` — a **vector-Jacobian product** through the
denoiser. This must be done **at every denoising step**. For K=4
steps and ~5 ms VJP cost, ΠGDM adds **~20–30 ms per policy call**
on a modern accelerator.

This is the cost [[training-time-rtc]] eliminates by moving the prefix
into the training-time input distribution: no inference-time gradient
needed.

## Why ΠGDM degrades when many dimensions are constrained

Empirically (Fig 3 of [[black-2025-training-time-rtc]]), inference-time
[[rtc]] degrades vs. training-time RTC as the prefix length `d` grows.
Three reasons compound:

| Failure mode | What grows with `d` | Mechanism |
|---|---|---|
| Train/inference mismatch | OOD fraction `d/H` of the chunk | denoiser sees clean tokens next to noisy ones, never seen at train time |
| Linearization error | gradient magnitude `‖g‖` | larger correction pushes `x_t` further out of the linear regime |
| Pseudoinverse error | # constrained dims | for non-trivial `A`, approximation error scales with constrained subspace |

`d = 0` is exact (no guidance); `d = H/2` is half-OOD.

## Where ΠGDM still wins over training-time RTC

- **No retraining required.** ΠGDM is a drop-in for any pretrained
  flow-matching policy. Training-time RTC needs fine-tuning.
- **Arbitrary observation operators.** ΠGDM handles any linear `A`;
  training-time RTC only supports a hard prefix of length `d`.
- **β is the deployment knob.** ΠGDM exposes a clipping constant `β`
  for the gradient strength — tunable per task. Training-time RTC's
  `d_max` is set during training and is harder to retune.

## Connection to other guidance mechanisms

| Mechanism | Inference-time? | What it constrains | Approximation |
|---|---|---|---|
| Classifier guidance (Dhariwal & Nichol 2021) | yes | classifier label | trained classifier |
| Classifier-free guidance (Ho & Salimans 2021) | partial | training-time prompt | none, but requires conditional + unconditional training |
| **ΠGDM** | yes | linear measurement `y = Ax_0` | linearized denoiser + pseudoinverse |
| Reconstruction guidance (Chung et al. 2022) | yes | nonlinear measurement | per-step gradient on a hand-written loss |
| Training-time prefix (TT-RTC) | **no** | fixed prefix of length `d` | exact (in training distribution) |
| Diffusion forcing (πR² staircase) | **no** | per-position noise schedule | exact (per-position τ trained for) |

## Open questions

- **Adaptive `β` schedule.** Current RTC uses a fixed clipping constant;
  whether an adaptive schedule (e.g., shrink `β` as `d` grows) closes
  the gap with TT-RTC is open.
- **ΠGDM for action-flow heads with implicit conditioning.** Modern
  policies have many auxiliary conditioning paths (proprio, goal, prior
  chunk); how ΠGDM compose with these is unclear.
- **Hybrid ΠGDM + training-time clamp.** For action heads partially
  fine-tuned, can ΠGDM cover the gap between train-time `d_max` and a
  larger deployment `d`?

## See also

- [[rtc]] — the canonical robotics application.
- [[training-time-rtc]] — the successor that removes ΠGDM entirely.
- [[pi-r-squared]] — the joint training-time + architectural
  generalization that subsumes ΠGDM-style inpainting.
- [[flow-matching]] — the underlying generative substrate.
- [[train-inference-mismatch]] — the cross-cutting framing.
