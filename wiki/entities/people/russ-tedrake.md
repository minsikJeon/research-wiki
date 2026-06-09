---
type: entity
entity_type: person
name: Russ Tedrake
status: growing
tags: [robotics, manipulation, control, optimization, diffusion-policy]
sources:
  - "[[chi-2024-diffusion-policy]]"
  - "[[chen-2024-diffusion-forcing]]"
related:
  - "[[mit-csail]]"
  - "[[diffusion-policy]]"
  - "[[diffusion-forcing]]"
created: 2026-06-08
updated: 2026-06-08
---

# Russ Tedrake

## Affiliation

Professor at [[mit-csail]]; VP of Robotics Research at Toyota Research
Institute (TRI). Long-running line on optimization-based control,
underactuated robotics, and (more recently) imitation learning for
manipulation.

## Main contributions (within this wiki)

- **[[diffusion-policy]]** (2023/2024, [[chi-2024-diffusion-policy]]) —
  co-author with Cheng Chi (lead), Shuran Song (senior). The
  foundational visuomotor diffusion policy paper.
- **[[diffusion-forcing]]** (2024 NeurIPS,
  [[chen-2024-diffusion-forcing]]) — co-author with Boyuan Chen (lead),
  Vincent Sitzmann. The per-token-noise sequence-modeling paradigm
  underlying πR² and streaming control.

## Sources in this wiki

- [[chi-2024-diffusion-policy]] (co-author)
- [[chen-2024-diffusion-forcing]] (co-author)

## Notes

Two papers in this wiki, both load-bearing for the
robotics-policy / diffusion-forcing thread. The pairing —
diffusion-policy (action chunking + DDPM) + diffusion-forcing
(per-position noise) — is the precise lineage that πR²
([[anon-2026-pi-r-squared]]) eventually weaves together for real-time
VLA control.