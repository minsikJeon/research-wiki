---
type: entity
entity_type: person
name: Russ Tedrake
status: growing
tags: [robotics, manipulation, control, optimization, diffusion-policy]
sources:
  - "[[chi-2024-diffusion-policy]]"
  - "[[chen-2024-diffusion-forcing]]"
  - "[[song-2025-history-guided-video-diffusion]]"
related:
  - "[[mit-csail]]"
  - "[[boyuan-chen]]"
  - "[[diffusion-policy]]"
  - "[[diffusion-forcing]]"
  - "[[dfot]]"
created: 2026-06-08
updated: 2026-06-11
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

- **[[dfot]]** (2025 ICML, [[song-2025-history-guided-video-diffusion]])
  — co-author with Kiwhan Song + [[boyuan-chen]] (equal-contribution
  leads), Vincent Sitzmann. Extends [[diffusion-forcing]] to non-causal
  video DiTs; introduces [[history-guidance]].

## Sources in this wiki

- [[chi-2024-diffusion-policy]] (co-author)
- [[chen-2024-diffusion-forcing]] (co-author)
- [[song-2025-history-guided-video-diffusion]] (co-author)

## Notes

Three papers in this wiki, all load-bearing for the diffusion-forcing /
robotics-policy / video-diffusion intersection. The trajectory —
diffusion-policy (action chunking + DDPM) → diffusion-forcing (per-position
noise) → DFoT (non-causal DiT video instance) — feeds directly into πR²
([[anon-2026-pi-r-squared]]), which uses the same per-position-AdaLN
recipe for real-time VLA control. Tedrake's role across the line is
senior author / advisor rather than algorithm lead.