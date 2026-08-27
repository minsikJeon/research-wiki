---
type: entity
entity_type: dataset
name: EPIC-Kitchens
status: stub
tags: [dataset, egocentric, human-object-interaction, manipulation, video-understanding]
sources:
  - "[[soraki-2026-objectforesight]]"
related:
  - "[[objectforesight]]"
  - "[[something-something-v2]]"
  - "[[point-tracks-as-manipulation-interface]]"
created: 2026-08-27
updated: 2026-08-27
---

# EPIC-Kitchens

## What it is

Large **egocentric** (head-mounted) video dataset of unscripted kitchen
activity (Damen et al., ECCV 2018; expanded as EPIC-Kitchens-100). Dense
action-segment annotations (verb + noun) over many hours of first-person
manipulation. Standard benchmark for egocentric action recognition,
anticipation, and hand-object interaction.

> Stub — recorded because it is [[objectforesight]]'s entire training
> corpus. Expand if another source leans on it.

## Where it's used in this wiki

- [[soraki-2026-objectforesight]] — source of the auto-curated 2M+ 3D
  object-trajectory dataset: 76K action segments → 72K clips → SAM2 tracks →
  59K pose-aligned tracks → 3.06M raw → **2.07M** filtered (C+H) 6-DoF
  trajectory windows. Object *poses* are pseudo-GT (FoundationPose +
  SpaTrackerV2 + TRELLIS); EPIC provides only RGB + action segments.

## Notes / biases

- **Egocentric + monocular, no metric 3D / camera GT** → any 3D supervision
  is pseudo-GT from a depth + pose stack, inheriting its errors (same
  caveat as [[something-something-v2]] for [[motionforesight]]).
- **Kitchen / tabletop manipulation** — hand-scale rigid + articulated
  objects; strong ego-motion (the reason [[objectforesight]] canonicalizes
  to anchor-frame coords).
- Distinct from [[something-something-v2]]: EPIC is real unscripted
  egocentric activity; SSv2 is crowd-sourced compositional templates.
