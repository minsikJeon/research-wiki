---
type: entity
entity_type: person
name: Boyuan Chen
status: growing
tags: [generative-models, diffusion, video, robotics, sequence-modeling]
sources:
  - "[[chen-2024-diffusion-forcing]]"
  - "[[song-2025-history-guided-video-diffusion]]"
  - "[[song-2026-gvs]]"
related:
  - "[[mit-csail]]"
  - "[[diffusion-forcing]]"
  - "[[dfot]]"
  - "[[history-guidance]]"
  - "[[gvs]]"
  - "[[chonghyuk-song]]"
created: 2026-06-11
updated: 2026-06-12
---

# Boyuan Chen

## Affiliation

MIT CSAIL (PhD). Personal site: https://boyuanchen.com (project websites
under https://boyuan.space/).

## Focus areas relevant to this wiki

- **Diffusion forcing** — Chen is the lead author of the foundational
  [[chen-2024-diffusion-forcing]] paper (NeurIPS 2024), which introduced
  per-token independent noise levels as a sequence-modeling paradigm.
- **Video diffusion + history conditioning** — co-lead (equal
  contribution with Kiwhan Song) on
  [[song-2025-history-guided-video-diffusion]] (ICML 2025), extending DF
  to non-causal video DiTs and introducing the History Guidance family.

## Sources in this wiki

- [[chen-2024-diffusion-forcing]] — *Diffusion Forcing: Next-Token
  Prediction Meets Full-Sequence Diffusion*. NeurIPS 2024.
- [[song-2025-history-guided-video-diffusion]] — *History-Guided Video
  Diffusion*. ICML 2025.
- [[song-2026-gvs]] — *Generative View Stitching*. ICLR 2026
  (co-author with [[chonghyuk-song]]).

## Notes

Three-source author. The 2024 → 2025 → 2026 trajectory shows Chen's
diffusion-forcing line is load-bearing for the modern video-diffusion +
real-time-control intersection: DF was foundational; DFoT operationalized
it for video; GVS extended it to long-horizon offline stitching; πR²
applied the same mechanism to VLA action chunks. All four papers run
through MIT CSAIL — see [[mit-csail]].

Frequent co-authors in this wiki: [[russ-tedrake]], Max Simchowitz,
Yilun Du, Vincent Sitzmann, [[chonghyuk-song]].
