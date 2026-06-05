---
type: method
title: V-DPM (Video Dynamic Point Maps)
status: stub
tags: [4d-reconstruction, video-reconstruction, transformer, dynamic-point-maps, vggt-fine-tune, scene-flow]
sources:
  - "[[sucar-2026-v-dpm]]"
related:
  - "[[vggt]]"
  - "[[dynamic-point-maps]]"
  - "[[4d-reconstruction]]"
  - "[[pointmap-representation]]"
created: 2026-05-24
updated: 2026-05-24
---

# V-DPM (Video Dynamic Point Maps)

Multi-view (video) extension of pairwise Dynamic Point Maps, implemented
by **fine-tuning [[vggt]]** with new dynamic heads. Recovers shape + 3D
motion + camera + intrinsics in a single feed-forward representation.

## One-line summary

VGGT backbone + a **time-variant DPM head** (per-frame point maps) + a
**time-invariant DPM head** (target-time-conditioned, AdaLN-modulated)
that fuses all input views into a reference-time, reference-viewpoint
reconstruction. Fine-tuned on synthetic dynamic data (Kubric).

## Inputs / outputs

- **In:** N video frames; target time `t_τ` (chosen reference time).
- **Out:** per-pixel DPMs `P_i(t_j, π_k)` — encoding scene shape +
  motion + camera, viewpoint- and time-invariant after fusion.
- **Derived:** depth, scene flow (via `ΔP`), camera params.

## How it works

1. **Backbone:** VGGT-style alternating attention. Loaded from VGGT
   weights, fine-tuned.
2. **Two-phase decode:**
   - Phase 1 (time-variant): per-frame DPMs at source time `t_i` for
     each frame. Preserves VGGT-like static behavior.
   - Phase 2 (time-invariant): additional decoder layers attend to
     phase-1 outputs + a target-time token (AdaLN conditioning) →
     reconstruction at the target time + reference viewpoint.
3. **Multi-image DPM reduction:** instead of O(N²) pairwise DPMs,
   predict all maps relative to view 1 → O(N) maps per target time.
4. **Training:** fine-tune VGGT on Kubric synthetic dynamic data —
   relatively little new training needed.

## Where it's been applied

- Standard 4D benchmarks (vs DPM, MonST3R, St4RTrack) — halves error rate.

## Known limitations

- Trained only on synthetic dynamic data.
- Compared mostly against pre-VGGT 4D methods; missing head-to-head vs
  concurrent feed-forward 4D ([[any4d]], [[4rc]], [[trace-anything]]).
- [[4rc]] critique: "high computational cost and inflexible decoding."

## Related methods

- **Backbone:** [[vggt]] (fine-tuned).
- **Direct precursor:** DPM (Sucar et al. 2025) — pairwise version.
- **Concurrent feed-forward 4D competitors:** [[any4d]], [[4rc]],
  [[trace-anything]], Pi3 (extends VGGT for dynamic depth only).
- **Strategy:** "smallest possible architectural change to make VGGT
  dynamic" — supports the VGGT-as-backbone hypothesis.
