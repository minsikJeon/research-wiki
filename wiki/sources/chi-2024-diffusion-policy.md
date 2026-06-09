---
type: source
source_type: paper
title: "Diffusion Policy: Visuomotor Policy Learning via Action Diffusion"
authors:
  - Chi, Cheng
  - Xu, Zhenjia
  - Feng, Siyuan
  - Cousineau, Eric
  - Du, Yilun
  - Burchfiel, Benjamin
  - Tedrake, Russ
  - Song, Shuran
year: 2024
venue: IJRR 2024 (RSS 2023 best paper)
url: https://arxiv.org/abs/2303.04137
raw_path: papers/2303.04137v5.pdf
status: ingested
tags: [robotics, manipulation, imitation-learning, diffusion, visuomotor-policy, action-chunking, multimodal-distribution]
sources: []
related:
  - "[[diffusion-policy]]"
  - "[[action-chunking]]"
  - "[[flow-matching]]"
  - "[[zhao-2023-act]]"
  - "[[russ-tedrake]]"
created: 2026-06-08
updated: 2026-06-08
---

# Diffusion Policy: Visuomotor Policy Learning via Action Diffusion

## TL;DR

Visuomotor policy as a **conditional denoising diffusion process** over
action sequences. Three contributions: (1) receding-horizon control via
action chunking + DDPM; (2) visual conditioning via image-encoder
features as cross-attention keys; (3) time-series transformer for the
denoiser. **+46.9% average improvement over prior state of the art**
across 15 tasks from 4 benchmarks (Robomimic, Push-T, Multi-Stage, Real).

Together with [[zhao-2023-act]], establishes the two canonical
expressive action-chunking policy paradigms: **CVAE (ACT) vs diffusion
(DP)**. Diffusion has won — all subsequent VLAs ([[rtc]] base π0/π0.5,
[[anon-2026-pi-r-squared]] base GR00T-N1.7) use diffusion/flow heads.

## Why it matters

- **Foundational visuomotor diffusion policy.** Every subsequent
  diffusion-VLA in this wiki ([[rtc]], [[black-2025-training-time-rtc]],
  [[anon-2026-pi-r-squared]]) inherits the receding-horizon + DDPM-head
  + chunked-prediction recipe from this paper.
- **Multimodal action distribution handling.** DDPM's
  expressiveness over multi-modal distributions is the key advantage
  over explicit policies (regression to mean) and implicit policies
  (training instability with EBMs). This is why diffusion-style action
  heads are now standard.
- **The bridge from ACT to flow-matching VLAs.** ACT replaced
  regression with CVAE; DP replaced CVAE with DDPM; flow matching
  ([[flow-matching]]) replaces DDPM. Same chunking skeleton throughout.

## Key claims

- **Three policy representations** (Fig. 1):
  - Explicit: `a = F_θ(o)` — regression. Mean-collapses on multi-modal
    demos.
  - Implicit: `E_θ(o, a)` — energy-based; samples by minimization.
    Unstable training on continuous actions.
  - **Diffusion: `∇E_θ(o, a)` — denoising.** Models the *score* of
    the action distribution, not the energy or the action itself.
    Trained stably, samples multimodally.
- **Conditional DDPM over action chunks.** `A_t = [a_t, ..., a_{t+H−1}]`,
  noise schedule from DDPM, conditioned on observation `o_t`.
- **Receding-horizon control.** Predict `H = 16` actions, execute
  `s = 8`, replan. Same chunking idea as ACT; here the chunk
  *distribution* is multi-modal due to diffusion.
- **Visual conditioning via cross-attention.** Image encoder
  (ResNet/CLIP) → per-frame features → transformer cross-attention from
  action tokens. No FiLM, no concat-and-flatten.
- **Time-series diffusion transformer.** Stack of transformer blocks
  over the action chunk; self-attention within chunk + cross-attention
  to obs. (CNN-1D variant also reported; transformer wins.)
- **+46.9% average improvement** over prior SOTA on 15 tasks.

## Methods

### Setup

- Observation `o_t` = stack of `T_o = 2` recent frames + proprio.
- Action chunk `A_t = [a_t, ..., a_{t+H−1}]`, `H = 16`.
- Execute first `s` actions, replan. `s = 8` typical.
- DDPM loss: `L = E[‖ε − ε_θ(A_t^k, o_t, k)‖²]` where `A_t^k` is
  noisy at level `k`, `ε` is the added noise.

### Architecture

- **CNN backbone variant**: 1D temporal conv over action chunk; FiLM
  conditioning on obs features.
- **Transformer variant** (better): self-attention within `A_t`,
  cross-attention to obs features. Standard DiT-style block.
- **Both** use sinusoidal time embedding for diffusion timestep.

### Inference

- `K = 100` (or 10 with DDIM) denoising steps per chunk.
- Receding horizon: re-denoise on every replan.

## Results (headline)

| Benchmark | Prior SOTA | DP | Gain |
|---|---|---|---|
| Robomimic-PH (avg) | ~76% | 91% | +15% |
| Robomimic-MH (avg) | ~67% | 90% | +23% |
| Push-T (avg score) | 0.79 | 0.91 | +0.12 |
| Multi-Stage Avoiding | 0.41 | 0.91 | +0.50 |
| Real (3 tasks) | varies | 95% avg | +large |

- **+46.9% relative improvement** averaged across 15 tasks.
- Beats LSTM-GMM, IBC, BCRNN, BC-RNN-GMM.
- DDIM 10-step ≈ DDPM 100-step performance with 10× inference speedup.

## Limitations / open questions

- **Inference cost is the obvious one.** `K` denoising steps per replan
  is expensive — motivates the [[rtc]] / [[black-2025-training-time-rtc]]
  / [[anon-2026-pi-r-squared]] real-time line.
- **Action chunking is open-loop within chunk.** Same issue as ACT;
  inherited limitation.
- **No language conditioning.** This is the pre-VLA era; the recipe
  later gets extended with VLM frontends (RT-2, OpenVLA, π0).
- **Evaluated only on imitation learning, not RL.** Diffusion-as-policy
  has been explored in RL too but this paper sticks to BC.

## Connections

- **Method page:** [[diffusion-policy]] (created in this ingest).
- **Concept page:** [[action-chunking]] — DP keeps the ACT recipe,
  changes the head.
- **Direct predecessors:** [[zhao-2023-act]] (chunking origin),
  IBC (Florence et al. 2021 — implicit policies).
- **Direct successors:** every diffusion / flow VLA in this wiki —
  π0, π0.5, π0.6 (Physical Intelligence), GR00T-N1.7 (NVIDIA), OpenVLA,
  RT-2, RDT-1B, [[rtc]], [[black-2025-training-time-rtc]],
  [[anon-2026-pi-r-squared]].
- **Authors:** Cheng Chi (Columbia; lead — defer entity), Zhenjia Xu,
  Siyuan Feng, Eric Cousineau, Yilun Du (MIT; also on Diffusion
  Forcing), Benjamin Burchfiel (TRI), [[russ-tedrake]] (TRI/MIT — also
  on Diffusion Forcing — promoted), Shuran Song (Stanford; senior —
  defer entity).

## Citation

Chi, C., Xu, Z., Feng, S., Cousineau, E., Du, Y., Burchfiel, B.,
Tedrake, R., & Song, S. (2024). *Diffusion Policy: Visuomotor Policy
Learning via Action Diffusion.* IJRR 2024 (RSS 2023). arXiv:2303.04137v5.
https://arxiv.org/abs/2303.04137
