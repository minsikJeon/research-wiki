---
type: concept
title: Train / Inference Mismatch (Recurring Pattern)
status: growing
tags: [train-inference-mismatch, online-tracking, action-chunking, streaming-perception]
sources:
  - "[[xiao-2025-spatialtracker-v2]]"
  - "[[black-2025-rtc]]"
  - "[[anon-2026-point4d]]"
  - "[[black-2025-training-time-rtc]]"
  - "[[anon-2026-pi-r-squared]]"
  - "[[chen-2024-diffusion-forcing]]"
related:
  - "[[online-vs-offline-tracking]]"
  - "[[streaming-perception]]"
  - "[[action-chunking]]"
  - "[[trajectory-chaining]]"
  - "[[asynchronous-control]]"
  - "[[diffusion-forcing]]"
  - "[[fast-slow-policy]]"
created: 2026-05-28
updated: 2026-06-08
---

# Train / Inference Mismatch (Recurring Pattern)

## Definition

A pattern observed across multiple sub-fields in the wiki: the model is
**trained on clean, idealized inputs**, but at inference is given
**noisy carry-over or asynchronously-delayed inputs**. The training
distribution and the deployment distribution disagree, and the model is
not explicitly trained to bridge the gap.

## Why it matters

This concept page exists to **link the analogous patterns across
perception and control** so the wiki captures them as one phenomenon:

| Domain                                  | Trained on                     | Deployed on                                                                 | How it's handled                                       | Source                              |
|-----------------------------------------|--------------------------------|------------------------------------------------------------------------------|--------------------------------------------------------|--------------------------------------|
| Online 3D point tracking                | Clean GT query + zero-motion init | Carry-over query synthesized from previous window's most-confident frame    | Heuristic (max `vis × conf` frame selection)           | [[xiao-2025-spatialtracker-v2]]      |
| Long-video 4D reconstruction            | Single-window 4D                | Multi-window chaining with depth-error compounding                          | Structural: 3D-coordinate queries, Sim(3) chain       | [[anon-2026-point4d]]                |
| Diffusion / flow VLA control (inference-time fix) | Independent action chunks       | Async inference: first `d` actions of new chunk are already committed      | Structural: hard-freeze prefix + ΠGDM inpaint rest    | [[black-2025-rtc]]                   |
| Diffusion / flow VLA control (training-time fix) | Sampled `d`, prefix clamped clean | Same — but with no inference-time overhead | Structural: per-position τ, masked loss | [[black-2025-training-time-rtc]] |
| Diffusion / flow VLA control (joint fix) | Sampled `d` + sampled `d_vlm` | Per-tick denoising + async cached vision | Structural: staircase schedule + slow/fast split | [[anon-2026-pi-r-squared]] |
| Sequence generation (general) | Shared noise level across tokens | Streaming, partial observation, mixed-staleness | Structural: per-token independent noise | [[chen-2024-diffusion-forcing]] |
| Streaming object detection              | Per-frame ground-truth          | Output stream evaluated against current world state at every instant       | Pipeline: forecaster + dynamic scheduler              | [[li-2020-streaming-perception]]     |

## Two response strategies observed

1. **Heuristic / engineering at inference.** The model itself is
   unchanged; engineering bridges the gap.
   - SpaTrackerV2's confidence-weighted overlap-frame selection.
   - Streaming-perception's asynchronous Kalman forecaster.
   - Temporal ensembling (ACT) for action chunking.
2. **Structural / representational.** The interface that the model
   exposes is changed so train and inference use the same regime, even
   if training data is still clean.
   - Point4D's 3D-coordinate queries: queries are well-defined
     under occlusion in both training and inference.
   - RTC's inpainting: training is unchanged but inference is reframed
     so the previous chunk's actions become inpainting targets.

## Conjecture

Across both domains, the **structural fixes tend to win** at the cost
of one design iteration. The heuristic fixes accumulate hyperparameters
and are brittle (replace_ratio, confidence-weighting, decay schedules).
The structural fixes (3D queries, inpainting, query-based decoders) are
cleaner but require rethinking the model's input or output interface.

Within the structural-fix camp, a **further trajectory** is visible:
fixes migrate from inference-time → training-time → joint
(training + architecture). RTC family is the cleanest example:

- [[rtc]] (inference-time inpainting) — keeps training unchanged; pays
  per-step ΠGDM cost.
- [[training-time-rtc]] (training-time conditioning) — same interface,
  no inference cost, slightly less flexible.
- [[pi-r-squared]] (joint structural fix) — generalizes RTC's prefix
  clamp via diffusion forcing + adds a fast-slow channel split for
  modality-staleness.

Each step *eats* the previous step's overhead. The same trajectory
hasn't yet played out on the perception side — SpaTrackerV2 sits at the
*heuristic* corner; Point4D sits at *inference-time structural*. The
*training-time* and *joint* corners are open research directions for
streaming 3D / 4D.

## Open questions

- **Is there an end-to-end fix that closes the gap during training?**
  Streaming-aware training (the 2020 paper's Appendix C.2 sketch),
  curriculum on noisy carry-over queries, or train-with-the-deployment-
  controller approaches all exist in literature but no source in this
  wiki demonstrates them at scale.
- **Cross-domain transfer of fixes.** Could RTC's soft-masking idea
  apply to sliding-window perception inference? Could the Sim(3)
  alignment idea from [[point4d]] apply to action chunking? Open.
