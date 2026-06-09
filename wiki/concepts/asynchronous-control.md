---
type: concept
title: Asynchronous Control (under Inference Delay)
status: growing
tags: [robotics, real-time, latency, vla, control, asynchronous-control]
sources:
  - "[[black-2025-rtc]]"
  - "[[black-2025-training-time-rtc]]"
  - "[[anon-2026-pi-r-squared]]"
related:
  - "[[action-chunking]]"
  - "[[rtc]]"
  - "[[training-time-rtc]]"
  - "[[pi-r-squared]]"
  - "[[fast-slow-policy]]"
  - "[[diffusion-forcing]]"
  - "[[vla]]"
  - "[[streaming-perception]]"
created: 2026-05-28
updated: 2026-06-08
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

| Variant | Per-call cost | Reactivity | Source |
|---|---|---|---|
| Synchronous (`δ ≤ Δt`) | Trivial | OK for small models | — |
| Synchronous (`δ > Δt`) with pauses | High | Visible pauses, dyn. shifts | Default |
| Naive asynchronous | One full denoise | Mode jumps at boundaries | — |
| Temporal ensembling (ACT) | One full denoise | Smoother, invalid actions | [[zhao-2023-act]] |
| Inference-time RTC (inpainting) | One denoise + ΠGDM VJP | Robust to ~300 ms delay | [[black-2025-rtc]] |
| Training-time RTC (prefix conditioning) | One denoise (no VJP) | ≥ inference-time RTC at d ≥ 2 | [[black-2025-training-time-rtc]] |
| **πR² (diffusion forcing + fast/slow)** | **One Euler substep** | Per-control-tick (25 Hz on 7 Hz VLA) | [[anon-2026-pi-r-squared]] |

The trajectory across these variants is clear:
- Inference-time only → training-time → joint training + architectural
  split.
- Each step **moves the same fix earlier in the pipeline**, with less
  overhead and more reactivity.
- πR² combines all three:
  - Training-time prefix clamp (from training-time RTC).
  - Diffusion-forcing ramped interior (from CDF).
  - Slow/fast channel split (new contribution).

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
