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
related:
  - "[[russ-tedrake]]"
  - "[[boyuan-chen]]"
  - "[[diffusion-policy]]"
  - "[[diffusion-forcing]]"
  - "[[dfot]]"
  - "[[history-guidance]]"
created: 2026-06-08
updated: 2026-06-11
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
- [[boyuan-chen]] (PhD; diffusion forcing line — DF + DFoT).

## Sources from MIT CSAIL

- [[chi-2024-diffusion-policy]] — Cheng Chi (visiting; led from
  Columbia), Yilun Du, Russ Tedrake co-authors.
- [[chen-2024-diffusion-forcing]] — Boyuan Chen (lead), Yilun Du, Max
  Simchowitz, Russ Tedrake, Vincent Sitzmann (all MIT CSAIL).
- [[song-2025-history-guided-video-diffusion]] — Kiwhan Song +
  [[boyuan-chen]] (equal contribution leads), Yilun Du, Russ Tedrake,
  Vincent Sitzmann.

## Notes

The combination — diffusion-policy as the action-chunking head +
diffusion-forcing as the per-token-noise paradigm + DFoT as the
non-causal video instance — is the substrate for the modern
real-time-chunking line (RTC / Training-Time RTC / πR²) and the
modern flexible-conditioning video generation line. The Chen + Sitzmann
+ Tedrake cluster is now responsible for *three* wiki sources spanning
this thread, with [[boyuan-chen]] specifically appearing as a recurring
lead. Worth tracking the next paper in the line.