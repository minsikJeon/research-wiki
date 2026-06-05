---
type: entity
entity_type: person
name: Mehdi S. M. Sajjadi
status: growing
tags: [3d-reconstruction, 4d-reconstruction, scene-representation-transformer, foundation-model]
sources:
  - "[[zholus-2025-tapnext]]"
  - "[[zhang-2025-d4rt]]"
related:
  - "[[google-deepmind]]"
  - "[[d4rt]]"
  - "[[tapnext]]"
created: 2026-05-26
updated: 2026-05-26
---

# Mehdi S. M. Sajjadi

## Affiliation

[[google-deepmind]].

## Main contributions (within this wiki)

The **SRT → D4RT line** at DeepMind — Sajjadi-led research on
representation-transformer architectures for 3D/4D understanding:

- **SRT (Scene Representation Transformer)** (Sajjadi et al. 2022) —
  not yet ingested. Original encoder-decoder transformer pattern with
  one global representation + query-based decoder. Foundation for the
  query-based 4D paradigm.
- **[[d4rt]]** (2025, [[zhang-2025-d4rt]]) — **lead author**. Dynamic
  4D extension of SRT. The canonical 2D-pixel-query 4D decoder that
  [[point4d]], [[4rc]], and others build on.
- **[[tapnext]]** (2025, [[zholus-2025-tapnext]]) — co-author.

## Sources in this wiki

- [[zholus-2025-tapnext]] (co-author)
- [[zhang-2025-d4rt]] (project lead, corresp.)

## Notes

The **architectural common thread** across his contributions is the
**Scene Representation Transformer** pattern: a single transformer
produces a global latent scene representation, then a lightweight
decoder queries it. The TAPNext "ViT + SSM" backbone shares
philosophical DNA with this — separate stages for global context vs.
per-token decoding.

Sajjadi is the **most-recurring DeepMind author** in this wiki's 4D
work after [[carl-doersch]]. Future Sajjadi-led papers are high
priority.
