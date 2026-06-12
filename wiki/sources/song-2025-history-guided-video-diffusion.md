---
type: source
source_type: paper
title: "History-Guided Video Diffusion"
authors:
  - Song, Kiwhan
  - Chen, Boyuan
  - Simchowitz, Max
  - Du, Yilun
  - Tedrake, Russ
  - Sitzmann, Vincent
year: 2025
venue: ICML 2025 (PMLR 267)
url: https://arxiv.org/abs/2502.06764
raw_path: papers/2502.06764v2.pdf
status: ingested
tags: [video-diffusion, diffusion-forcing, generative-model, transformer, long-context, classifier-free-guidance, world-model, robotics]
sources: []
related:
  - "[[dfot]]"
  - "[[diffusion-forcing]]"
  - "[[diffusion-forcing-method]]"
  - "[[chen-2024-diffusion-forcing]]"
  - "[[history-guidance]]"
  - "[[train-inference-mismatch]]"
  - "[[russ-tedrake]]"
  - "[[mit-csail]]"
created: 2026-06-11
updated: 2026-06-11
---

# History-Guided Video Diffusion (DFoT)

## TL;DR

Extends [[chen-2024-diffusion-forcing]] from causal RNNs to **non-causal
DiT transformers** for video generation. Trains a single video DiT to
denoise sequences with **per-frame independent noise levels** (Diffusion
Forcing applied to a transformer), then introduces **History Guidance
(HG)**: a CFG-like family that conditions on *arbitrary subsets* of past
frames at *arbitrary noise levels*. Result: stable 862-frame autoregressive
rollouts from a single image (~10× longer than prior); on-par with
industry models (W.A.L.T) at much less compute; SOTA on Kinetics-600,
RealEstate10K, Minecraft long-context.

## Why it matters

This is the **first practical demonstration that diffusion forcing scales
to large non-causal video transformers**, and the first paper to use the
"noise level = mask" identity to derive a new family of guidance methods.

Three reasons it's load-bearing for this wiki:

- **Direct extension of [[chen-2024-diffusion-forcing]]** by the same lead
  (Boyuan Chen) + Sitzmann + Tedrake (MIT CSAIL again). Establishes DF as
  more than a foundational paper — it's now a deployable training
  paradigm in the video-diffusion regime.
- **The "noise as masking" framing** generalizes CFG: unconditional
  score = all-history-noised, conditional = clean-history, intermediate
  noise = partial conditioning. This is the same idea πR² uses for action
  chunks (clean prefix = clamped, ramped interior, noise tail), now
  formalized for guidance composition.
- **Most relevant precedent for the user's perception-side middle-ground
  design.** The user's 3D-tracker MVP needs a way to use a streaming 3D
  backbone's recurrent state as conditioning during chunk denoising —
  DFoT is exactly "condition on partially-noised history frames in a
  non-causal transformer." The non-causal property (vs CDF's causal RNN)
  is what makes it portable to chunked perception.

## Key claims

- **DFoT generalizes CFG.** Standard CFG drops one conditioning variable
  (text) via binary masking. DFoT recasts *any* subset of frames at *any*
  noise level as conditioning. CFG is the special case where the binary
  mask is realized by a separate-encoder swap.
- **Noise as masking** (§4). Forward diffusion at `k=0` = clean frame,
  `k=1` = pure noise = "fully masked". Intermediate `k` = partially
  masked. One unified framework.
- **Per-frame independent noise levels** at training (Eq 4). Each frame
  `x_t` gets `k_t ∼ Unif[0,1]`. Loss = MSE on noise prediction over
  all frames. Same per-token recipe as [[chen-2024-diffusion-forcing]]
  but applied to a non-causal video DiT.
- **Architecture is minimal change.** Standard video DiT + AdaLN with
  per-frame noise level. **Same parameter count.** Existing pretrained
  models can be fine-tuned into DFoT (12.5% of scratch training cost).
- **Framewise Binary Dropout (BD) is worse.** Binary dropout as the
  alternative way to support flexible history performs significantly
  worse than DFoT (Table 1, FVD 6.4 vs 4.3). Reason: BD wastes tokens —
  only the unmasked subset contributes to loss; DFoT computes loss on
  *all* frames simultaneously.
- **History Guidance (HG) family** (§5):
  - **HG-v (Vanilla)** — use any history length as conditioning;
    unconditional = full-noise history (Eq 2).
  - **HG-t (Temporal)** — compose scores from *different history
    windows* `{H_long, H_short}` to balance long-range coherence vs
    short-range reactivity.
  - **HG-f (Fractional)** — condition on history corrupted at varying
    noise levels `k_H ∈ (0, 1)`. Effectively a *low-pass filter on
    history*: high-frequency details unconstrained → more dynamics,
    less static-video collapse.
  - **HG-tf** — composition along both axes.
- **HG-v improves quality monotonically with guidance scale `ω`** but at
  cost of dynamics (collapse to static videos at `ω ≥ 3`).
- **HG-f resolves the static-video collapse** by guiding with only
  low-frequency history components (`k_H ∈ (0,1)`). Beats SD and DFoT-no-guidance
  (FVD 170.4 vs 247.5 vs 208.0) on Kinetics-600.
- **HG-t enables robust OOD-history generation.** Splits long OOD
  histories into shorter in-distribution subsequences and composes
  scores (Fig 7). Other methods fail to generalize.
- **Ultra long rollouts.** 862-frame navigation video from one image
  on RE10K. Standard baselines diverge in dozens of frames.
- **Long-horizon reactive imitation learning** (Fruit Swapping, §6.4
  Task 3). Combines full-history score (memory) + single-frame score
  (reactivity) via HG-t. 83% success vs baselines that "fail completely."

## Methods

### Setup

- Video sequence `x_{1:T}`. Indices partitioned into history `H ⊂ T`
  and generation `G = T \ H`.
- Per-frame noise levels `k_T = [k_1, ..., k_T] ∈ [0, 1]^T`.
- Diffusion forward process: `x_t^{k_t} = α_{k_t} x_t^0 + σ_{k_t} ε`.

### Noise-as-masking (§3)

- `k_t = 0`: clean frame (full conditioning).
- `k_t = 1`: fully noised = "fully masked" = no information.
- `k_t ∈ (0, 1)`: partial masking with a noisy snapshot.

This identity lets one model serve **any** subset / noise-level
combination of history conditioning.

### DFoT training (Eq 4)

```
for each minibatch:
    sample video x_{1:T}
    for t = 1, ..., T:
        sample k_t ∼ Unif[0, 1]
    x_t^{k_t} = α_{k_t} x_t + σ_{k_t} ε_t
    loss = || ε_θ(x_T^{k_T}, k_T) − ε_T ||²
```

- Loss computed on **all** `T` frames (per-frame independent noise).
- Non-causal architecture: full attention across all frames; per-frame
  AdaLN conditions on per-frame `k_t`.
- Theorem 4.1: this objective optimizes a reweighted ELBO over expected
  log-likelihoods of all subsequences (App A.1).

### History Guidance variants (§5)

- **HG-v (Eq 2):** condition on history `H` at `k_H = 0`; unconditional
  estimate via `k_H = 1` (fully noised history).
- **HG-t:** compose scores from multiple history windows of different
  lengths.
- **HG-f (Eq 6):**
  `∇log p(x_G | x_H) + ω · [∇log p(x_G | x_H^{k_H}) − ∇log p(x_G)]`
  where `k_H ∈ (0,1)` controls the masking degree; higher `k_H` =
  guide only with lower-frequency history.
- **HG-tf (Eq 5):** sum of scores conditioned on different history
  subsequences `{H_i}` at potentially different noise levels.

### Sampling

Standard DDPM / DDIM. The flexibility comes from how `k_T` is chosen at
each step — set `k_H = 0` for history frames, `k_G = current noise level`
for generation.

## Results (headline)

### Generic video quality (Kinetics-600, FVD ↓)

| Method | Flexible? | FVD |
|---|---|---|
| MAGVIT-v2 (industry) | ✗ | 4.3 |
| W.A.L.T (industry) | ✗ | 3.3 |
| Video Diffusion | △ | 16.2 |
| MAGVIT (orig.) | ✓ | 9.9 |
| Standard Diffusion (SD) | ✗ | 4.8 |
| Full-Sequence (FS) | △ | 95.5 |
| Binary Dropout (BD) | ✓ | 6.4 |
| **DFoT (scratch)** | ✓ | **4.3** |
| **DFoT (fine-tuned)** | ✓ | **4.7** |

DFoT matches the best fixed-history model (SD) **while supporting
flexible history**, and is on-par with industry-compute models.

### History guidance gains (Kinetics-600)

- DFoT without guidance: FVD 208.0
- DFoT + HG-v: 181.6 (improves quality, hurts dynamics at high ω)
- **DFoT + HG-f: 170.4** (best — improves quality *and* dynamics)
- SD baseline: 247.5
- Full-sequence baseline: 1040

### OOD history (RealEstate10K extreme camera rotations)

- DFoT + HG-t: minimal performance drop; coherent generation.
- Baselines: blurry, inconsistent at slight OOD; unrecognizable at OOD.

### Long-context generation (Minecraft)

- HG-t blends long-context score (memory) + short-context score
  (robustness to OOD): FVD 97.63 → 79.19.

### Long-horizon reactive imitation (Fruit Swapping, Task 3)

- DFoT + HG-t: **83% success rate**.
- Baselines: "fail to perform the task completely."
- Combines long-history score (object position memory) + single-frame
  score (reactivity to disturbance).

### Ultra-long rollout (RealEstate10K, Fig 1)

- Single image → 862-frame navigation video with consistent transitions
  (kitchen → bedroom → outdoors). Many times longer than prior methods.

## Limitations / open questions

- **Compute scaling.** DFoT's per-batch cost is comparable to standard
  diffusion (same architecture), but the training-set size needed for
  long-history conditioning is unstudied. Paper trains separate models
  per dataset; foundation-scale fine-tuning is left to future work.
- **No discrete-token version.** All experiments are continuous-latent.
  Discrete-token videos (VQ-VAE pipelines) would need an analogue of DF.
- **Guidance scale selection.** `ω` and `k_H` are picked empirically.
  No principled selector.
- **Causal variant trade-off.** Paper analyzes a causal adaptation (App
  A.5) but doesn't fully evaluate; the non-causal version is the headline.
  For *streaming* perception/control the causal variant matters more.
- **No comparison with diffusion-forcing CDF directly** under matched
  compute; positioned as a generalization rather than an ablation.

## Connections

- **Method page:** [[dfot]] (created in this ingest).
- **Concept page:** [[history-guidance]] (created in this ingest).
- **Direct predecessor:** [[chen-2024-diffusion-forcing]] — same lead
  (Boyuan Chen); DFoT applies DF to non-causal video DiT.
- **Mechanism cousin:** [[anon-2026-pi-r-squared]] — same per-position
  noise mechanism applied to action chunks; πR² staircase is a special
  case of DFoT's `k_T` choice.
- **Concept updates:**
  - [[diffusion-forcing]] — DFoT is the non-causal DiT instance; HG is
    the guidance family it enables.
  - [[train-inference-mismatch]] — DFoT is "train for the inference
    regime" instance at the video-conditioning level (any history mask
    is in-distribution).
- **Architectural similarity:** DiT with per-frame AdaLN = exactly the
  architecture used by [[training-time-rtc]] and [[pi-r-squared]]'s
  action heads — generalized here to video frames.
- **For the user's middle-ground 3D tracker:** **highly relevant**.
  DFoT's "noise as masking + non-causal DiT + per-frame AdaLN" recipe
  is the closest existing instance of the mechanism the user wants to
  apply to per-query 3D trajectories. The history-guidance composition
  (HG-tf) gives a principled way to mix "long memory" + "fresh
  observation" scores — directly relevant to perception's
  fast/slow channel question. See
  [[q-perception-control-symmetry]] and
  [[q-fast-slow-perception]].
- **Authors:**
  - Kiwhan Song (MIT, lead; equal contribution).
  - [[boyuan-chen]] (MIT, equal contribution; also led
    [[chen-2024-diffusion-forcing]]).
  - Max Simchowitz (CMU, theoretical contributor; also on
    [[chen-2024-diffusion-forcing]]).
  - Yilun Du (Harvard; recurring MIT CSAIL alumni;
    [[chi-2024-diffusion-policy]] coauthor).
  - [[russ-tedrake]] (MIT CSAIL).
  - Vincent Sitzmann (MIT CSAIL).
- **Org:** [[mit-csail]] (most authors) + Carnegie Mellon (Simchowitz)
  + Harvard (Du).

## Citation

Song, K., Chen, B., Simchowitz, M., Du, Y., Tedrake, R., & Sitzmann, V.
(2025). *History-Guided Video Diffusion.* International Conference on
Machine Learning (ICML), PMLR 267. https://arxiv.org/abs/2502.06764
Project: https://boyuan.space/history-guidance
