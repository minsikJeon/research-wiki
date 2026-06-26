---
type: method
title: TAPNext
status: stub
tags: [point-tracking, state-space-model, vit, transformer, masked-decoding, online-tracking]
sources:
  - "[[zholus-2025-tapnext]]"
related:
  - "[[point-tracking]]"
  - "[[tap-vid-dataset]]"
  - "[[tapir]]"
  - "[[pips]]"
created: 2026-05-24
updated: 2026-06-26
---

# TAPNext

A [[point-tracking]] method that casts TAP as **masked token decoding**:
unknown per-point trajectory positions are filled in by a generic backbone
with no tracking-specific inductive biases.

## One-line summary

Concatenate `[T, h×w, C]` ViT patch tokens with `[T, Q, C]` per-point
trajectory tokens (known query = positional embedding of `(x,y)`;
unknown = learned mask token); pass through L × (RecurrentGemma SSM-over-time
+ ViT-over-space) layers; read each trajectory token through a
classification coordinate head + binary visibility head. Causal, per-frame.

## Inputs / outputs

- **In:** RGB video `[T, H, W, 3]` + Q query points `(t, x, y)`.
- **Out:** `[T, Q, 2]` coordinates + `[T, Q, 1]` visibility, one frame
  at a time.

## How it works

1. **Tokenize.** Image → 8×8 patches → linear projection + spatial PE
   → `[T, h×w, C]`. Each query becomes T trajectory tokens
   (query frame = PE(x,y); rest = mask token). No temporal PE — it harms
   length generalization.
2. **Backbone.** L blocks of (a) SSM scan along time treating space as batch,
   (b) ViT self-attention across `h×w + Q` tokens per frame treating time
   as batch. The SSM is RecurrentGemma; the ViT is big_vision's standard.
3. **Heads.** Coordinate head = 256-way classifier per axis with expectation
   read-out for continuous output; loss = softmax CE + Huber on continuous
   prediction. Visibility head = sigmoid BCE. Auxiliary losses at every layer.
4. **Train.** Sim-only Kubric extension (500K videos × 48 frames). Sizes:
   TAPNext-S 56M / TAPNext-B 194M.
5. **BootsTAPNext.** Fine-tune ~1.5K steps on ~15M real clips via BootsTAP's
   student-teacher EMA equivariance recipe.

## Where it's been applied

- TAP-Vid benchmark (DAVIS + Kinetics) — see [[zholus-2025-tapnext]] for
  full numbers.
- _(no further applications ingested yet.)_

## Known limitations

- Fails on videos > ~150 frames (SSM state not trained for that length).
  Mitigations: clip the SSM forget gate to [0, 0.1]; broadcast query
  features across the temporal axis.
- Requires the full BootsTAP semi-supervised pipeline + ~15M real clips
  to hit best numbers.
- No explicit appearance-memory slot; long-occlusion robustness depends on
  whatever the SSM state happens to retain.

## Related methods

- **Direct successor:** [[tapnext-plus-plus]] — same architecture, fixes
  the >150-frame failure mode via 1024-frame training + roll augmentation
  + occluded-point supervision.
- **Predecessors in the TAP line (DeepMind):** TAPNet, [[tapir]]
  (depthwise-conv refinement + global init + uncertainty),
  BootsTAP / BootsTAPIR (same code lineage, different inductive-bias
  choices). TAPNext's thesis ("none of these tracking-specific
  components are necessary") is explicitly aimed at [[tapir]]'s and
  [[pips]]'s engineered cost-volume + iterative-refinement stack — see
  [[q-emergent-tracking-heuristics]].
- **Window/offline competitors:** [[cotracker]], [[cotracker3]],
  LocoTrack, TAPTR(v2/v3), OmniMotion, Dino-Tracker.
- **Causal per-frame competitor:** [[track-on2]] (memory-based instead of
  SSM-based). Convergent design: both use classification coordinate heads.
- **Backbone:** TRecViT — interleaved SSM (time) + ViT (space). TAPNext is
  the first application of this backbone to TAP.
- **Coordinate-as-classification:** inspired by Farebrother et al.'s
  "stop regressing" line of work.
