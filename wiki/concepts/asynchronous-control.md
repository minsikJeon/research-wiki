---
type: concept
title: Asynchronous Control (under Inference Delay)
status: stub
tags: [robotics, real-time, latency, vla, control]
sources:
  - "[[black-2025-rtc]]"
related:
  - "[[action-chunking]]"
  - "[[rtc]]"
  - "[[vla]]"
  - "[[streaming-perception]]"
created: 2026-05-28
updated: 2026-05-28
---

# Asynchronous Control (under Inference Delay)

## Definition

A control regime where the policy's inference happens **concurrently
with action execution**. The next chunk / action is generated while the
previous one is still being executed, so by the time the new chunk is
available, the controller is already `d` steps into the future
(`d := ⌊δ / Δt⌋` controller timesteps).

## Why it matters

Modern VLAs have **inference latency comparable to or larger than the
controller period**:

- π0 (3B parameters, RTX 4090): 46 ms KV-cache prefill alone, target
  50 Hz control (`Δt = 20 ms`).
- OpenVLA-7B (A100, optimized): 321 ms inference.
- Mobile remote inference: +13–20 ms network latency in good conditions.

So `δ > Δt` is the normal operating regime, not the exception.
Asynchrony is forced.

## Variants

- **Synchronous (`δ ≤ Δt`).** Trivial: each chunk is generated between
  controller ticks. Only feasible for small models.
- **Synchronous (`δ > Δt`), with pauses.** Default in many prior works
  but produces visible pauses and dynamics shifts.
- **Naive asynchronous.** Start inference at step `s − d` so the new
  chunk lands at step `s`. But transitions between `a_{s−1|0}` and
  `a_{s|s−d}` are unconstrained — produces mode jumps (Fig 2 of
  [[black-2025-rtc]]).
- **Temporal ensembling (ACT, Zhao 2023).** Average overlapping chunks.
  Smooths but produces invalid actions.
- **[[rtc]] (Black et al. 2025).** Freeze the `d` actions guaranteed to
  execute; inpaint the rest with soft-masked guidance from the previous
  chunk. Inference-time only.

## Relationship to streaming perception

Asynchronous control is the **control-side analogue of
[[streaming-perception]]**. Both reckon with the fact that finite
inference time means *the world has moved on by the time the model
finishes*. Perception solves this with forecasting + zero-order hold;
control solves it with action chunking + inpainting.

A unified framework that handles both ends (perception + control under
shared latency budget) is an obvious open direction. The closest the
wiki has is the implicit perception-pipeline forecasting in
[[li-2020-streaming-perception]], but no source ingested so far has
the closed-loop view across both.
