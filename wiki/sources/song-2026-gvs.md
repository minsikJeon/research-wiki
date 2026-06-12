---
type: source
source_type: paper
title: "Generative View Stitching"
authors:
  - Song, Chonghyuk
  - Stary, Michal
  - Chen, Boyuan
  - Kopanas, George
  - Sitzmann, Vincent
year: 2026
venue: ICLR 2026
url: https://arxiv.org/abs/2510.24718
raw_path: papers/2510.24718v3.pdf
status: ingested
tags: [video-diffusion, diffusion-stitching, camera-guided-generation, training-free, history-guidance, omni-guidance, loop-closure, long-horizon-video, diffusion-forcing]
related:
  - "[[gvs]]"
  - "[[dfot]]"
  - "[[history-guidance]]"
  - "[[song-2025-history-guided-video-diffusion]]"
  - "[[chen-2024-diffusion-forcing]]"
  - "[[diffusion-forcing]]"
  - "[[boyuan-chen]]"
  - "[[mit-csail]]"
  - "[[train-inference-mismatch]]"
created: 2026-06-12
updated: 2026-06-12
---

# Generative View Stitching

## TL;DR

GVS is a **training-free** diffusion-stitching method for **long-horizon
camera-guided** video generation. It overcomes a basic failure of
autoregressive (AR) video diffusion — that AR cannot condition on the
*future*, so a predefined camera trajectory collides with the generated
scene and rollouts collapse. GVS samples the entire video in parallel,
splits it into overlapping chunks, denoises every target chunk *jointly*
with its temporal neighbors. Adds **Omni Guidance** (a CFG-style score
correction conditioning on past AND future) and **cyclic conditioning**
for explicit loop closure. Compatible with any off-the-shelf
[[dfot|DFoT]]-trained video model.

## Why it matters

For the user's wiki, GVS is the **bidirectional / global** version of
[[history-guidance]] — the same diffusion-forcing backbone, but now
sampling extends to *future* conditioning. This matters for three
threads:

- **Diffusion-forcing thread:** GVS demonstrates DFoT models are
  *already* stitching-capable without retraining — strong empirical
  validation of the noise-as-masking framework's flexibility.
- **Robotics/planning thread:** GVS's camera-guided video generation is
  effectively long-horizon trajectory planning with collision
  constraints — adjacent to motion planning and synthetic data
  generation for autonomous driving.
- **For the middle-ground 3D-tracker:** GVS shows you can take a *fixed*
  DFoT-style chunk denoiser and run it bidirectionally at inference
  time via Omni Guidance. The same trick may apply to 3D-tracker
  inference when offline context exists (e.g., for re-anchoring after
  drift).

## Key claims

- **CompDiffuser needed a custom-trained model for stitching; GVS does
  not.** DFoT's per-frame independent noise schedule is the sole
  affordance required (§3.2). DFoT was trained for video generation —
  GVS shows it doubles as a stitching backbone for free.
- **Vanilla GVS** (just joint denoising of chunks) achieves weak
  temporal consistency (Fig 3 top row, Table 2a) because the target
  chunk is denoised under a *joint* score instead of the original
  *conditional* score.
- **Omni Guidance (Eqs 4–8)** is the load-bearing fix:
  ```
  ε̃_θ = (1+γ) ε_θ(x^k_{t-1:t+1} | p_{t-1:t+1})
       − γ ε_θ(∅, x^k_t, ∅ | ∅, p_t, ∅)
  ```
  Steer the joint score toward the *desired* conditional by subtracting
  the unconditional and re-adding the bidirectional past+future score.
  Generalization of HG-f ("Fractional History Guidance") where neighbor
  noise levels *change* throughout stitching.
- **Partial stochasticity** (η ∈ (0, 1)) reduces oversmoothing.
  Maximum stochasticity (η = 1) helps consistency but causes oversmoothing;
  Omni Guidance permits partial stochasticity without losing consistency
  (Fig 3, Table 2b).
- **Loop closure requires explicit cyclic conditioning (§3.5).** Even
  with global theoretical receptive field, GVS fails to "visually return
  to the same place" without cyclic conditioning. Cyclic conditioning
  alternates between **temporal windows** (temporal neighbors) and
  **spatial windows** (temporally-distant but spatially-close neighbors).
- **Generates 120-frame to 862-frame videos** through impossible-staircase
  topologies (Fig 7), panoramas, circles, straight lines — all from an
  8-frame-context DFoT backbone.

## Methods

### Setup (§3.1, Eq 2)

CompDiffuser frames a sequence distribution compositionally:
```
p(x | x_start, x_goal) ∝ p_0(x_0 | x_start, x_1) p_{T−1}(x_{T−1} | x_{T−2}, x_goal)
                          × Π_{t=1..T−2} p_t(x_t | x_{t−1}, x_{t+1})
```
This requires a custom-trained model `ε_θ(x_t, k, x_{t−1}, x_{t+1})`.

### GVS observation (§3.2, Eq 3)

A DFoT video model trained on `(x_1, ..., x_T) ~ p_θ(x | p)` already
factors as `Π p_t(x_t | x_{t−1}, x_{t+1}, p_{t−1}, p_t, p_{t+1})`.
The chunks `[x^k_{t−1}, x^k_t, x^k_{t+1}]` can be denoised *jointly*
inside a single forward pass — no custom model.

### Vanilla GVS (Fig 2)

```
For each diffusion step k = K..1:
    For each chunk t:
        x'_{t−1:t+1} = ε_θ([x^k_{t−1}, x^k_t, x^k_{t+1}], k)
        update x^k_t with x'_t          # keep target update
        discard x'_{t−1}, x'_{t+1}      # neighboring chunks not updated
```

Window includes target + ≥1 neighbor. Quality suffers because the
target chunk is denoised under a joint score, not the conditional score
(§3.2 end).

### Omni Guidance (§3.4, Eqs 4–8)

Inspired by Inner Guidance (Chefer et al., 2025). Modifies sampling:
```
p̄(x^k_{t-1:t+1} | p_{t-1:t+1}) ∝ p_θ(x^k_{t-1:t+1} | p_{t-1:t+1})
                                  × [p_θ(x^k_{t-1:t+1} | x^k_t, p_{t-1:t+1}) / p_θ(x^k_{t-1:t+1} | p_{t-1:t+1})]^γ
```
Single-γ form (Eq 8):
```
ε̃_θ = (1+γ) ε_θ(x^k_{t-1:t+1} | p_{t-1:t+1})
     − γ ε_θ(∅, x^k_t, ∅ | ∅, p_t, ∅)
```
The unconditional term `ε_θ(∅, x^k_t, ∅ | ∅, p_t, ∅)` is computed by
replacing the noisy neighboring chunks with pure noise and setting
their `k_t = 1`. This is **a generalization of HG-f** with the twist
that conditioning chunks' noise levels *evolve* throughout stitching.

### Cyclic conditioning for loop closure (§3.5, Fig 5)

Even with theoretically-global receptive field, vanilla GVS fails to
close loops. Cyclic conditioning *alternates* between two sets of
context windows per denoising step:
- **Temporal windows:** condition on temporally-adjacent chunks.
- **Spatial windows:** condition on temporally-distant but
  spatially-close chunks (based on camera-frustum overlap).

By alternating, the target chunk effectively conditions on both spatial
and temporal neighbors over the full denoising process.

### Maximum stochasticity vs partial stochasticity

```
σ^k = η √(1 − α^{k−1})       η ∈ (0, 1]
```
η = 1 is StochSync's maximum stochasticity → consistency but
oversmoothing. η ∈ (0.5, 0.9) reduces oversmoothing while keeping
consistency, provided Omni Guidance is active.

## Results

### Camera-guided video generation (Table 1)

7 challenging predefined camera trajectories: Panorama 1-loop, Panorama
2-loop, Circle 1-loop, Circle 2-loop, Straight line, Stairs, Staircase
circuit. Metrics: frame-to-frame consistency (F2FC), long-range
consistency (LRC), image quality (IQ), aesthetic quality (AQ),
collision avoidance (CA).

| Trajectory | AR sampling F2FC / LRC | StochSync F2FC / LRC | **GVS F2FC / LRC** |
|---|---|---|---|
| Panorama 1-loop | 0.168 / 0.339 | 0.183 / 0.164 | **0.138 / 0.141** |
| Panorama 2-loop | 0.169 / 0.171 | 0.155 / 0.116 | **0.155 / 0.116** |
| Circle 1-loop | 0.220 / 0.411 | 0.204 / 0.258 | **0.160 / 0.244** |
| Staircase circuit | 0.132 / 0.449 | 0.179 / 0.221 | **0.129 / 0.176** |

**GVS beats AR + StochSync on F2FC, LRC, and CA** on all trajectories
while keeping comparable IQ/AQ.

### Ablations

- **Omni Guidance** (Table 2): consistently better F2FC at all
  stochasticity levels. Crucial for keeping quality at η < 1.
- **Loop closure** (Table 3): without explicit loop closing, LRC fails
  even *with* Omni Guidance. Both are needed.
- **Impossible Staircase (Fig 7):** novel 120-frame application —
  navigating Reutersvärd's variant of Penrose's impossible staircase
  while forming a *visually* continuous loop despite height
  discontinuity. Showcases all three contributions (GVS, Omni Guidance,
  cyclic conditioning).

## Limitations / open questions

- **Offline only.** GVS breaks causality by design — incompatible with
  streaming / online settings where future camera poses are unknown.
- **Performance ties to backbone.** GVS does not retrain the DFoT
  backbone; failure modes (e.g., wide-baseline loop closure outside
  training data) are inherited.
- **Spatial windows manually defined.** Cyclic conditioning's spatial
  windows are picked per trajectory. No automatic selector.
- **Conditioning on external images is difficult.** GVS struggles to
  propagate context frames provided as image-prompts to distant chunks.

## Connections

- **[[dfot]]** — the necessary backbone. GVS uses DFoT's per-frame
  independent noise levels as its sole affordance — no retraining.
- **[[song-2025-history-guided-video-diffusion]]** — direct predecessor.
  HG-f corresponds to *static* low-pass guidance; Omni Guidance is the
  dynamic version where conditioning chunks' noise levels co-evolve
  with the target chunk's.
- **[[history-guidance]]** — concept page; Omni Guidance is a new
  variant to add to the HG family table.
- **[[chen-2024-diffusion-forcing]]** — original CDF; the
  per-token-noise paradigm is what makes both stitching and HG possible.
- **CompDiffuser** (Luo et al. 2025) — prior diffusion stitching method
  requiring custom backbone. GVS shows the custom training is
  unnecessary.
- **StochSync** (Yeo et al. 2025) — prior training-free stitching for
  panoramas/textures. Lacks temporal consistency for video.
- **Inner Guidance** (Chefer et al. 2025) — score-correction idea
  borrowed by Omni Guidance.
- **For middle-ground 3D-tracker:** when an offline trajectory is
  available (post-hoc re-tracking, dataset annotation), Omni Guidance
  on a DFoT-style chunk denoiser is a candidate for re-anchoring tracks
  to fix drift — same mechanism, different domain.

## Citation

Song, C., Stary, M., Chen, B., Kopanas, G., & Sitzmann, V. (2026).
*Generative View Stitching.* ICLR 2026.
arXiv:2510.24718. https://arxiv.org/abs/2510.24718
