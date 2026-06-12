---
type: source
source_type: paper
title: "Test-Time Training Done Right"
authors:
  - Zhang, Tianyuan
  - Bi, Sai
  - Hong, Yicong
  - Zhang, Kai
  - Luan, Fujun
  - Yang, Songlin
  - Sunkavalli, Kalyan
  - Freeman, William T.
  - Tan, Hao
year: 2025
venue: arXiv (cs.LG)
url: https://arxiv.org/abs/2505.23884
raw_path: papers/2505.23884v1.pdf
status: ingested
tags: [test-time-training, fast-weights, long-context, sequence-modeling, novel-view-synthesis, language-modeling, video-diffusion, diffusion-forcing, muon-optimizer]
related:
  - "[[test-time-training]]"
  - "[[sun-2024-ttt]]"
  - "[[lact]]"
  - "[[diffusion-forcing]]"
  - "[[feed-forward-3d-reconstruction]]"
created: 2026-06-12
updated: 2026-06-12
---

# Test-Time Training Done Right

## TL;DR

Existing [[test-time-training|TTT]] methods waste GPUs: they update the
fast weights every 16–64 tokens, hitting <5% peak FLOPS utilization.
This paper flips the strategy — **Large-Chunk TTT (LaCT)** updates the
fast weights once per chunk of 2K–1M tokens. This unlocks (i) hardware
utilization ≥70% in pure PyTorch (no custom kernels), (ii) nonlinear
SwiGLU-MLP fast weights up to **40% of model parameters**, and (iii)
Muon-optimizer test-time updates. Validated across three modalities —
novel view synthesis with 1M-token contexts, language modeling, and a
**14B-parameter autoregressive video diffusion** model handling 56K visual
tokens.

## Why it matters

For the user's wiki, LaCT is the missing efficiency layer underneath
the TTT-for-3D papers ([[loger]], [[vgg-t3]], TTT3R). It also lands a
**14B-parameter autoregressive video diffusion** model on top of TTT
+ sliding window — directly relevant to your middle-ground 3D-tracker
chunk-denoiser slot (which currently leans on [[dfot]]). LaCT's NVS
experiment achieves what amounts to a **streaming
1M-token-context** scene encoder — the kind of memory backbone CUT3R
and LoGeR are trying to be.

The paper also explicitly does autoregressive video diffusion with
"diffusion-forcing style" frame-independent noise (§4.3, Eq 12), which
slots into the [[diffusion-forcing]] family alongside [[dfot]] and
[[chen-2024-diffusion-forcing]].

## Key claims

- **Chunk size = TPS/utilization knob (Fig 1a).** Per-token TTT-MLP gets
  ~57 TFLOPS on H100; LaCT (large chunks) gets 515 TFLOPS — a 9×
  throughput gain at *higher* state capacity (§2.2).
- **Compute-to-memory ratio bound (Eq 3).** For an MLP fast weight,
  `r = 2h²b / (2h² + 4hb) ≤ min(h/2, b)`. Small chunk size `b` makes the
  operation memory-bound — this is the TFLOPS-utilization story.
- **Nonlinear fast weights work and outperform linear** (§5.4, Fig 8a)
  even with smaller state size (SwiGLU 6d² beats Linear 9d²).
- **Muon optimizer for test-time updates outperforms GD and Momentum**
  (Fig 7b). Muon normalizes the spectral norm via Newton–Schulz —
  better-conditioned online gradient steps.
- **State size up to 40% of model parameters** with measured monotonic
  improvement (Fig 7a). Previous TTT methods hovered at 0.1%–5%.
- **NVS scaling**: 1.3B-parameter model trained on 1.8T tokens at up to
  1M-token sequence length (48 input views at 960×536). Outperforms 3D
  Gaussian Splatting up to 128 input views (Fig 4c).
- **AR video diffusion**: a 14B-parameter bidirectional Wan 2.1 T2V model
  is **fine-tuned** into an autoregressive video diffusion model by
  swapping bidirectional attention layers for LaCT + sliding window.
  Generates consistent videos up to **56K visual tokens** / 14s (§5.3).
- **Hybrid causality (Fig 3).** The Update–Apply order in LaCT gives
  block-wise causal / shifted block-wise causal / strided block-wise
  causal masks. The same attention-pattern menu CDF / DFoT use.
- **Window attention + LaCT layer** = hybrid architecture: quadratic
  local attention (intra-chunk locality) + linear LaCT (inter-chunk
  global memory).

## Methods

LaCT block has three layers: (1) **window attention** (causal or
bidirectional, captures intra-chunk locality), (2) **large-chunk TTT
layer** (fast weight `W` of a SwiGLU MLP, updated per chunk), (3)
**feed-forward** (channel mixing). Architecture: Fig 2.

**Per-chunk update rule (Eqs 4–9):**

```
g = ∇_W Σ_i η_i L(f_W(k_i), v_i)         # chunk-summed gradient
W ← weight-update(W, g)
W ← L2Normalize(W − Muon(g))             # Muon variant
o_i = f_W(q_i)                            # apply on every query in chunk
```

Fast weight `f_W(x) = W₂ [SiLU(W₁ x) ∘ (W₃ x)]` (SwiGLU MLP, three
matrices). Loss is dot-product attention loss `L = −f_W(k_i)^T v_i`.
Weight normalization (L2 over input dimension) prevents magnitude
explosion across many chunks.

**Three task instantiations (Table 1):**

| Task | Chunk size | State size / model | Max length |
|---|---|---|---|
| Novel view synthesis | full sequence | `6d²` | 1M tokens |
| AR video diffusion | 3 frames | `3d²`, `0.75d²` (1.3B, 14B) | 56,160 |
| Language modeling | 2K–4K tokens | `0.75d²` | 32,768 |

For NVS, the **apply** step resembles a parallel decode (rendering novel
views); **update** resembles prefill (encoding input views). For AR video
diffusion, fast weights only update on **clean** frames; noisy frames
only apply (Eq 12) — preserving the diffusion-forcing teacher-forcing
recipe.

## Results

### Novel View Synthesis (Fig 4, §5.1)

- **GSO 256×256 / 512×512**: matches/beats full-attention LVSM at
  4–48 views; clearly beats Perceiver-style register attention.
- **DL3DV 960×536 scene set (Fig 4c)**: beats 3D Gaussian Splatting up
  to 128 input views (1M tokens), where 3DGS plateaus due to optimization
  difficulty.
- **Inference (Table 2, 48 × 512² input)**: prefill compute O(1) vs full
  attention's O(n²); LaCT runs at 38.7 FPS vs full attention 2.3 FPS at
  1.4 s vs 16.1 s prefill.

### Language Modeling (Fig 5)

- 760M / 3B model, 32K context. **Lower per-position loss at large token
  indices** vs GLA, DeltaNet at both scales (Fig 5a, c) — long-context
  modeling wins.
- **S-NIAH-1 / S-NIAH-2 retrieval**: better at 32K than GLA SWA and
  DeltaNet SWA. Muon variant > Momentum variant.

### AR Video Diffusion (Fig 6, §5.3)

- Wan 2.1 14B text-to-video → AR via LaCT replacement of bidirectional
  attention. Window size = 2 AR chunks (~6 frames).
- **Comparable validation denoising loss to full-attention baseline.**
- **Outperforms Mamba-SWA and SWA-alone**, and generalizes to longer
  videos than training horizon (Fig 6c — 12-chunk eval, trained on 7).

### Ablations (Figs 7, 8)

- **Larger state size monotonically wins** for NVS and LM up to 12d²
  (40% of model). Tested d=768, NVS PSNR 32 → 38 as state grows.
- **Muon ≫ Momentum > Vanilla GD** (Fig 7b).
- **Nonlinear SwiGLU > linear fast weights** at the same or smaller state
  (Fig 8a) — capacity beats raw memory.
- **Large-chunk recurrence > per-token recurrence** at matched state size
  (Fig 8b). The per-image NVS task is naturally chunked; even for LM
  where chunks aren't intrinsic, LaCT wins when combined with nonlinear
  state + Muon.

## Limitations / open questions

- **No rotational invariance.** SwiGLU + linear fast weight doesn't
  inherit RoPE-style invariance softmax attention has (§7).
- **Three tasks isn't enough.** Unposed 3D reconstruction (no input
  poses) explicitly *not* attempted — the NVS experiment assumes camera
  poses.
- **Reasoning unexplored.** State-based models historically weak on
  reasoning; LaCT didn't have the budget to test this.
- **Video diffusion scalability metric.** Validation denoising loss is
  the only signal — no FVD or human eval.
- **Chunk size selection.** Empirically picked per task (2K for LM,
  full-sequence for NVS, 3-frame for video). No principled selector.

## Connections

- **[[test-time-training]]** — LaCT is the efficiency / scaling answer
  to the original [[sun-2024-ttt]] line. The wiki's TTT page already
  describes per-token (TTT3R), per-chunk (LoGeR), per-scene (VGG-T3)
  granularities; LaCT formalizes the chunk-size knob and shows nonlinear
  state + Muon make the chunk regime dominant.
- **[[sun-2024-ttt]]** — the foundational TTT-Linear / TTT-MLP paper.
  LaCT cites it (ref [2]) and explicitly extends with much larger chunk
  size and nonlinear SwiGLU.
- **[[loger]] / [[vgg-t3]]** — both use TTT for 3D reconstruction and pick
  empirical chunk sizes. LaCT validates the design choice that large
  chunks work better than per-token. LoGeR's "scene-fit then transfer"
  is a per-chunk TTT instance.
- **[[diffusion-forcing]]** — LaCT's AR video diffusion (§4.3) is
  explicitly diffusion-forcing-style: per-frame independent noise on
  noisy chunks, only-update on clean chunks. Joins [[chen-2024-diffusion-forcing]],
  [[dfot]], [[pi-r-squared]], [[training-time-rtc]] in the
  diffusion-forcing family.
- **[[dfot]]** — DFoT is non-causal full attention; LaCT video model is
  causal large-chunk TTT + SWA. Two different efficiency strategies for
  the same AR-video-diffusion target.
- **Cross-thread for user's design doc:** LaCT is the missing
  "test-time-trained streaming backbone" alternative to CUT3R for the
  middle-ground 3D-tracker. CUT3R uses a fixed-size token bank updated
  by attention; LaCT uses a SwiGLU MLP whose weights *are* the memory,
  updated by Muon. Both are constant per-frame, but LaCT scales state
  capacity to 40% of params; CUT3R doesn't.

## Citation

Zhang, T., Bi, S., Hong, Y., Zhang, K., Luan, F., Yang, S.,
Sunkavalli, K., Freeman, W. T., & Tan, H. (2025). *Test-Time Training
Done Right.* arXiv:2505.23884. https://arxiv.org/abs/2505.23884
