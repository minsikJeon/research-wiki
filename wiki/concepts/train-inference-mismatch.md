---
type: concept
title: Train / Inference Mismatch (Recurring Pattern)
status: stub
tags: [train-inference-mismatch, online-tracking, action-chunking, streaming-perception]
sources:
  - "[[xiao-2025-spatialtracker-v2]]"
  - "[[black-2025-rtc]]"
  - "[[anon-2026-point4d]]"
related:
  - "[[online-vs-offline-tracking]]"
  - "[[streaming-perception]]"
  - "[[action-chunking]]"
  - "[[trajectory-chaining]]"
  - "[[asynchronous-control]]"
created: 2026-05-28
updated: 2026-05-28
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
| Diffusion / flow VLA control            | Independent action chunks       | Async inference: first `d` actions of new chunk are already committed      | Structural: hard-freeze prefix + inpaint rest         | [[black-2025-rtc]]                   |
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

This is worth tracking as new methods arrive: which camp does the new
method land in?

## Open questions

- **Is there an end-to-end fix that closes the gap during training?**
  Streaming-aware training (the 2020 paper's Appendix C.2 sketch),
  curriculum on noisy carry-over queries, or train-with-the-deployment-
  controller approaches all exist in literature but no source in this
  wiki demonstrates them at scale.
- **Cross-domain transfer of fixes.** Could RTC's soft-masking idea
  apply to sliding-window perception inference? Could the Sim(3)
  alignment idea from [[point4d]] apply to action chunking? Open.
