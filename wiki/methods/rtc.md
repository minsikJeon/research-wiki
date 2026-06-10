---
type: method
title: RTC (Real-Time Chunking via Inpainting)
status: stub
tags: [robotics, manipulation, vla, flow-matching, action-chunking, real-time, inference-time-algorithm, inpainting, asynchronous-control]
sources:
  - "[[black-2025-rtc]]"
related:
  - "[[action-chunking]]"
  - "[[asynchronous-control]]"
  - "[[flow-matching]]"
  - "[[vla]]"
  - "[[train-inference-mismatch]]"
  - "[[pi-gdm]]"
  - "[[training-time-rtc]]"
  - "[[pi-r-squared]]"
created: 2026-05-28
updated: 2026-06-10
---

# RTC (Real-Time Chunking via Inpainting)

Inference-time algorithm that lets any diffusion / flow-based VLA produce
**smooth, asynchronous, real-time control** without retraining. Treats
cross-chunk continuity as an **inpainting** problem: freeze the prefix
that has already been committed by the time the new chunk arrives, then
inpaint the rest consistent with the previous chunk's overlap region
and the latest observation.

## One-line summary

Asynchronous action chunking with `d`-action hard-frozen prefix +
soft-masked intermediate region (exponential decay) + freely generated
tail; [[pi-gdm|ΠGDM]] pseudoinverse guidance added to the flow-matching
velocity field at every denoising step.

## Inputs / outputs

- **Hyperparameters:** prediction horizon `H`, execution horizon
  `s ∈ [d, H − d]`, guidance clip `β`.
- **In (each loop):** observation `o_t`, previous chunk's remaining
  actions, current inference-delay estimate `d`.
- **Out:** the next action chunk `A_t = [a_t, ..., a_{t+H−1}]`, made
  available at time `t + d·Δt`.

## How it works

1. **Asynchronous schedule.** Inference starts at execution step
   `s − d` so that the new chunk is ready at step `s`. The controller
   thread consumes one action per Δt; the inference thread runs in the
   background.
2. **Freeze prefix.** The first `d` actions of the new chunk get mask
   weight 1 and are pinned to the corresponding actions of the
   previous chunk that are guaranteed to execute.
3. **Soft-mask intermediate.** Actions `d ≤ i < H − s` get
   exponentially decaying weights
   `c_i = (e^{i−1} − 1) / (H − s − d + 1)`, allowing the policy to
   honor the previous chunk's trajectory near the seam and become
   freer later.
4. **Free tail.** The last `s` actions get weight 0 — purely generated.
5. **ΠGDM velocity correction** (Eq 2 of source): at every denoising
   step `τ`, add a vector-Jacobian-product term that pushes the
   one-step denoised estimate `Â₁` toward the soft-masked target `Y`.
   Clipped by `β` for stability under few denoising steps.
6. **Flow integration** (Eq 1): `A_t^{τ+1/n} = A_t^τ + (1/n) · v_ΠGDM`.

## Where it's been applied

- **Kinetix simulator** (new benchmark, 12 dynamic tasks): throwing,
  catching, balancing.
- **Real bimanual manipulation** (6 tasks, π0.5 base policy):
  match-lighting is the flagship demo, robust to >300ms inference delay.

## Known limitations

- Compute overhead from ΠGDM guidance (vector-Jacobian product per
  denoising step).
- Requires a generative policy (flow / diffusion). Non-generative
  policies need a separate solution.
- Soft-mask decay schedule and `β` clipping are hand-designed.
- Real-world benchmarks are custom to π0.5; no shared comparison
  protocol vs other VLA real-time methods.

## Related methods

- **Predecessors:** [[act]] / temporal ensembling ([[zhao-2023-act]]);
  BID (cited as the best alternative in their sim).
- **Asynchronous-control siblings:** [[diffusion-policy]] inference
  variants — receding horizon over standard DDPM.
- **Train-time successor:** [[training-time-rtc]] /
  [[black-2025-training-time-rtc]] — moves the prefix conditioning to
  training; eliminates ΠGDM overhead. Drop-in replacement; wins for
  `d ≥ 2`.
- **Joint generalization:** [[pi-r-squared]] /
  [[anon-2026-pi-r-squared]] — training-time prefix clamp + ramped
  interior via diffusion forcing + fast/slow channel split.
  Training-time RTC is its `s = 0` corner.
- **Base VLA:** π0.5 (Black et al. 2024) — not yet ingested directly.
