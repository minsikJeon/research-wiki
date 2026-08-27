---
type: entity
entity_type: person
name: Homanga Bharadhwaj
status: growing
tags: [manipulation, point-tracking, imitation-learning, web-video]
sources:
  - "[[bharadhwaj-2024-track2act]]"
  - "[[bharadhwaj-2026-motionforesight]]"
  - "[[soraki-2026-objectforesight]]"
related:
  - "[[cmu-ri]]"
  - "[[johns-hopkins]]"
  - "[[university-of-washington]]"
  - "[[meta-ai]]"
  - "[[shubham-tulsiani]]"
  - "[[roozbeh-mottaghi]]"
  - "[[track2act]]"
  - "[[motionforesight]]"
  - "[[objectforesight]]"
  - "[[point-tracks-as-manipulation-interface]]"
created: 2026-06-27
updated: 2026-08-27
---

# Homanga Bharadhwaj

## Affiliation

- **Now:** [[johns-hopkins]] — **Brains, Bots, and Behavior Lab**
  (faculty/lead), per [[bharadhwaj-2026-motionforesight]].
- **Previously:** [[cmu-ri]] (CMU SCS / Robotics Institute) — PhD student
  in [[shubham-tulsiani]]'s group during the Track2Act work.

## Main contributions (within this wiki)

- **Three distinct research nodes now**, all through the human-video →
  object-motion lens: **[[track2act]]** (2D tracks → robot policy, CMU),
  **[[motionforesight]]** (dense 3D scene-flow forecast, no policy, JHU),
  **[[objectforesight]]** (rigid 6-DoF pose forecast, no policy, UW×CMU).
  MotionForesight and ObjectForesight are explicit siblings —
  flow-vs-pose forecasting of the same future-object-motion task.
- **[[track2act]]** (ECCV 2024) — first author on the founding paper
  of the "predict point tracks from web videos → drive robot
  manipulation" line. Email `hbharadh@cs.cmu.edu`.
- **[[motionforesight]]** (2026, JHU) — first author (with Yash Jangir).
  Repurposes a video-model 3D tracker ([[trackcraft3r]]) into a **future
  3D scene-flow forecaster**. The 3D + forecasting-only evolution of the
  Track2Act idea; no policy. Beats language-conditioned MolmoMotion
  (1.16M videos) trained on just 40K SSv2.
- **Hand-Object Mask paper** (Bharadhwaj × Gupta × Kumar × Tulsiani,
  ICRA 2024 — Track2Act ref [4]): the immediate predecessor of
  Track2Act; replaces hand+object masks with full point tracks.
- **RoboAgent** (Bharadhwaj × Vakil × Sharma × Gupta × Tulsiani ×
  Kumar, ICRA 2024 — Track2Act ref [5]): semantic augmentation +
  action chunking baseline.

## Sources in this wiki

- [[bharadhwaj-2024-track2act]] — first author (CMU).
- [[bharadhwaj-2026-motionforesight]] — first author (JHU).
- [[soraki-2026-objectforesight]] — 2nd author (UW × CMU; Soraki lead).

## Notes

- **Three sources now.** Track2Act (2D tracks → policy, CMU) →
  MotionForesight (dense 3D scene-flow *forecasting*, no policy, JHU) →
  ObjectForesight (rigid 6-DoF *pose* forecasting, no policy, UW×CMU)
  traces his arc through [[point-tracks-as-manipulation-interface]]:
  control-coupled 2D founding paper → two plan-only forecasting branches
  that differ only in output representation (flow vs pose). His
  MotionForesight paper argues flow ⊃ pose; ObjectForesight's diffusion
  formulation conversely handles the multimodality MotionForesight can't.
- Track2Act and Dex4D ([[kuang-2026-dex4d]]) form a clear Tulsiani-
  group lineage on point-tracks-as-manipulation-interface. Dex4D's
  acknowledgments thank Bharadhwaj for discussions.
- Author chain Park × Bharadhwaj × Tulsiani is also behind
  **DemoDiffusion** (2025, Dex4D ref [38]) — one-shot human imitation
  using pre-trained diffusion policy. Not yet ingested.
- Co-author on **Gen2Act** (Bharadhwaj et al., 2024 — Dex4D ref [2]):
  human video generation in novel scenarios for generalizable robot
  manipulation. Not yet ingested.
