---
type: entity
entity_type: org
name: Google DeepMind
status: growing
tags: []
sources:
  - "[[doersch-2023-tapir]]"
  - "[[zholus-2025-tapnext]]"
  - "[[jung-2026-tapnext-plus-plus]]"
  - "[[jin-2026-zipmap]]"
related:
  - "[[carl-doersch]]"
  - "[[artem-zholus]]"
  - "[[haian-jin]]"
  - "[[aleksander-holynski]]"
  - "[[tapir]]"
  - "[[tapnext]]"
  - "[[tapnext-plus-plus]]"
  - "[[zipmap]]"
created: 2026-05-24
updated: 2026-08-14
---

# Google DeepMind

## Type

Industry research lab. Formed 2023 from the merger of DeepMind (London)
and Google Brain.

## Focus areas relevant to this wiki

Foundation models, video understanding, point tracking (the entire TAP-Vid
→ TAPIR → BootsTAP → TAPNext → TAPNext++ line), robotics (RT-X / Open
X-Embodiment), perception, world models, scaling laws. Likely to be one
of the most-cited orgs in this wiki given the field.

## Members in this wiki

- [[carl-doersch]]
- [[artem-zholus]] (intern affiliation; primary affiliation Mila /
  Polytechnique Montréal)
- [[mehdi-sajjadi]] (D4RT lead; SRT line)
- [[skanda-koppula]] (TAPVid-3D introducer; D4RT + TAPNext co-author)
- [[ignacio-rocco]] (D4RT + TAPNext co-author; prev. Meta AI on
  CoTracker)
- [[haian-jin]] (ZipMap lead; Cornell PhD + GDM; LVSM / RayZer line)
- [[aleksander-holynski]] (senior on ZipMap + CUT3R; streaming/stateful 3D)

## Sources from this org

- [[doersch-2023-tapir]] — Doersch + Yang + DeepMind/Oxford joint;
  the canonical TAP method of 2023.
- [[zholus-2025-tapnext]]
- [[jung-2026-tapnext-plus-plus]]
- [[zhang-2025-d4rt]] (lead [[mehdi-sajjadi]]; co-authors
  [[skanda-koppula]], [[ignacio-rocco]])
- [[wang-2025-cut3r]] (UC Berkeley + DeepMind joint via Holynski)
- [[jin-2026-zipmap]] (GDM × Cornell × MIT; [[haian-jin]] lead,
  [[aleksander-holynski]] senior) — linear-time TTT 3D reconstruction.
  Marks DeepMind's feed-forward-3D line (beyond the TAP tracking line).

## Notes

Sub-line tracking: the TAPNext family is the latest in DeepMind's TAP
line; expect future iterations to maintain the
classification-coordinate-head + recurrent-backbone theme. Watch also
for related lines (RecurrentGemma, V-JEPA, Genie) that share architectural
DNA.
