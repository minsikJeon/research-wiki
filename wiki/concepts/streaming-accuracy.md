---
type: concept
title: Streaming Accuracy (Metric)
status: stub
tags: [streaming-perception, latency, evaluation, metric]
sources:
  - "[[li-2020-streaming-perception]]"
related:
  - "[[streaming-perception]]"
  - "[[online-vs-offline-tracking]]"
created: 2026-05-28
updated: 2026-05-28
---

# Streaming Accuracy (Metric)

## Definition

A latency-aware evaluation metric that **jointly captures accuracy and
latency in a single score**. The algorithm's output stream `{(ŷ_j, s_j)}`
is matched against the ground-truth stream `{(y_i, t_i)}` by, at every
ground-truth timestamp `t_i`, comparing to the *most recent* output
produced strictly before `t_i`.

## Formal definition (from [[li-2020-streaming-perception]])

Define `φ(t) := argmax_j (s_j < t)` — the index of the most recent
output available at time `t`. Equivalent to zero-order-hold of the
algorithm's output stream. Then for any single-frame loss `L`:

```
L_streaming = L({(y_i, ŷ_{φ(t_i)})})_{i=1..T}
```

The benefit: the same single-frame metric can be lifted to a streaming
metric, so detection-AP, segmentation-IoU, 3D-tracking-AJ etc. all become
streaming-compatible without redefining the per-frame loss.

## Key properties

- **Forces forecasting.** A finite-time algorithm cannot use the current
  frame `x_t` to produce a prediction at time `t`, so it must predict
  forward from its last-processed observation. This is *built into*
  the metric, not a separate requirement.
- **Hardware-dependent.** The output timestamps `s_j` depend on
  algorithm runtime, which depends on hardware. The 2020 paper proposes
  a simulation harness that abstracts hardware into per-module
  runtimes.
- **Evaluates the stack.** Compares modular pipelines vs. end-to-end
  models on equal footing. Critical for embodied perception where
  modular pipelines are common.
- **Extensible with downstream lookahead.** Replace `t_i` with
  `t_i + η` in the comparison to also account for downstream-actuator
  latency `η`. The benchmark then asks the perception stack to
  forecast `η` further into the future.

## Open questions

- **Variance from runtime stochasticity** is small (≤0.5 % std in the
  2020 setup) but may grow on modern non-deterministic GPU pipelines
  (e.g. JIT compilers, dynamic batching).
- **Streaming AP vs streaming sAP**: later literature has fragmented
  terminology. The 2020 paper's `L_streaming` is essentially what
  later work calls "sAP" for detection.
