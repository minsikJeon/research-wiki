---
type: concept
title: Vision-Language-Action Models (VLAs)
status: growing
tags: [vla, robotics, manipulation, foundation-model, multimodal]
sources:
  - "[[zhao-2023-act]]"
  - "[[chi-2024-diffusion-policy]]"
  - "[[black-2025-rtc]]"
  - "[[black-2025-training-time-rtc]]"
  - "[[anon-2026-pi-r-squared]]"
  - "[[kim-2026-pri4r]]"
related:
  - "[[action-chunking]]"
  - "[[asynchronous-control]]"
  - "[[rtc]]"
  - "[[training-time-rtc]]"
  - "[[pi-r-squared]]"
  - "[[pri4r]]"
  - "[[diffusion-forcing]]"
  - "[[fast-slow-policy]]"
  - "[[flow-matching]]"
  - "[[train-inference-mismatch]]"
  - "[[point-tracks-as-manipulation-interface]]"
created: 2026-05-28
updated: 2026-06-27
---

# Vision-Language-Action Models (VLAs)

## Definition

A class of foundation models that **take vision + language inputs and
produce robot actions**. Architecturally usually a VLM (vision-language
model) decoder repurposed to output action tokens or action chunks
instead of text tokens. Examples: π0/π0.5/π0.6, OpenVLA, RT-2,
GR00T-N1.7.

## Why they matter

VLAs are the current dominant paradigm for **generalist manipulation
policies**. Trained on large multi-task multi-robot demonstration data
plus optionally co-trained on language and vision data, they aim to be
the robotics analogue of LLMs.

## Anatomy of a modern VLA

Three-stage pipeline:

```
       ┌─────────────────┐   ┌──────────────────┐   ┌──────────────┐
       │     INPUTS      │ → │  VLM BACKBONE    │ → │ ACTION HEAD  │ → actions
       │  image(s)       │   │  ~1B–7B params   │   │  ~10–100M    │
       │  language       │   │  (the "brain",   │   │  (the "hand",│
       │  proprio        │   │   slow ~60 ms)   │   │   fast)      │
       └─────────────────┘   └──────────────────┘   └──────────────┘
```

- **Inputs:** multi-view RGB (~30 Hz), text instruction (static),
  proprioception (~500–1000 Hz).
- **VLM backbone:** pretrained vision-language model (PaliGemma, SigLIP
  + Llama, Qwen2-VL); usually fine-tuned, sometimes frozen.
- **Action head:** small fast model converting the VLM latent into an
  H-step action chunk (H ≈ 8–16).

## Two action-head families

| Family | Examples | Mechanism |
|---|---|---|
| **Autoregressive token head** | RT-2, OpenVLA | tokenize actions; decode one at a time |
| **Diffusion / flow-matching head** | π0, π0.5, π0.6, GR00T-N1.7, Diffusion Policy | denoise a chunk of continuous actions |

The flow-matching family is where [[rtc]] / [[training-time-rtc]] /
[[pi-r-squared]] do their work — they exploit the chunk-denoising
substrate to inject prefix conditioning and per-position noise schedules.

## Why latency is the central VLA problem

- **Large parameter counts (3B–7B+)** → high per-call latency
  (>100 ms typical) → robot executes ~7 actions open-loop while the
  next chunk denoises. See [[asynchronous-control]] and
  [[train-inference-mismatch]].
- **Action chunking is standard** to amortize policy cost across many
  control ticks. See [[action-chunking]].
- The full real-time-control story can be read as **a sequence of
  latency mitigations** ranging from the inference-time RTC fix to the
  architectural [[fast-slow-policy]] split in πR².

## The real-time chunking lineage (this wiki's spine)

```
ACT (2023, chunking origin)
   │
Diffusion Policy (2024, DDPM action head)
   │
RTC (2025, inference-time ΠGDM inpainting)
   │
Training-Time RTC (2025, prefix-clamp at training)
   │
πR² (2026, diffusion-forcing staircase + async slow/fast split)
```

Each step **closes a different mismatch** between training-time
denoising assumptions and inference-time chunking reality. See
[[train-inference-mismatch]] for the cross-cutting framing.

## Output parameterizations

- **Discrete action tokens** (RT-2, OpenVLA) — tokenize continuous
  actions into a fixed vocabulary; decode AR.
- **Continuous regression** — direct MLP over backbone features.
- **Conditional flow matching** (π0/π0.5/π0.6) — denoise from noise
  conditioned on observation; same loss as [[diffusion-policy]] in a
  continuous-time formulation.
- **Diffusion** ([[diffusion-policy]] family).

[[rtc]] applies to the diffusion / flow family without retraining;
[[training-time-rtc]] and [[pi-r-squared]] require fine-tuning but
recover the inference cost.

## Examples cited in the wiki (not yet ingested as primary sources)

- **π0 / π0.5 / π0.6** (Black et al. 2024–2025; [[physical-intelligence]])
  — flow-matching VLAs; π0.5 is the base for [[black-2025-rtc]] and
  π0.6 is the base for [[black-2025-training-time-rtc]].
- **GR00T-N1 / N1.7** ([[nvidia]] 2025–2026) — DiT-based VLA; base
  policy fine-tuned by [[anon-2026-pi-r-squared]].
- **OpenVLA** (Kim et al. 2024) — 7B open-source VLA; 321 ms on A100.
- **RT-2** (Brohan et al. 2023) — Google's earlier tokenized VLA.

## Open questions

- **Architectures beyond decoder-only VLAs.** Mixture-of-experts,
  retrieval-augmented, action-flow hybrid — all open.
- **Latency-aware training.** [[anon-2026-pi-r-squared]] establishes
  that vision staleness can be made in-distribution during training
  (`d_vlm` scalar embedding). Generalizing this to *external* latency
  (network, robot communication) is open.
- **Composing VLAs with feed-forward 3D / 4D perception** — high-value
  ingest area given the user's MSR program. Direct candidates:
  Point4D-as-perception-backbone + πR²-style action head.
- **VLA size scaling laws** — does the 7-Hz ↔ 25-Hz gap close at a
  particular VLM size, or does fast-slow remain mandatory at scale?
- **VLA action heads beyond flow / DDPM.** Streaming variants
  ([[diffusion-forcing]]-style), implicit Q-learning, energy-based
  heads — open.

## Auxiliary supervision (training-time geometry)

[[kim-2026-pri4r]] (Pri4R, 2026) adds a lightweight 3D point-track
head to π₀ / π₀.₅ / OpenVLA-OFT during fine-tuning, predicting per-step
3D point displacements as **privileged supervision**. At inference the
head is discarded — zero overhead, identical architecture. Gains:
+13.2% RoboCasa for OpenVLA-OFT, +9.8% on LIBERO-Long, +1–4% across
the π family.

Ablation finding: among supervision targets, 3D point tracks beat
goal-only, 2D tracks, and depth maps (+13.2% vs +0.7 / +3.9 / +8.3).
The signal must be **temporally dense** + **metric 3D** + **identity-
registered across time**; depth lacks the last property and goal-only
lacks the first. This is the lowest-overhead way to inject 4D scene
dynamics into a VLA. See [[point-tracks-as-manipulation-interface]]
for the broader family of methods using point tracks as a manipulation
representation.
