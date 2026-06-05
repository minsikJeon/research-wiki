---
type: concept
title: Sim-to-Real Gap in Point Tracking
status: growing
tags: [point-tracking, synthetic-data, pseudo-labeling, sim2real]
sources:
  - "[[karaev-2024-cotracker3]]"
  - "[[aydemir-2025-track-on2]]"
  - "[[jung-2026-tapnext-plus-plus]]"
  - "[[zholus-2025-tapnext]]"
related:
  - "[[point-tracking]]"
  - "[[pseudo-labeling-point-tracking]]"
  - "[[kubric-dataset]]"
created: 2026-05-24
updated: 2026-05-24
---

# Sim-to-Real Gap in Point Tracking

## The problem

[[point-tracking]] needs pixel-precise per-frame annotations, which are
prohibitively expensive to collect on real video. Nearly every modern
tracker trains on synthetic Kubric ([[kubric-dataset]]) and hopes the
features and motion priors transfer to real video. They mostly do, but
not perfectly.

## The two responses

**A. Fine-tune on real video with pseudo-labels.**
- BootsTAP (Doersch et al., 2024): student–teacher with EMA, three loss
  masks, augmentation equivariance, **15M unlabeled real clips**.
- BootsTAPNext: BootsTAP recipe applied to [[tapnext]].
- [[cotracker3]]: simpler version — random-teacher distillation,
  no EMA / no augmentation / no synthetic ground-truth mix, **15K
  real clips** beats BootsTAPIR (1000× less data).

**B. Train better on synthetic alone.**
- [[aydemir-2025-track-on2]]: synthetic-only, beats real-data-fine-tuned
  competitors via memory-based architecture + long training clips +
  classification-first head.
- [[jung-2026-tapnext-plus-plus]]: synthetic-only on Kubric-1024
  (1024-frame extension), beats BootsTAPNext on long sequences.

## Where the debate stands (May 2026)

The two camps now both have strong results:

- **Pseudo-label-real camp:** wins when the synthetic distribution is
  narrow vs the deployment distribution. Best on Internet-like content.
- **Better-synthetic camp:** wins when the failure mode is *training
  length* or *augmentation coverage*, not real-distribution mismatch.
  Both [[track-on2]] and [[tapnext-plus-plus]] point in this direction.

Neither has cleanly invalidated the other. A reasonable position: **start
with the best synthetic recipe; only add real-data fine-tuning if a
specific deployment distribution mismatch demands it**.

## Open questions

- Is there a regime where the two are complementary? (Train on long
  Kubric, fine-tune on real?)
- For 3D ([[tapip3d]] / [[spatialtracker-v2]]), the sim-real gap is
  newer and less explored. ST-V2's 17-dataset mix is a partial answer.
- What's the failure mode that the real-data camp uniquely fixes? Need
  per-category analyses (textures, lighting, weather, etc.).
