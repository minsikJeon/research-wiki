---
type: entity
entity_type: dataset
name: Argoverse-HD (High-frame-rate Detection)
status: stub
tags: [streaming-perception, object-detection, autonomous-driving, benchmark, dataset]
sources:
  - "[[li-2020-streaming-perception]]"
related:
  - "[[streaming-perception]]"
  - "[[streaming-accuracy]]"
created: 2026-05-28
updated: 2026-05-28
---

# Argoverse-HD (High-frame-rate Detection)

## Composition

Re-annotation of the [Argoverse 1.1](https://argoverse.org/) urban-
driving video dataset with **dense 30 FPS 2D bounding boxes** for 8
COCO-style classes:

- person, bicycle, car, motorcycle, bus, truck, traffic light, stop sign.

Introduced by [[li-2020-streaming-perception]] specifically to enable
streaming-perception evaluation.

## Why it matters

The **first benchmark designed for [[streaming-perception]]**.
Standard video detection datasets (YouTube-VIS, MOT) are annotated
sparsely (e.g. 6 FPS on 30 FPS video) which makes streaming evaluation
impossible. Argoverse-HD's dense 30 FPS annotation is the design
requirement.

## Metrics

- **Streaming AP** (= sAP in later literature): zero-order-hold the
  detector's output stream vs ground-truth at every frame, compute
  standard COCO-style AP.
- Standard COCO breakdowns: AP, AP_50, AP_75, AP_S/M/L.

## Composition details

- 24 validation videos, 15–30 seconds each, 15K total frames.
- Center RGB camera only (Argoverse has multi-sensor; the streaming
  benchmark uses only this).
- COCO-format annotation including `iscrowd` for ambiguous instances.

## Known biases / caveats

- **Driving scenes only.** Generalization to other embodied perception
  settings (indoor manipulation, AR/VR) is an open question.
- **8 classes** is a narrow slice of COCO.
- **Hardware-dependent absolute numbers** (the 2020 paper used a GTX
  1080 Ti); rankings are more stable than absolute scores.

## Papers using it in this wiki

- [[li-2020-streaming-perception]] (introducing paper). Mask R-CNN R50
  @ s0.5 + scheduling + association + forecasting reaches sAP 17.8.
- _(Many later streaming-perception papers — StreamYOLO, DAMO-StreamNet,
  etc. — use this benchmark; promote when ingested.)_
