---
type: entity
entity_type: person
name: Homanga Bharadhwaj
status: growing
tags: [manipulation, point-tracking, imitation-learning, web-video]
sources:
  - "[[bharadhwaj-2024-track2act]]"
  - "[[bharadhwaj-2026-motionforesight]]"
related:
  - "[[cmu-ri]]"
  - "[[johns-hopkins]]"
  - "[[meta-ai]]"
  - "[[shubham-tulsiani]]"
  - "[[track2act]]"
  - "[[motionforesight]]"
  - "[[point-tracks-as-manipulation-interface]]"
created: 2026-06-27
updated: 2026-07-16
---

# Homanga Bharadhwaj

## Affiliation

- **Now:** [[johns-hopkins]] — **Brains, Bots, and Behavior Lab**
  (faculty/lead), per [[bharadhwaj-2026-motionforesight]].
- **Previously:** [[cmu-ri]] (CMU SCS / Robotics Institute) — PhD student
  in [[shubham-tulsiani]]'s group during the Track2Act work.

## Main contributions (within this wiki)

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

## Notes

- **Two sources now.** Track2Act (2D tracks → policy, CMU) →
  MotionForesight (3D scene-flow *forecasting*, no policy, JHU) traces
  his own arc through [[point-tracks-as-manipulation-interface]]: from
  the control-coupled 2D founding paper to the plan-only 3D branch.
- Also authored **ObjectForesight** (Soraki × Bharadhwaj × Farhadi ×
  Mottaghi 2026, arXiv:2601.05237) — future rigid 6-DoF *pose*
  forecasting; MotionForesight argues scene flow generalizes past it.
  Not yet ingested.
- Track2Act and Dex4D ([[kuang-2026-dex4d]]) form a clear Tulsiani-
  group lineage on point-tracks-as-manipulation-interface. Dex4D's
  acknowledgments thank Bharadhwaj for discussions.
- Author chain Park × Bharadhwaj × Tulsiani is also behind
  **DemoDiffusion** (2025, Dex4D ref [38]) — one-shot human imitation
  using pre-trained diffusion policy. Not yet ingested.
- Co-author on **Gen2Act** (Bharadhwaj et al., 2024 — Dex4D ref [2]):
  human video generation in novel scenarios for generalizable robot
  manipulation. Not yet ingested.
