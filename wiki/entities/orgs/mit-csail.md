---
type: entity
entity_type: org
name: MIT CSAIL
status: growing
tags: [research-lab, robotics, computer-vision, generative-models]
sources:
  - "[[chi-2024-diffusion-policy]]"
  - "[[chen-2024-diffusion-forcing]]"
  - "[[song-2025-history-guided-video-diffusion]]"
  - "[[zhang-2025-lact]]"
  - "[[song-2026-gvs]]"
related:
  - "[[russ-tedrake]]"
  - "[[boyuan-chen]]"
  - "[[chonghyuk-song]]"
  - "[[diffusion-policy]]"
  - "[[diffusion-forcing]]"
  - "[[dfot]]"
  - "[[gvs]]"
  - "[[lact]]"
  - "[[history-guidance]]"
created: 2026-06-08
updated: 2026-06-12
---

# MIT CSAIL (Computer Science and Artificial Intelligence Laboratory)

## Type / focus

Academic research lab at MIT. Within scope of this wiki, the relevant
research lines are:
- **Robotics policy + sequence modeling** ([[russ-tedrake]] group +
  collaborators).
- **Generative models for video / planning** (Vincent Sitzmann group;
  Yilun Du).

## Members in this wiki

- [[russ-tedrake]] (Professor, robotics / control / policy learning).
- [[boyuan-chen]] (PhD; diffusion forcing line — DF + DFoT + GVS co-author).
- [[chonghyuk-song]] (PhD; GVS lead).

## Sources from MIT CSAIL

- [[chi-2024-diffusion-policy]] — Cheng Chi (visiting; led from
  Columbia), Yilun Du, Russ Tedrake co-authors.
- [[chen-2024-diffusion-forcing]] — Boyuan Chen (lead), Yilun Du, Max
  Simchowitz, Russ Tedrake, Vincent Sitzmann (all MIT CSAIL).
- [[song-2025-history-guided-video-diffusion]] — Kiwhan Song +
  [[boyuan-chen]] (equal contribution leads), Yilun Du, Russ Tedrake,
  Vincent Sitzmann.
- [[zhang-2025-lact]] — Tianyuan Zhang + Songlin Yang + William T. Freeman
  (MIT); Sai Bi, Yicong Hong, Kai Zhang, Fujun Luan, Kalyan Sunkavalli,
  Hao Tan (Adobe Research). MIT–Adobe collaboration.
- [[song-2026-gvs]] — [[chonghyuk-song]] (lead), Michal Stary,
  [[boyuan-chen]], George Kopanas (Runway ML), Vincent Sitzmann.

## Notes

Now **five** sources from MIT CSAIL, anchored by two threads:
1. **Diffusion-forcing line** — Diffusion Policy → Diffusion Forcing → DFoT
   → GVS, with Boyuan Chen + Vincent Sitzmann as the recurring anchors.
   Adjacent to πR² which uses the same per-position-AdaLN recipe.
2. **Sequence-modeling efficiency line** — LaCT, the large-chunk TTT paper,
   contributes to the [[test-time-training]] thread; Tianyuan Zhang + William
   Freeman + Songlin Yang are the MIT contributors.

The two threads converge: LaCT's autoregressive video diffusion (§4.3)
uses the same noisy/clean-chunk interleaving recipe as DFoT. CSAIL is now
the de facto center of both diffusion-forcing-for-video and TTT-for-long-context
streaming.