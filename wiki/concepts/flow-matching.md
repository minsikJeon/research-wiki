---
type: concept
title: Flow Matching (and Diffusion Conversion)
status: stub
tags: [flow-matching, diffusion, generative-model, robot-policy, action-chunking]
sources:
  - "[[black-2025-rtc]]"
related:
  - "[[action-chunking]]"
  - "[[rtc]]"
  - "[[vla]]"
created: 2026-05-28
updated: 2026-05-28
---

# Flow Matching (and Diffusion Conversion)

## Definition

A generative-modeling framework that trains a **velocity field** `v(x, t)`
to interpolate between a simple source distribution (e.g. Gaussian) and
a target data distribution by following an ODE: `dx/dt = v(x, t)`.
**Conditional flow matching** (Lipman et al. 2023) is the standard
training objective.

## Why it matters in the wiki

Flow matching is the training objective for the current best dexterous
manipulation VLAs (π0, π0.5; see [[vla]]). [[rtc]] explicitly targets
"any diffusion or flow-based VLA" and converts diffusion policies to
flow policies at inference time using known equivalences (Pokle 2024).

## Relevant properties for control

- **Few-step integration.** Unlike diffusion, flow matching can produce
  high-quality samples in 3–5 ODE steps, suitable for real-time
  control.
- **ΠGDM guidance compatible.** Pseudoinverse guidance (Song 2023)
  adds a Jacobian-based correction to the velocity field at every step,
  enabling inference-time conditioning without retraining.
- **Inference-time conditioning generalizes.** Image inpainting,
  classifier-free guidance, and [[rtc]]'s soft-masked chunk continuity
  all use the same algebra.

## Equation (RTC simplified form)

Standard Euler integration:
`A^{τ+1/n} = A^τ + (1/n) · v(A^τ, o, τ)`

With ΠGDM guidance for inpainting:
`v_ΠGDM = v + min(β, ((1−τ) / (τ · r_τ²))) · (Y − Â₁)^⊤ diag(W) ∂Â₁ / ∂A^τ`

where `Â₁ = A^τ + (1−τ)·v` is the one-step denoised estimate,
`W` is the soft mask, `Y` is the target, `β` is a guidance clip
constant (RTC's stability addition for low-step regimes), and
`r_τ² = 2(1−τ)² / (τ² + (1−τ)²)`.

## Open questions

- **Flow matching for perception outputs?** Most flow / diffusion
  literature targets continuous actions or images. Whether the same
  machinery applies to discrete-structured perception outputs
  (bounding boxes, tracks, pointmaps) is open.
- **Optimal denoising-step budget for control.** 3–5 steps is common
  but task-dependent; [[rtc]] ablates this implicitly via `β`.
