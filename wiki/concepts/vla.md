---
type: concept
title: Vision-Language-Action Models (VLAs)
status: stub
tags: [vla, robotics, manipulation, foundation-model, multimodal]
sources:
  - "[[black-2025-rtc]]"
related:
  - "[[action-chunking]]"
  - "[[asynchronous-control]]"
  - "[[rtc]]"
  - "[[flow-matching]]"
created: 2026-05-28
updated: 2026-05-28
---

# Vision-Language-Action Models (VLAs)

## Definition

A class of foundation models that **take vision + language inputs and
produce robot actions**. Architecturally usually a VLM (vision-language
model) decoder repurposed to output action tokens or action chunks
instead of text tokens. Examples: π0, π0.5, OpenVLA, RT-2.

## Why they matter

VLAs are the current dominant paradigm for **generalist manipulation
policies**. Trained on large multi-task multi-robot demonstration data
plus optionally co-trained on language and vision data, they aim to be
the robotics analogue of LLMs.

## Properties relevant to this wiki

- **Large parameter counts** (3B–7B+) → high inference latency
  (>100 ms typical) → asynchronous-control regime is forced. See
  [[asynchronous-control]] and [[rtc]].
- **Action chunking is standard.** Most VLAs output H-step action
  chunks (H ≈ 8–16) per inference call. See [[action-chunking]].
- **Output parameterizations vary.** Discrete action tokens (RT-2,
  OpenVLA), continuous regression heads, conditional flow matching
  (π0, π0.5), diffusion policies. [[rtc]] applies to the
  diffusion / flow family without retraining.

## Examples cited in the wiki (not yet ingested as sources)

- **π0** (Black et al. 2024) — Physical Intelligence's first VLA;
  conditional flow matching.
- **π0.5** (Black et al. 2024) — successor; base policy for the [[rtc]]
  experiments.
- **OpenVLA** (Kim et al. 2024) — 7B open-source VLA; optimized for
  inference, still 321ms on A100.
- **RT-2** (Brohan et al. 2023) — Google's earlier VLA; precursor.

## Open questions

- **Architectures beyond decoder-only VLAs.** Mixture-of-experts,
  retrieval-augmented, action-flow hybrid — all open.
- **Latency-aware training.** Standard VLA training is offline /
  open-loop; training with closed-loop latency awareness is the
  perception-side analogue of [[streaming-perception]]'s end-to-end
  Streamer.
- **Composing VLAs with feed-forward 3D / 4D perception** is an obvious
  direction the wiki hasn't yet seen a source on. Likely a high-value
  ingest area given the user's MSR program.
