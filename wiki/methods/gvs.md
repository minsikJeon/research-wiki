---
type: method
title: GVS (Generative View Stitching)
status: growing
tags: [video-diffusion, diffusion-stitching, training-free, camera-guided-generation, omni-guidance, loop-closure, long-horizon-video, history-guidance]
sources:
  - "[[song-2026-gvs]]"
related:
  - "[[dfot]]"
  - "[[history-guidance]]"
  - "[[diffusion-forcing]]"
  - "[[chen-2024-diffusion-forcing]]"
  - "[[song-2025-history-guided-video-diffusion]]"
created: 2026-06-12
updated: 2026-06-12
---

# GVS (Generative View Stitching)

A **training-free** diffusion stitching method for **camera-guided
long-horizon video generation**. Composes the joint distribution over
arbitrarily long videos as a product of overlapping chunk distributions
denoised in parallel by an off-the-shelf [[dfot|DFoT]] backbone.
Introduces **Omni Guidance** and **cyclic conditioning** for temporal
consistency and loop closure.

## One-line summary

Split the long video into overlapping chunks, denoise every chunk
jointly with its temporal neighbors, steer the score toward the
desired past+future-conditional via Omni Guidance, alternate temporal +
spatial windows for loop closure.

## Inputs / outputs

- **In:** predefined camera trajectory `p_0, p_1, ..., p_{T−1}`;
  optionally start/goal frames.
- **Out:** video `x_0, ..., x_{T−1}` faithful to the trajectory,
  temporally consistent, collision-free, and loop-closed.
- **Backbone:** any model trained with Diffusion Forcing — no retraining.

## How it works

### Joint-denoise the target chunk with temporal neighbors

Each diffusion step:
```
For each target chunk x_t:
    window = [x^k_{t-1}, x^k_t, x^k_{t+1}]
    update = ε_θ(window, k, p_{t-1:t+1})
    only keep update for x^k_t   # discard neighbor updates
```

This is **vanilla GVS** — already 1 forward pass per chunk per step,
with no custom training.

### Omni Guidance (Eqs 4–8)

Vanilla GVS denoises under the *joint* score `p_θ(x_{t-1:t+1} | p)`
instead of the desired *conditional* score `p_θ(x_t | x_{t-1}, x_{t+1},
p_{t-1:t+1})`. Omni Guidance corrects this:

```
ε̃_θ = (1+γ) ε_θ(x^k_{t-1:t+1} | p_{t-1:t+1})
     − γ ε_θ(∅, x^k_t, ∅ | ∅, p_t, ∅)
```

The second term is computed by **replacing neighboring chunks with
pure noise** (`k_{t-1} = k_{t+1} = 1`) — possible only because of DFoT's
per-frame independent noise levels.

This is a **generalization of HG-f** ([[history-guidance]]) where the
conditioning chunks' noise levels evolve during stitching instead of
staying fixed.

### Partial stochasticity

```
σ^k = η √(1 − α^{k−1})    η ∈ (0, 1]
```

- η = 0: no random noise → high quality but weak consistency.
- η = 1: maximum stochasticity → strong consistency but oversmoothing.
- η ∈ (0.5, 0.9) **with Omni Guidance** → consistency without
  oversmoothing.

### Cyclic conditioning for loop closure (§3.5)

```
At each diffusion step, alternate:
    Mode A (temporal windows): condition on temporally-adjacent chunks
    Mode B (spatial windows):  condition on temporally-distant but
                                spatially-close chunks
```

Spatial windows are picked from camera-trajectory overlap (FoV-based).
Alternation ensures the target chunk effectively conditions on both
temporal and spatial neighbors over the course of denoising.

## Where it's been applied

In [[song-2026-gvs]]:
- 120-frame navigation videos through panoramas, circles, straight
  lines, stairs.
- 862-frame stable rollouts on RealEstate10K.
- Variant of Penrose's **Impossible Staircase** — a 120-frame video
  that visually closes a loop despite height discontinuity in the
  conditioning trajectory.

Beats autoregressive sampling and StochSync on frame-to-frame
consistency, long-range consistency, and collision avoidance.

## Known limitations

- **Offline only** — requires the full camera trajectory ahead of time.
- **Backbone-bound** — performance ceiling set by the DFoT model.
- **Spatial windows hand-picked** — no automatic selector for cyclic
  conditioning.

## Related methods

- **Required backbone:** [[dfot]] — GVS only works with DFoT-trained
  models. The per-frame independent noise schedule is the necessary
  affordance.
- **Direct precursor:** [[history-guidance]] HG-f variant — Omni
  Guidance is its dynamic extension.
- **Prior stitching:** CompDiffuser (custom-trained), StochSync
  (training-free but not video).
- **Score correction inspiration:** Inner Guidance (Chefer et al. 2025).
- **For 3D tracking analogue:** in offline post-hoc tracking (dataset
  annotation, re-anchoring after drift), Omni Guidance over a
  DFoT-style chunk denoiser is a candidate for fixing accumulated
  errors using future observations.
