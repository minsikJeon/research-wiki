---
type: entity
entity_type: dataset
name: Something-Something V2 (SSv2)
status: stub
tags: [dataset, video-understanding, human-object-interaction, manipulation, action-recognition]
sources:
  - "[[bharadhwaj-2026-motionforesight]]"
related:
  - "[[motionforesight]]"
  - "[[point-tracks-as-manipulation-interface]]"
created: 2026-07-16
updated: 2026-07-16
---

# Something-Something V2 (SSv2)

## What it is

Large crowd-sourced video dataset of **basic human–object interactions**
(Goyal et al., ICCV 2017 for V1; V2 is the expanded release). ~220K short
clips over 174 action templates phrased compositionally — "putting
[something] behind [something]", "pushing [something] so it falls off",
etc. Objects are generic placeholders, forcing models to learn the
*interaction dynamics* rather than object identity. Standard benchmark for
temporal / motion-centric video understanding.

> Stub — recorded because it is [[motionforesight]]'s entire training
> corpus. Expand if a paper leans on its schema or biases.

## Where it's used in this wiki

- [[bharadhwaj-2026-motionforesight]] — 40K SSv2 clips as the training set
  for future 3D scene-flow forecasting. Object *name* annotations used
  only offline (to seed SAM masks); the forecasting model receives **no**
  language / action labels.

## Notes / biases

- Motion-centric by construction — well-suited to learning "how do objects
  move when manipulated," which is exactly the [[motionforesight]] target.
- Monocular, casual capture, no metric 3D / camera ground truth → any 3D
  supervision (as in MotionForesight) is **pseudo-GT** from a depth +
  tracking pipeline, inheriting those errors.
- Tabletop / hand-scale interactions; not egocentric, not whole-body.
