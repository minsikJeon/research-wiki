---
type: concept
title: Fast / Slow Policy Architecture (Hierarchical or Asymmetric Conditioning)
status: growing
tags: [robotics, vla, control, latency, hierarchical-policy, fast-slow-policy, asynchronous-control]
sources:
  - "[[anon-2026-pi-r-squared]]"
related:
  - "[[asynchronous-control]]"
  - "[[action-chunking]]"
  - "[[vla]]"
  - "[[diffusion-forcing]]"
  - "[[train-inference-mismatch]]"
created: 2026-06-08
updated: 2026-06-08
---

# Fast / Slow Policy Architecture

## Definition

A control architecture that **splits policy inference into two
asynchronous channels** running at different rates:

- **Slow channel** (low frequency, high-bandwidth representation):
  vision-language understanding, scene context, task plan. Updated at
  the timescale of large-model inference (10 Hz or less).
- **Fast channel** (high frequency, low-dimensional state):
  proprioception, joint state, contact forces. Updated every control
  tick (~50–1000 Hz).

The action head conditions on **both channels simultaneously**: fresh
fast state + cached slow representation. Slow refreshes asynchronously
without blocking the control loop.

## Why it matters

This is the **architectural answer to a quantitative gap**: modern
VLAs have per-call latency of 100–500 ms (one or more orders of
magnitude slower than the control loop). Pure synchronous inference
forces an open-loop window of that size, which destroys reactivity
on contact-rich tasks (slip recovery, dynamic grasping, scrape-pickup).

There are two common architectural responses:

1. **Two-model hierarchy** (System 2 / System 1): separate large
   planner + small policy, e.g., Helix, Gemini Robotics, GR00T-N1.
   Slow planner runs in a separate process; fast policy consumes its
   output.
2. **One-model asymmetric conditioning** (πR²): single VLA with two
   conditioning channels internal to the model — slow VLM features
   cached + fast proprioception fresh per call. **No separate model**,
   no inter-process communication.

[[anon-2026-pi-r-squared]] is the first deployment of strategy (2) at
VLA scale and the first instance in this wiki.

## Variants observed

| Variant | Slow channel | Fast channel | Slow latency tolerated | Source |
|---|---|---|---|---|
| Helix | VLM (separate process) | Small visuomotor policy | ~100 ms | secondary (cited) |
| Gemini Robotics | Gemini planner | RL policy | ~100 ms | secondary (cited) |
| GR00T-N1 | VLM planner | Diffusion action head | ~100 ms | secondary (cited) |
| **πR² (πR-style async)** | VLM features inside same model, cached | Proprio MLP every tick | trained with `d_vlm ∼ Unif{0,...,d_max}` | [[anon-2026-pi-r-squared]] |
| Streaming Diffusion Policy | (small model, no split) | (small model, no split) | N/A | secondary (cited) |

## Key design considerations

- **Which inputs go in which channel?** Proprio is "naturally fast"
  because it's low-dim and inexpensive. Vision/language is "naturally
  slow" because of preprocessing + VLM forward. Putting them on
  different rates exploits the asymmetry.
- **How is slow-channel staleness made in-distribution?** πR² uses a
  learned scalar embedding of `d_vlm` (slow channel delay in control
  ticks) added to the slow representation, with `d_vlm` randomized
  during training. The model literally has a "this feature is N ticks
  stale" input.
- **What does the fast channel need to do?** Local reactive
  corrections: contact response, slip compensation, force matching.
  Vision/language provides global context for coarse motion — the
  fast path just needs to refine the existing plan.

## Connection to [[diffusion-forcing]]

πR² combines fast-slow with diffusion forcing: the two solve
**orthogonal staleness problems**:

- *Modality staleness* (vision 60 ms behind) → fast-slow split.
- *Temporal staleness within the action chunk* (front actions already
  committed) → diffusion forcing schedule.

Both are required for closed-loop control on a slow VLA. Either alone
leaves a gap.

## Connection to perception side

Conceptually, fast-slow control mirrors **hybrid memory** in
long-context perception ([[zhang-2026-loger]] SWA + TTT): lossless
local representation (fast / SWA) + compressed global representation
(slow / TTT). Both architectures split a single sequence model into
two memory paths operating at different timescales / bandwidths. The
parallel is sharper than it first appears.

## Open questions

- **How to allocate compute between slow and fast?** πR² runs slow at
  whatever rate the VLM supports; no principled compute budget
  partitioning yet.
- **Multi-rate beyond two channels?** Plausible to add a "medium"
  channel (e.g., depth at 10 Hz, vision at 5 Hz, proprio at 50 Hz).
  No source in this wiki yet.
- **End-to-end training with simulated staleness across all channels.**
  πR² randomizes vision delay during training; what about depth /
  audio / tactile staleness?
- **Generalization to non-VLA architectures.** Fast-slow as a
  paradigm probably applies broadly (e.g., perception-fast +
  reasoning-slow), but the architecture has only been shown for VLA
  control so far.
