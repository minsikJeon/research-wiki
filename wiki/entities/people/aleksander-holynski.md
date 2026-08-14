---
type: entity
entity_type: person
name: Aleksander Hołyński
status: stub
tags: [3d-reconstruction, streaming, novel-view-synthesis, generative-models]
sources:
  - "[[wang-2025-cut3r]]"
  - "[[jin-2026-zipmap]]"
related:
  - "[[cut3r]]"
  - "[[zipmap]]"
  - "[[google-deepmind]]"
  - "[[qianqian-wang]]"
created: 2026-08-14
updated: 2026-08-14
---

# Aleksander Hołyński

## Affiliation

Google DeepMind (+ UC Berkeley). Works across feed-forward 3D
reconstruction, streaming/stateful scene representations, and generative
view synthesis.

## Main contributions in this wiki

- Senior author on **[[cut3r|CUT3R]]** ([[wang-2025-cut3r]], 2025) — stateful
  recurrent online 3D reconstruction (**fixed implicit state**).
- Senior author on **[[zipmap|ZipMap]]** ([[jin-2026-zipmap]], 2026) —
  linear-time stateful 3D reconstruction (**TTT fast-weight state**).

**Through-line:** both his wiki papers are *stateful streaming 3D
reconstruction* — CUT3R with an attention-updated fixed token bank, ZipMap
with a gradient-updated fast-weight MLP. Same problem, two memory
mechanisms; he is on both sides of the fixed-state-vs-TTT-state design axis.

## Papers in this wiki

- [[wang-2025-cut3r]]
- [[jin-2026-zipmap]]

(**2 sources**)
