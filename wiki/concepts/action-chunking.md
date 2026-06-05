---
type: concept
title: Action Chunking (for Robot Policies)
status: growing
tags: [robotics, manipulation, vla, policy, action-chunking, real-time]
sources:
  - "[[black-2025-rtc]]"
related:
  - "[[asynchronous-control]]"
  - "[[vla]]"
  - "[[rtc]]"
  - "[[flow-matching]]"
created: 2026-05-28
updated: 2026-05-28
---

# Action Chunking (for Robot Policies)

## Definition

A control paradigm where the policy outputs **a chunk of `H` future
actions** per inference call, not a single action: `π(A_t | o_t)` where
`A_t = [a_t, ..., a_{t+H−1}]`. Only the first `s ≤ H` actions are
executed before re-querying with a new observation. Originated with ACT
(Zhao et al. 2023) and is now standard in dexterous manipulation VLAs.

## Why it matters

Two related benefits and two related problems:

**Benefits:**
- **Temporal consistency.** A coherent multi-step strategy survives
  observation noise.
- **Throughput amortization.** One slow VLA forward pass produces `s`
  controller steps of actions, amortizing inference cost.

**Problems:**
- **Reactivity loss.** A long execution horizon `s` means stale
  responses to new observations.
- **Mode jumps at chunk boundaries.** Adjacent chunks may pick different
  "strategies" from the learned distribution, producing discontinuities.
  Out-of-distribution accelerations are particularly damaging because
  they shift the dynamics distribution at train vs. test.

## How it's typically handled

- **Long execution horizon (s ≈ H/2).** Default in many works
  (ACT, π0, π0.5, DP). Trades reactivity for stability.
- **Temporal ensembling (ACT).** Average overlapping action predictions
  from multiple chunks. Reduces acceleration but produces invalid
  actions outside the learned distribution.
- **Real-time chunking with inpainting ([[rtc]]).** Pose the boundary
  problem as inpainting: freeze the prefix that's already committed,
  inpaint the rest consistent with the previous chunk + new observation.
  Inference-time only; no retraining.

## Connection to perception-side problems

Action chunking + asynchronous inference has the *same shape* as the
**sliding-window inference with cross-window initialization** problem
we mapped in [[spatialtracker-v2]]:

| Perception                          | Robot control                       |
|-------------------------------------|-------------------------------------|
| Window of frames                    | Chunk of actions                    |
| Overlap region between windows      | Overlap between adjacent chunks     |
| Re-initialize new window's tracks   | Re-initialize new chunk's actions   |
| Train on clean GT queries           | Train on clean ground-truth chunks  |
| Inference uses noisy carry-over     | Inference uses prev-chunk overlap   |
| → [[train-inference-mismatch]]      | → [[train-inference-mismatch]]      |

RTC handles the mismatch explicitly by **freezing the prefix that
inference latency guarantees will execute, then inpainting the rest**.
SpaTrackerV2 handles the analogous problem by **picking the most-
confident overlap frame and synthesizing a fresh query** — implicit and
heuristic, not structural. The contrast is one of the more interesting
cross-domain patterns in the wiki.

## Open questions

- **Optimal execution horizon `s` is task-dependent.** [[rtc]]'s
  Figure 5 shows only RTC and BID benefit from shorter `s`; standard
  chunking penalizes too-short `s`.
- **Inpainting vs. ensembling vs. autoregression.** Three families of
  chunk-boundary solutions; the empirical comparison in RTC is partial.
  A unified evaluation across diffusion / flow / transformer policies
  is still open.
- **Chunking + asynchrony in non-generative VLAs.** RTC requires a
  generative policy. Whether transformer-decoder VLAs need a different
  inference-time mechanism is unsettled.
