---
type: method
title: Diffusion Policy
status: growing
tags: [robotics, manipulation, imitation-learning, diffusion, visuomotor-policy, action-chunking]
sources:
  - "[[chi-2024-diffusion-policy]]"
related:
  - "[[action-chunking]]"
  - "[[flow-matching]]"
  - "[[act]]"
  - "[[rtc]]"
  - "[[pi-r-squared]]"
  - "[[russ-tedrake]]"
created: 2026-06-08
updated: 2026-06-08
---

# Diffusion Policy (DP)

Conditional DDPM over action chunks for visuomotor imitation learning.
Inherits ACT's chunking + receding-horizon control; replaces the CVAE
with a denoising diffusion head. Foundation of every modern
diffusion / flow VLA in this wiki.

## One-line summary

Image encoder (ResNet/CLIP) → cross-attention from action-chunk tokens
→ time-series transformer denoiser; `K = 100` denoising steps per
replan (or 10 with DDIM); receding horizon `H = 16`, execution `s = 8`.

## Inputs / outputs

- **In:** stack of `T_o = 2` recent frames + proprioception.
- **Out:** action chunk `A_t = [a_t, ..., a_{t+H−1}]`, `H = 16`. Execute
  first `s = 8` actions, replan.

## How it works

### Architecture variants

- **Transformer (preferred):** DiT-style blocks over the action chunk;
  self-attention within chunk + cross-attention to obs features.
  Sinusoidal time embedding for diffusion timestep.
- **CNN-1D variant:** 1D temporal conv over action chunk; FiLM
  conditioning on obs features. Cheaper, slightly worse.

### Training

DDPM noise-prediction loss:

```
L = E[‖ε − ε_θ(A_t^k, o_t, k)‖²]
```

where `A_t^k = sqrt(α̅_k) A_t + sqrt(1 − α̅_k) ε`, standard DDPM noise
schedule.

### Inference

- `K = 100` DDPM steps, or `K = 10` DDIM steps (essentially equivalent
  performance, 10× faster).
- Receding horizon: re-denoise the full chunk on every replan.

## Where it's been applied

- **Robomimic** (PH, MH) — proficient + multi-human demos.
- **Push-T** — multi-modal demos benchmark.
- **Multi-Stage tasks** — long-horizon manipulation.
- **Real bimanual / single-arm** tasks (3 real-world).
- **As the standard action head** for nearly all post-2024
  diffusion-VLAs: OpenVLA, π0 / π0.5 / π0.6 (Physical Intelligence
  family), GR00T-N1.7 (NVIDIA), RDT-1B, RT-2-like systems.

## Known limitations

- **Inference latency.** `K` denoising steps per replan — motivates
  every follow-up in the real-time line ([[rtc]],
  [[training-time-rtc]], [[pi-r-squared]]).
- **Open-loop within chunk.** Same as ACT.
- **No language conditioning** in the original paper — extended later
  by VLA derivatives.
- **Benchmarks are imitation only.** Diffusion-as-policy in RL is
  explored elsewhere.

## Related methods

- **Concept page:** [[action-chunking]] (inherited from
  [[act]]).
- **Direct predecessor:** [[act]] — CVAE → diffusion swap.
- **Real-time successors:** [[rtc]], [[training-time-rtc]],
  [[pi-r-squared]] — solve the inference-latency problem.
- **Flow-matching cousin:** [[flow-matching]] — same expressiveness,
  fewer sampling steps; replaces DDPM in most recent VLAs.
- **VLA descendants:** OpenVLA, π0 / π0.5 / π0.6, GR00T-N1.7.
