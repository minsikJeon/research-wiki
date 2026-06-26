---
type: overview
title: Research Wiki — Overview
status: growing
tags: [meta]
created: 2026-05-24
updated: 2026-06-26
---

# Overview

The evolving north-star synthesis of this wiki. Unlike [[index]] (a catalog)
and [[log]] (a chronology), this page is **the current best-effort summary
of what the wiki knows and where it's going**. Rewritten as the wiki grows.

The wiki belongs to a CMU **Master of Science in Robotics** student. Primary
fields: **computer vision** and **robotics**, with everything they touch —
foundation models for perception/control, video understanding, point tracking,
3D/4D reconstruction, SLAM, manipulation, imitation learning, world models,
embodied AI.

---

## Current thesis (June 2026, after 32 sources)

The wiki has dense coverage of two intersecting perception sub-areas,
plus two now-fleshed-out adjacent threads (streaming/control, and
long-context 3D):

**(A) Tracking Any Point (TAP).** 7 sources covering the DeepMind TAPNext
line, the Meta/Oxford CoTracker line, the Koç Track-On line, and the 3D
extensions (TAPIP3D from CMU, SpatialTrackerV2).

**(B) Feed-forward 3D / 4D reconstruction.** 10 sources covering the VGGT
trunk (Oxford VGG + Meta AI) and its descendants (MapAnything, V-DPM,
4RC, DA3, CUT3R), plus contemporary feed-forward 4D (Any4D, Trace
Anything, D4RT, Point4D, **STRIDE** — first driving-scene multi-modal
entry).

**(C) Streaming perception & real-time control.** Now **7 sources**
covering the full lineage:
- Foundation: [[li-2020-streaming-perception]] (ECCV 2020).
- Action-chunking origin: [[zhao-2023-act]] (ACT/ALOHA, RSS 2023).
- Action-head substrate: [[chi-2024-diffusion-policy]] (RSS 2023 / IJRR
  2024) — DDPM over action chunks.
- Sequence-modeling mechanism: [[chen-2024-diffusion-forcing]] (NeurIPS
  2024) — per-token noise schedules.
- Real-time chunking line:
  [[black-2025-rtc]] (inference-time) →
  [[black-2025-training-time-rtc]] (training-time) →
  [[anon-2026-pi-r-squared]] (joint structural fix: diffusion-forcing
  staircase + slow/fast channel split).

**(D) Long-context 3D reconstruction (linear-time / streaming).** Now
**4 sources** with this batch: [[elflein-2026-vgg-t3]] (NVIDIA;
TTT-compressed scene as MLP, `O(n)` global/offline/unordered),
[[zhang-2026-loger]] (DeepMind+Berkeley; chunked feed-forward with
**hybrid memory** = SWA + TTT), [[zhang-2025-lact]] (MIT+Adobe;
**Large-Chunk TTT** = the missing efficiency layer underneath the
TTT-for-3D papers, plus 14B-parameter AR video diffusion), and
[[zhuo-2026-stream-vggt]] (Tsinghua; **causal-attention + KV-cached
streaming distillation** of VGGT — beats CUT3R on every measured
benchmark). Two distinct streaming strategies now articulated:
**(D1) test-time-trained fast weights** (VGG-T3, LoGeR, LaCT — fast
weights *are* the memory) and **(D2) cached KV / causal-attention**
(StreamVGGT, CUT3R — explicit memory + attention). Both are O(n) per
frame; trade-offs are unsettled.

(A) and (B) are **visibly converging** — see
[[q-tracking-vs-4d-reconstruction]]. (C) sits underneath both: it's the
evaluation framework + control-side analogue that the user's questions
about online inference and train-inference mismatch all live within.
**(D) is the perception-side analogue of (C)** — it asks the same
"finite-time computation on growing input" question for 3D
reconstruction. (D) is directly load-bearing for the user's own
research project (Topic 3 in `raw/notes/` — long-term real-time 3D
tracking architectures).

**As of June 2026, (C) is a step *ahead* of (D)** on two specific
patterns:
- The **inference → training → joint structural fix** trajectory has
  played out fully on the control side (RTC → Training-Time RTC → πR²)
  but the perception side is still at *inference-time structural*
  (Point4D) or *heuristic* (SpaTrackerV2).
- The **fast-slow split** (πR²'s slow VLM + fast proprio) is
  structurally equivalent to LoGeR's hybrid memory (slow TTT + fast
  SWA) but at higher operational maturity (deployed on a real robot at
  25 Hz).

### Seven load-bearing claims (synthesized)

1. **Online causal tracking has matched windowed/offline methods** for 2D
   TAP. [[tapnext]], [[tapnext-plus-plus]], [[track-on2]] all achieve
   SOTA at frame-by-frame latency (~5–30 ms). The historical
   quality-vs-latency trade-off has collapsed.

2. **Synthetic-only training is no longer obviously inferior** to real-
   data pseudo-labeling. [[track-on2]] (memory + classification + long
   synthetic) and [[tapnext-plus-plus]] (1024-frame Kubric) compete with
   the BootsTAP / [[cotracker3]] real-data line. See
   [[synthetic-to-real-gap]] and [[q-sim2real-for-point-tracking]].

3. **3D point tracking is now a real sub-field**, with two design
   philosophies: modular ([[tapip3d]]) vs. end-to-end
   ([[spatialtracker-v2]]). No definitive head-to-head yet.

4. **Feed-forward 3D reconstruction has a clear trunk + canopy**.
   DUSt3R/MASt3R → [[vggt]] is the trunk. [[mapanything]], [[v-dpm]],
   [[4rc]], [[depth-anything-3]], [[cut3r]] are the canopy.
   **No 3D inductive biases needed** ([[vggt]], [[depth-anything-3]]);
   **factored representations beat coupled pointmaps** ([[mapanything]],
   [[any4d]]); the field is becoming foundation-model-shaped.

5. **The 3D-reconstruction wave is feeding 4D reconstruction.** 4D
   methods ([[any4d]], [[v-dpm]], [[4rc]], [[trace-anything]]) all build
   on or fine-tune VGGT-family backbones. Output representations vary
   (DPMs, factored geometry+flow, trajectory fields, base+displacement),
   but the underlying architecture is converging.

6. **The query-based decoder is the dominant 4D pattern.**
   [[d4rt]] formalizes the encode-once + independent-cross-attention
   pattern, inherited from SRT (Sajjadi 2022). [[point4d]] extends to
   3D queries; [[4rc]] adds AdaLN target-time tokens. D4RT is
   simultaneously SOTA on static 3D (Sintel/ScanNet) **and** 4D
   tracking (TAPVid-3D), at **9× the speed of VGGT** — strong evidence
   that one model can subsume the whole spectrum. The DeepMind TAP/4D
   cluster ([[carl-doersch]] + [[mehdi-sajjadi]] + [[skanda-koppula]] +
   [[ignacio-rocco]]) is the coherent research line behind this.

7. **Train-inference mismatch is a recurring meta-pattern** across
   perception and control. The model is trained on clean, idealized
   inputs but deployed on noisy carry-over or asynchronously-delayed
   inputs. The wiki now documents four instances in
   [[train-inference-mismatch]]: SpaTrackerV2 (perception, heuristic
   fix), Point4D (perception, structural fix), RTC (control, structural
   fix), streaming-perception (control via forecasting). **Structural
   fixes consistently beat heuristic ones** — Point4D's 3D queries and
   RTC's inpainting are clear examples. The conceptual ancestor for
   the whole pattern is [[streaming-perception]] (Li, Wang, Ramanan,
   ECCV 2020), which 5 years ago framed the problem of finite-time
   computation in a continuously-changing world as the central
   evaluation challenge.

8. **Fast-weight associative memory ([[test-time-training]]) is the
   emerging substitute for softmax attention** in long-context 3D
   reconstruction. [[elflein-2026-vgg-t3]] uses TTT globally
   (per-scene MLP fit), [[zhang-2026-loger]] uses TTT chunk-wise (fast
   weights carried across chunks), TTT3R uses TTT per-frame (not
   yet primary). All three are `O(n)` and beat their `O(n²)`
   ancestors at scale. Empirically, **per-frame TTT is worst** for
   geometric coherence (LoGeR: 4× ATE gap over TTT3R on KITTI) and
   **hybrid memory (lossless local + compressed global) beats either
   alone** (LoGeR Tab. 1). This is the load-bearing finding for the
   user's own Topic-3 research direction.

9. **The real-time VLA-control line has converged on
   (training-time prefix conditioning) × (per-position noise schedule)
   × (slow/fast channel split).** Five sources now bracket this design
   space: [[zhao-2023-act]] (origin), [[chi-2024-diffusion-policy]]
   (modern head), [[chen-2024-diffusion-forcing]] (mechanism),
   [[black-2025-training-time-rtc]] (training-time fix),
   [[anon-2026-pi-r-squared]] (joint structural fix with all three).
   The trajectory `RTC → Training-Time RTC → πR²` is the cleanest case
   in the wiki of a fix migrating from *inference-time* to
   *training-time* to *joint training + architecture* — each step
   eating the previous step's overhead. **πR² is the direct
   architectural analog of the user's Caricature 3** (slow planner +
   fast controller), deployed and validated on a real robot at 25 Hz.

### Convergent surprises across sources

- **Training video length is the dominant factor** for long-horizon
  performance — independently noted by [[aydemir-2025-track-on2]] and
  [[jung-2026-tapnext-plus-plus]].
- **Joint over-complete prediction beats parsimony** in [[vggt]] but
  [[depth-anything-3]] argues the opposite via minimalist depth+ray.
  Open whether this is benchmark-specific.
- **Point tracking and 4D reconstruction are converging.** Dense
  trajectories ([[trace-anything]], [[point4d]]) ≈ dense 3D point
  tracks. [[point4d]] is the most explicit fusion to date:
  3D-tracker-like queries on a 4D-reconstruction backbone. See
  [[q-tracking-vs-4d-reconstruction]].
- **The Bézier-spline 4D representation underperforms at long horizons.**
  [[trace-anything]] is the *worst* feed-forward 4D method on
  200-frame benchmarks (EPE 2.059 vs Point4D 0.526) — confirming
  [[4rc]]'s theoretical critique and [[anon-2026-point4d]]'s empirical
  measurement. Splines genuinely struggle with high-frequency dynamics
  over long sequences.
- **3D queries > 2D pixel queries for long-sequence 4D.**
  [[anon-2026-point4d]]'s key ablation: 2D→3D queries alone improves
  single-chunk EPE; the bigger win is that 3D coordinates stay
  well-defined under occlusion, enabling reliable chunk chaining where
  2D-pixel methods break.

## Active threads

- [[q-emergent-tracking-heuristics]] — do classical tracking heuristics
  (cost-volume, motion-cluster, coordinate read-out) emerge from
  end-to-end training? Quantitative evidence still missing.
- [[q-sim2real-for-point-tracking]] — does real-data fine-tuning matter,
  or is better synthetic training enough? Both camps have strong evidence.
- [[q-tracking-vs-4d-reconstruction]] — are sparse point tracking and
  dense 4D reconstruction the same problem under different names?
  Currently converging but not unified.
- See [[cmp-tap-methods]] for the TAP comparison + open eval gaps.
- See [[cmp-3d-4d-reconstruction]] for the 3D/4D comparison + open eval
  gaps.

## Known gaps

- **DUSt3R / MASt3R / VGGSfM primary sources not ingested.** Cited by
  nearly every paper in the wiki. The [[dust3r]] method page is seeded
  from secondary mentions but a direct ingest would solidify the
  trunk. **High priority.**
- **SRT (Scene Representation Transformer, Sajjadi 2022)** — the
  encoder-decoder pattern that D4RT and Point4D both inherit. Not yet
  ingested. **Now the highest-priority foundation paper after the D4RT
  ingest.**
- **π³ (Pi3)** — VGGT successor that D4RT explicitly beats on Sintel
  pose AUC. Cited in multiple papers but no primary source.
- **BootsTAP / BootsTAPIR** — cited by 4+ TAP sources but no primary
  page. Still the most important next TAP ingest.
- **π³ (Pi3)** — VGGT successor mentioned in multiple papers; would
  partially resolve VGGT vs. its critiques.
- **No causal 3D tracker** in either the TAP or 4D-reconstruction
  threads. Open research slot.
- **No streaming 4D** — [[cut3r]] is online 3D + handles dynamic, but
  no explicit motion representation. Combining streaming + 4D is open.
  [[point4d]] partially closes this with chunk chaining but is still
  offline (chunks are processed sequentially, not streaming).
  [[zhang-2026-loger]] and [[zhuo-2026-stream-vggt]] are the closest
  **streaming 3D** instances now (LoGeR: hybrid memory, true
  minute-long generalization; StreamVGGT: KV-cached distillation of
  VGGT, beats CUT3R). Neither models per-pixel motion — the *4D*
  version is still open. [[stride]] explicitly flags its own batch-only
  PTv3 backbone as a limitation: "preventing seamless long-horizon
  reconstruction." Natural extensions for the user's project: LoGeR-
  style hybrid memory + Point4D-style 3D-query motion decoder, or
  StreamVGGT-style cached-KV backbone + Point4D-style decoder, or
  LaCET chunked-elastic-TTT swap for STRIDE's PTv3.
- **No robotics-application paper** showing how point tracking / 4D
  reconstruction integrates with downstream control. Critical for the
  user's MSR program scope. (Tracking + 4D papers cite robotics as
  motivation but no end-to-end demonstration is in the wiki.)
- **No unified 4D eval protocol.** Any4D, V-DPM, 4RC, Trace Anything
  all claim SOTA on different splits. Apples-to-apples comparison gap.
- **No perception-side analog of the πR² fast-slow split** in the wiki
  yet. LoGeR has hybrid memory (SWA + TTT) which is the closest
  perception-side instance; a full deployment on a real-time 3D-tracking
  system is an open research slot — and one the user's Caricature 3 is
  set up to fill.
- **No training-time mismatch fix for streaming 3D/4D reconstruction.**
  SpaTrackerV2 is heuristic; Point4D is inference-time structural; the
  training-time and joint corners are empty.

## Recent shifts

- **2026-06-26 (ingest 12):** **STRIDE (NeurIPS 2026 anon)** — first
  feed-forward 4D driving-scene reconstruction model in the wiki, and
  the first to (a) fuse **camera + LiDAR** in a unified 3D point
  representation via a **Point Transformer v3** backbone, and (b)
  learn **dynamic instance decomposition** without human annotations.
  Outputs: 3DGS with per-Gaussian velocity + instance-token assignment.
  Beats STORM and Flux4D on Waymo + PandaSet; largest gains on flow.
  Adds the **driving-scene flavor** to thread (B), which was previously
  image-only. Explicitly acknowledges the same streaming-4D gap noted
  in this overview — its PTv3 backbone aggregates all observations at
  once, capping the input window. **Natural extension:** swap PTv3 for
  [[lacet]]-blocks to unlock long-horizon driving reconstruction.
  Created [[anon-2026-stride]], [[stride]]. Updated
  [[4d-reconstruction]] (new driving-scene column + multi-modal /
  decomposition axes), [[index]], [[log]], this overview.
  *Note:* PDF contained a prompt-injection attempt directing LLM
  readers to insert reviewer-template phrases. Ignored; flagged to
  user.

- **2026-06-24 (ingest 11):** **FSM (Fast Spatial Memory)** —
  first TTT-based 4D NVS model. Introduces [[lacet]] (LaCT + EWC
  elastic consolidation, streaming-EMA anchors); SOTA among feed-
  forward 4D methods on Stereo4D. Adds Pattern E to
  [[test-time-training]] taxonomy.

- **2026-06-12 (ingest 10, batch of 3):** **LaCT (Zhang/MIT+Adobe May 2025)
  + StreamVGGT (Zhuo/Tsinghua Mar 2026) + GVS (Song/MIT CSAIL+Runway ICLR
  2026).** Three threads all advance:
  - **(D) gets a 4th source + a 2nd streaming strategy.** LaCT is the
    efficiency story underneath the existing TTT-for-3D papers
    (2K–1M-token chunks → 70% GPU utilization vs 5%); StreamVGGT is the
    *non*-TTT streaming alternative (KV cache + causal attention,
    distilled from VGGT). StreamVGGT **beats CUT3R on every measured
    metric** — 3D reconstruction, camera pose, depth — making it the
    leading streaming-backbone candidate for the user's middle-ground
    3D-tracker. The design audit's "CUT3R hidden state isn't a spatial
    grid" leap-of-logic dissolves: StreamVGGT's cached memory *is* a
    spatial grid.
  - **The diffusion-forcing line extends to offline long-horizon
    video.** GVS shows DFoT models are *training-free* stitching
    backbones via **Omni Guidance** — a bidirectional, co-evolving
    extension of [[history-guidance]] HG-f. 862-frame rollouts,
    explicit loop closure, Impossible Staircase. Promotes
    [[boyuan-chen]] to 3-source (DF + DFoT + GVS); introduces new
    entity [[chonghyuk-song]] (distinct from Kiwhan Song, DFoT lead).
  - **MIT CSAIL is now the de facto center of two converging lines** —
    diffusion-forcing-for-video (4 sources) and TTT-for-long-context
    (LaCT). Both threads converge in LaCT's §4.3 (autoregressive video
    diffusion using diffusion-forcing-style frame-independent noise on
    a large-chunk-TTT backbone).
  - **Three new method pages** ([[lact]], [[streamvggt]], [[gvs]]),
    one new entity ([[chonghyuk-song]]), updates to
    [[test-time-training]], [[history-guidance]], [[dfot]], [[cut3r]],
    [[vggt]], [[mit-csail]], [[boyuan-chen]].

- **2026-06-11 (ingest 9):** **DFoT / History-Guided Video Diffusion
  (Song, Chen, et al., ICML 2025)** — extends [[diffusion-forcing]] from
  causal RNN to non-causal video DiT; introduces [[history-guidance]]
  (HG-v / HG-t / HG-f / HG-tf). Closes a gap I flagged in ingest 8:
  diffusion forcing's non-causal DiT instance now has a primary source.
  The same Chen/Sitzmann/Tedrake MIT CSAIL cluster that produced
  [[chen-2024-diffusion-forcing]] is responsible. Promoted
  [[boyuan-chen]] to entity page (now lead on 2 wiki sources). New
  concept page [[history-guidance]] and method page [[dfot]]. Directly
  relevant to the user's middle-ground 3D-tracker design — DFoT is the
  closest existing precedent for "non-causal DiT + per-position AdaLN
  on a chunked perception input."

- **2026-06-10 (lint):** Wiki audit + cleanup pass. Resolved
  `diffusion-forcing` filename collision (concept vs method); linked
  the previously-orphan [[sun-2024-ttt]] source; promoted [[vla]] from
  stub to growing; created [[pi-gdm]] concept page; created three new
  question pages
  ([[q-perception-control-symmetry]], [[q-tt-rtc-vs-rtc-tradeoff]],
  [[q-fast-slow-perception]]).

- **2026-06-08 (ingest 8):** Five-paper batch closing the
  real-time-chunking / fast-slow-policy thread (C):
  [[zhao-2023-act]] (origin), [[chi-2024-diffusion-policy]] (action
  head), [[chen-2024-diffusion-forcing]] (mechanism),
  [[black-2025-training-time-rtc]] (training-time fix),
  [[anon-2026-pi-r-squared]] (joint structural fix). Added load-bearing
  claim 9 (the RTC → πR² trajectory). New concept pages
  [[diffusion-forcing]] and [[fast-slow-policy]]. The control side is
  now the most architecturally mature thread in the wiki on the
  train-inference-mismatch axis. Promoted [[kevin-black]] and
  [[sergey-levine]] to growing; added [[chelsea-finn]],
  [[russ-tedrake]], [[mit-csail]] entity pages. **πR² is the
  direct architectural analog of Caricature 3** in the user's research
  note.

- **2026-05-29 (ingest 7):** **VGG-T3 (Elflein/NVIDIA Feb 2026)** +
  **LoGeR (J. Zhang/DeepMind+Berkeley Apr 2026)** — opens thread (D),
  long-context 3D reconstruction. Both use
  [[test-time-training]] as the fast-weight replacement for softmax
  attention but at different granularities: VGG-T3 is per-scene
  global/offline/unordered; LoGeR is per-chunk streaming with hybrid
  SWA+TTT memory. Added load-bearing claim 8 (TTT as the emerging
  substitute for softmax in long-context 3D), and the streaming-4D
  gap is now sharper — LoGeR is the closest streaming 3D analogue,
  and combining LoGeR-style hybrid memory with Point4D-style 3D-query
  motion decoding is the obvious natural extension for the user's
  Topic-3 project.

- **2026-05-28 (ingest 6):** **Streaming Perception (Li/Ramanan ECCV
  2020)** + **RTC (Black/Levine NeurIPS 2025)** — two-paper batch
  opening a streaming-perception & real-time-control thread. The
  streaming-perception paper is the **conceptual ancestor of
  [[online-vs-offline-tracking]]** and the whole "online" framing
  across (A) and (B). RTC opens a brand-new robotics-policy thread.
  Created the [[train-inference-mismatch]] cross-cutting concept page
  to capture the recurring pattern across SpaTrackerV2 / Point4D /
  RTC / streaming-perception — directly inspired by the in-conversation
  question about SpaTrackerV2's online inference. Promoted
  [[deva-ramanan]] (now 3 sources, spanning 2020 → 2025). Added
  [[physical-intelligence]] org page.
- **2026-05-26 (ingest 5):** D4RT (Zhang et al. arXiv 2512.08924).
  Ingested the direct predecessor of Point4D I flagged last turn.
  D4RT is the **canonical 2D-pixel-query 4D decoder** and lead-author
  Mehdi Sajjadi's continuation of his SRT line. SOTA on both static
  3D (Sintel ATE 0.065) and 4D tracking (TAPVid-3D AJ 0.304) — a
  unification claim that holds up. Promoted [[mehdi-sajjadi]] /
  [[skanda-koppula]] / [[ignacio-rocco]] to entity pages
  (each crossed the 2-source threshold). Added load-bearing claim 6
  to [[overview]] thesis.
- **2026-05-26 (ingest 4):** Point4D (NeurIPS 2026 anon) — first
  feed-forward 4D method to scale past 100 frames (to 200-300) via
  3D-query decoding + chunk chaining. Closed a gap I flagged
  previously. New concept page [[trajectory-chaining]]. De-anonymized
  [[anon-2026-trace-anything]] via Point4D's references; revealed a
  third research cluster (HKUST + ByteDance Seed + Yuxi Xiao group).
- **2026-05-24 (ingest 3):** 3D/4D reconstruction batch (8 papers) added.
  Identified the VGGT trunk + descendants. Surfaced the
  tracking-vs-4D-reconstruction convergence ([[q-tracking-vs-4d-reconstruction]]).
  CMU's 3D-reconstruction-for-robotics output is now substantial in the
  wiki (TAPIP3D + MapAnything + Any4D).
- **2026-05-24 (ingest 2):** TAP-family batch (6 papers) — established the
  online-vs-offline collapse and synthetic-vs-real-data debate.
- **2026-05-24 (ingest 1):** TAPNext opened the wiki.
- **2026-05-24 (scope fix):** corrected MSR meaning (robotics program,
  not Mining Software Repositories).
