---
type: concept
title: Joint Point Tracking
status: growing
tags:
  - point-tracking
  - video-understanding
  - attention
sources:
  - "[[karaev-2024-cotracker]]"
  - "[[karaev-2024-cotracker3]]"
related:
  - "[[tapnext]]"
  - "[[tapnext-plus-plus]]"
  - "[[cotracker]]"
  - "[[cotracker3]]"
  - "[[track-on2]]"
  - "[[tapip3d]]"
  - "[[spatialtracker-v2]]"
  - "[[tap-vid-dataset]]"
  - "[[joint-point-tracking]]"
  - "[[3d-point-tracking]]"
  - "[[online-vs-offline-tracking]]"
  - "[[synthetic-to-real-gap]]"
created: 2026-05-24
updated: 2026-05-24
---

# Joint Point Tracking

## Definition

A [[point-tracking]] design choice where **multiple tracked points
exchange information** during inference, rather than each point being
estimated independently. The hypothesis: points often have statistical
dependencies — they belong to the same object, move together under
rigid transformations, share appearance context — and using these
dependencies improves tracking, especially during occlusion.

## Why it matters

Most pre-2024 trackers (TAPIR, PIPs, PIPs++) treat tracks as independent.
CoTracker showed this leaves a lot on the table:

- Background points are dragged by foreground object motion if tracked
  independently in the wrong feature window (Fig 2 of [[karaev-2024-cotracker]]).
- Occluded points can be inferred from their visible neighbors that share
  rigid motion.

## How it's implemented

Two main patterns observed across this wiki:

1. **Cross-track attention** ([[cotracker]], [[cotracker3]]): build a
   `[T, N]` token grid, apply self-attention across the N (tracks)
   dimension to exchange information between tracks.
2. **Proxy / virtual tokens** ([[cotracker]]): a small set of register
   tokens that all tracks cross-attend to, reducing O(N²) to O(N·P)
   and enabling 70K tracks on one GPU.

[[tapnext]] / [[tapnext-plus-plus]] technically allow point-token
interaction via the ViT spatial-attention block, but don't explicitly
brand it as "joint tracking" — see Fig 4 of TAPNext for evidence that
this still emerges (motion-cluster attention heads).

## Evidence across sources

- [[karaev-2024-cotracker]]: cross-track attention introduced. Ablation:
  improves occluded points more than visible (+5.1 vs +1.6 AJ on
  Dynamic Replica).
- [[karaev-2024-cotracker3]]: reconfirms the cross-track-attention
  contribution; it survives architecture simplification.
- [[zholus-2025-tapnext]]: doesn't use explicit joint tracking but
  motion-cluster patterns *emerge* in attention heads — a form of
  learned joint tracking.

## Open questions

- Does joint tracking still help when point density is sparse (a handful
  of unrelated query points across the frame)?
- How does joint-track attention compose with 3D-aware attention
  ([[tapip3d]]'s N2N)?
- Can "support points" (auto-added context tracks) match cross-track
  attention gains, or is the implicit attention necessary?
