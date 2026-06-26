---
type: entity
entity_type: person
name: Carl Doersch
status: growing
tags: [point-tracking, video-understanding]
sources:
  - "[[doersch-2023-tapir]]"
  - "[[zholus-2025-tapnext]]"
  - "[[jung-2026-tapnext-plus-plus]]"
related:
  - "[[google-deepmind]]"
  - "[[oxford-vgg]]"
  - "[[tap-vid-dataset]]"
  - "[[point-tracking]]"
  - "[[tapir]]"
  - "[[tapnext]]"
  - "[[tapnext-plus-plus]]"
  - "[[artem-zholus]]"
created: 2026-05-24
updated: 2026-06-26
---

# Carl Doersch

## Affiliation

[[google-deepmind]] (London / Mountain View).

## Main contributions (within this wiki)

The "TAP line" at DeepMind — Doersch is the **connective thread** across
every generation of TAP method from the [[google-deepmind]] camp:

- **TAP-Vid** (2022, NeurIPS) — introduced the [[tap-vid-dataset]]
  benchmark and the TAP task formulation. (Not yet ingested.)
- **TAPIR** (2023, ICCV) — [[doersch-2023-tapir]]; the canonical
  TAP method of 2023, fusing TAP-Net global init with [[pips]] temporal
  refinement; introduced Panning MOVi-E + position uncertainty.
- **BootsTAP / BootsTAPIR** (2024) — self-supervised real-data
  fine-tuning recipe. (Not yet ingested; referenced from CoTracker3,
  TAPNext, TAPNext++.)
- **TAPNext** (2025, [[zholus-2025-tapnext]]) — masked-decoding framing,
  drops tracking-specific inductive biases.
- **TAPNext++** (2026, [[jung-2026-tapnext-plus-plus]]) — long-sequence
  + re-detection successor.

## Sources in this wiki

- [[doersch-2023-tapir]] (first author)
- [[zholus-2025-tapnext]] (co-author)
- [[jung-2026-tapnext-plus-plus]] (co-author)

## Notes

When future tracking papers arrive from DeepMind, check for Doersch in
the author list — it usually signals continuity with this lineage.
Pairs with [[artem-zholus]] on the TAPNext / TAPNext++ specifically.
The unifying technical theme across his contributions: **classification
over per-frame appearance matching**, with refinement / recurrence on top.
