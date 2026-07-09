---
type: overview
title: Research Wiki — Overview
status: growing
tags: [meta]
created: 2026-05-24
updated: 2026-07-09
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

## Current thesis (July 2026, after 44 sources)

The wiki has dense coverage of two intersecting perception sub-areas,
two now-fleshed-out adjacent threads (streaming/control, and
long-context 3D), and — new with this ingest — a **fifth thread**:
manipulation policies that use point tracks as their interface.

**(A) Tracking Any Point (TAP).** 9 sources now covering the full
historical arc: founding **PIPs** (CMU, ECCV 2022) → canonical
**TAPIR** (DeepMind+Oxford, ICCV 2023) → the DeepMind TAPNext line, the
Meta/Oxford CoTracker line, the Koç Track-On line, and the 3D
extensions (TAPIP3D from CMU, SpatialTrackerV2). The two
just-ingested foundation papers are the predecessor every later TAP
method cites and that [[zholus-2025-tapnext]] explicitly argues against.

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
**5 sources** with this batch: [[elflein-2026-vgg-t3]] (NVIDIA;
TTT-compressed scene as MLP, `O(n)` global/offline/unordered),
[[zhang-2026-loger]] (DeepMind+Berkeley; chunked feed-forward with
**hybrid memory** = SWA + TTT), [[zhang-2025-lact]] (MIT+Adobe;
**Large-Chunk TTT** = the missing efficiency layer underneath the
TTT-for-3D papers, plus 14B-parameter AR video diffusion),
[[zhuo-2026-stream-vggt]] (Tsinghua; **causal-attention + KV-cached
streaming distillation** of VGGT — beats CUT3R on every measured
benchmark), and [[wu-2025-point3r]] (Tsinghua; **explicit spatial
pointer memory** — memory scales with explored space, not time; same
group as StreamVGGT). Three distinct streaming strategies now
articulated: **(D1) test-time-trained fast weights** (VGG-T3, LoGeR,
LaCT — fast weights *are* the memory), **(D2) cached KV /
causal-attention** (StreamVGGT — memory scales with time), and **(D3)
geometry-indexed pointer memory** (Point3R — memory scales with
explored space; CUT3R sits between D2 and D3 as fixed-length implicit
state). All are O(n) per frame; trade-offs are unsettled — Point3R
wins long-sequence 3D quality but loses camera pose; StreamVGGT is
higher overall quality but linear memory growth.

**(E) Manipulation policies via point tracks.** Now **7 sources** —
the wiki's end-to-end demonstration of how TAP infrastructure feeds
robot manipulation. Arc spans 2024–2026:
- 2024: [[bharadhwaj-2024-track2act]] (CMU+Meta; DiT denoiser of 2D
  point tracks from web video + PnP + residual policy on Spot) and
  [[xu-2024-im2flow2act]] (Stanford+Columbia; object-only flow +
  sim-trained diffusion policy; zero real-robot data).
- 2025: [[zhi-2025-3dflowaction]] (SCUT+Tencent; lifts the line to
  3D flow + GPT-4o closed-loop verifier + optimization policy +
  ManiFlow-110k cross-source dataset).
- 2026: [[kuang-2026-dex4d]] (**CMU** Fragkiadaki×Tulsiani; first
  dexterous application: AP2AP sim-to-real RL + Paired Point
  Encoding), [[kim-2026-pri4r]] (KAIST+LG+CMU; inverts the role —
  point tracks as **privileged training-time supervision** for π₀ /
  π₀.₅ / OpenVLA-OFT, head discarded at inference, +13.2% RoboCasa),
  [[lee-2026-mu0]] (**UMD + SNU**; 3D trace-space **world model** —
  semantic DINOv2 keypoints, globally aligned 3D, event-centric
  captions, B-spline control-point flow matching; frozen features
  feed action expert; beats action-labeled π₀ on RoboCasa365 with
  zero action pretraining), and [[huang-2026-pointworld]]
  (**Stanford + NVIDIA**; action-conditioned **3D dynamics world
  model** — PTv3 + DINOv3 + URDF-derived robot flows; chunked 10-step
  @ 0.1s; ~2M-trajectory DROID+B1K; MPPI planner for zero-shot
  in-the-wild rigid / deformable / articulated / tool-use Franka
  manipulation; **published log-linear scaling laws** for 3D
  dynamics).

Four design axes now structure this family:
1. **What role do tracks play at inference?** Four poles:
   conditioning input (Track2Act / Im2Flow2Act / 3DFlowAction /
   Dex4D), privileged supervision (Pri4R), WM features (µ0), and
   action-conditioned dynamics simulator (PointWorld).
2. **Language-conditioned trace generator vs. action-conditioned
   dynamics WM** — first six answer "what motion should happen?";
   PointWorld answers "what happens given this action?" Different
   question, different stack. PointWorld pairs with MPPI planner;
   others pair with learned action policies.
3. **Heuristic vs. learned action policy vs. MPC** — Track2Act
   (open-loop) / 3DFlowAction (optimization) vs. Im2Flow2Act (sim BC)
   / Dex4D (sim-to-real RL) / µ0 (flow-matching action expert) vs.
   PointWorld (MPPI over WM rollouts).
4. **Fixed grid vs. semantic keypoints vs. dense per-pixel** — five
   prior use fixed-grid / dense sparse sampling; µ0 uses semantic
   DINOv2 clusters; PointWorld uses **dense per-pixel** back-projection
   (highest resolution, most compute; enables contact reasoning at
   articulation joints).

See [[point-tracks-as-manipulation-interface]] (concept) and
[[cmp-point-track-manipulation]] (comparison).

(A) and (B) are **visibly converging** — see
[[q-tracking-vs-4d-reconstruction]]. (C) sits underneath both: it's the
evaluation framework + control-side analogue that the user's questions
about online inference and train-inference mismatch all live within.
**(D) is the perception-side analogue of (C)** — it asks the same
"finite-time computation on growing input" question for 3D
reconstruction. (D) is directly load-bearing for the user's own
research project (Topic 3 in `raw/notes/` — long-term real-time 3D
tracking architectures).

**(E) is the downstream consumer of (A).** The user's institution
(CMU) is now the single largest contributor to the manipulation-from-
tracks line — Track2Act / Dex4D both come from the Tulsiani group;
Dex4D specifically combines the Fragkiadaki (TAP origin) and Tulsiani
(visual geometry) lines into a single Fragkiadaki × Tulsiani paper.
This closes the "no robotics application of point tracking is in the
wiki" gap that was previously listed under Known gaps.

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
- **BootsTAP / BootsTAPIR** — cited by 5+ TAP sources (now including
  [[doersch-2023-tapir]] as its parent) but no primary page. Still the
  most important next TAP ingest — the semi-supervised real-data
  fine-tune that closes the synthetic-to-real gap on top of TAPIR.
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
- ~~**No robotics-application paper** showing how point tracking / 4D
  reconstruction integrates with downstream control.~~ **Closed by
  the 2026-06-27 ingest** — Track2Act / Im2Flow2Act / 3DFlowAction /
  Dex4D / Pri4R now occupy this slot. Remaining gap: no source uses
  the *feed-forward 4D-reconstruction* backbones (VGGT / Point4D /
  STRIDE) as the perception side of a manipulation pipeline — all
  five sources use TAP-style trackers (CoTracker / TAPIR) rather than
  4D-reconstruction outputs. Combining VGGT/Point4D-as-perception +
  Dex4D-style policy is an open slot.
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

- **2026-07-09 (ingest 18, single):** [[allshire-2025-videomimic]]
  (**UC Berkeley**, CoRL 2025 Best Student Paper) — first
  **humanoid whole-body control from monocular video** in the wiki.
  Opens a new corner (thread F, seeded with one source; not yet a
  full thread):
  - **New sub-area: humanoid learning-from-video.** All prior (E)
    sources address gripper / hand manipulation on tabletop robots.
    VideoMimic is 23-DoF whole-body locomotion + contextual sitting /
    climbing on real Unitree G1. Different problem, different substrate.
  - **Real-to-sim-to-real** as a paradigm. Distinct from (E)'s
    direct-training patterns. Bridges via joint 4D human-scene
    reconstruction (SMPL + MegaSAM + JAX Levenberg-Marquardt joint
    opt with SMPL-height metric prior) → simulator-ready mesh + motion
    pair → 4-stage RL (MoCap pretrain → scene-conditioned DeepMimic
    → DAgger distill → PPO fine-tune).
  - **4D reconstruction downstream in policy training.** Prior wiki
    (B) sources ([[trace-anything]] / [[point4d]] / [[v-dpm]]) build
    4D reconstruction as an end goal. VideoMimic **consumes** 4D
    recon as a substrate for humanoid RL. First wiki source
    demonstrating downstream 4D use in real-robot control. Extends
    the "what is 4D good for?" answer beyond tracking + rendering.
  - **Single unified policy** for stairs + sitting + rough terrain —
    context (heightmap + root direction) replaces skill selection.
    Emergent single-leg-hop recovery.
  - **UC Berkeley bumps to 5 sources.** BAIR now spans feed-forward
    3D ([[cut3r]]), real-time control ([[rtc]] + [[training-time-rtc]]),
    long-context 3D ([[loger]]), and humanoid-from-video (VideoMimic).
    Two subcultures — Efros / Kanazawa CV line vs. Sergey Levine /
    PI robotics line — converge at BAIR + Kanazawa + Malik + Abbeel +
    Darrell on this paper.
  - **[[junyi-zhang]] promoted to 2 sources.** LoGeR (long-context
    3D) + VideoMimic (humanoid-from-video) — same PhD student across
    two threads within the Darrell lab. Cross-thread reach confirmed.
    Trevor Darrell now the shared senior advisor across those two —
    hold on Darrell person page pending a 3rd source.
  - **New held-at-1-source cluster:** Arthur Allshire (also IsaacGym
    co-author), Hongsuk Choi, David McAllister, Angjoo Kanazawa
    (foundational monocular human motion — HMR, SLAHMR line), Jitendra
    Malik (senior; RMA / legged / humanoid), Pieter Abbeel (senior;
    RL / VLA / PI advisor), Trevor Darrell (senior; LoGeR + VideoMimic
    → 2 sources but wait for 3rd).
  - **Priority ingest bumps:**
    (1) **DeepMimic** (Peng et al. SIGGRAPH 2018) — architectural
        ancestor of VideoMimic's Stage 2. Still #1 for humanoid
        learning-from-video coherence.
    (2) **SFV** (Peng, Kanazawa, Malik, Abbeel, Levine 2018) — direct
        predecessor from same senior authors. Would promote Kanazawa
        + Malik + Abbeel + Levine person pages.
    (3) **MonST3R** (Zhang et al. 2024/25) — cited by **9+ wiki
        sources** now. Beyond every earlier promotion threshold.
        Junyi Zhang lead. Ingest overdue.
    (4) **SLAHMR** (Ye et al. 2023) — VideoMimic bootstrap.
    (5) **MegaSAM** — perception substrate reused across VideoMimic +
        PointWorld annotation pipeline.
    (6) TraceGen still #1 for (E).

- **2026-07-09 (ingest 17, single):** [[huang-2026-pointworld]]
  (**Stanford + NVIDIA**, arXiv Jan 2026) — an **action-conditioned
  3D dynamics WM** joins thread (E). Concretely:
  - **(E) bumps to 7 sources.** Introduces a **fourth role for
    point tracks** — action-conditioned dynamics simulator — alongside
    the existing three (conditioning input; privileged supervision;
    WM features). Concept + comparison pages updated to a 4-axis
    framing.
  - **Shared state-action representation in 3D**: scene points from
    RGB-D back-projection, robot action points from URDF forward
    kinematics. Concatenated into a single point cloud → PTv3
    backbone. Sharp contrast with µ0's SmolVLM + Trace Expert
    (language-conditioned, semantic keypoints) — PointWorld is dense
    per-pixel and action-conditioned.
  - **First published scaling laws for 3D dynamics** — log-linear
    ℓ2-mover reduction across 50M → 1B params and 5% → 100% data on
    DROID. Analogous to LM / vision scaling laws; validates PTv3 +
    DINOv3 as the scalable substrate.
  - **DROID 3D annotation pipeline is reusable infrastructure.**
    FoundationStereo depth + VGGT-init extrinsics + robot-mesh
    optimization + CoTracker3 → recovers 60% of DROID (~200 hours)
    with reliable 3D flows. Same recipe applicable to any RGB-D
    manipulation corpus.
  - **Gripper-only action flows beat whole-body flows** on DROID +
    B1K — sparse gripper points concentrate learning signal; dense
    whole-body flows dilute. Design finding: contact-relevant
    geometry is where the signal lives.
  - **Chunked > autoregressive** (single forward pass predicts 10-step
    horizon jointly; 10× faster than AR sliding-window, less drift).
  - **PTv3 is now a 2-source backbone** in the wiki (PointWorld +
    [[stride]]). Tool-page candidate.
  - **DINOv3 has its first primary source** — encoder for scene point
    features. Frozen throughout.
  - **New role in (E) means direct µ0 vs. PointWorld comparison is
    difficult** — different benchmarks (RoboCasa365 + UR3 vs. Franka
    in-the-wild MPC on rigid / deformable / articulated / tool-use).
    Complementary paradigms; combining them (µ0 trace as MPPI target,
    PointWorld as simulator) is an unexplored slot.
  - **Depth requirement is the sharpest limitation.** PointWorld
    needs calibrated RGB-D + accurate depth (FoundationStereo in
    deployment). No pure-RGB pretraining path — cannot ingest
    human-hand video without depth. µ0's TraceExtract handles this via
    global-local reconstruction from RGB alone.
  - **Priority-ingest bumps:** (1) TraceGen still the sharpest µ0
    predecessor; (2) VoxPoser / ReKep (same Wenlong Huang author line,
    VLM-based waypoint methods; would promote him to multi-source),
    (3) GBND (Ai et al. Sci. Robotics 2025 — PointWorld's baseline),
    (4) FoundationStereo (depth substrate), (5) DROID + BEHAVIOR-1K
    dataset pages given multi-source use.
  - **New entities held at 1 source:** Wenlong Huang (Stanford; also
    VoxPoser / ReKep), Li Fei-Fei, Dieter Fox, Kaichun Mo, Yu-Wei
    Chao, Arsalan Mousavian, Ming-Yu Liu (all NVIDIA robotics), plus
    Stanford as an org (currently 3+ implicit sources: Im2Flow2Act
    Stanford collab, ACT via Finn's Stanford IRIS, PointWorld — but
    diffuse across labs; hold on the org page).

- **2026-07-09 (ingest 16, single):** [[lee-2026-mu0]] (**UMD + SNU**,
  arXiv Jun 2026) — a **3D trace-space world model** joins thread (E).
  Concretely:
  - **(E) bumps to 6 sources.** µ0 introduces a third role for point
    tracks (world-model target with frozen features) alongside the
    existing two (conditioning input; privileged supervision). The
    concept + comparison pages updated to a 3-role axis.
  - **Two mechanism upgrades vs. TraceGen (its direct predecessor):**
    (1) DINOv2 entity-cluster semantic keypoints replace fixed grids —
    directly targets the "under-sample tool tips / contact patches"
    failure mode all prior work shared; (2) globally aligned 3D via
    chunk-wise reconstruction removes camera / object motion
    conflation; (3) event-centric captions (Savitzky–Golay
    acceleration valleys → chunk boundaries) replace episode captions.
    TraceExtract curates ~8× the scale of TraceGen's dataset.
  - **B-spline control points + flow matching** replaces DiT waypoint
    denoising (Track2Act). Same primitive family as
    [[trace-anything]] — µ0 confirms the primitive is fine at
    manipulation horizons (T ≤ 32) even though it's the weakest
    representation at 200-frame 4D reconstruction. Adds L_done
    (validity) + L_rig (semantic-cluster rigidity) on top of flow.
  - **Video-only pretraining beats action-labeled π₀** on RoboCasa365
    (30.25 vs 25.25% avg SR). π₀.₅ still wins (42%) but not data-
    matched. Real-world UR3: µ0 91.7% avg, +18.4 pt over VLM+AE with
    same backbone (isolates the trace-expert contribution), +10 pt
    over TraceGen, +11.7 over π₀.₅.
  - **µ0 is the first end-to-end demonstration in the wiki that a
    frozen video-only WM outperforms action-labeled VLA pretraining
    at scale.** Reframes π₀-style pretraining as an intermediate
    representation choice, not a necessity.
  - **TraceGen is now the highest-priority next manipulation ingest.**
    Same first author (Seungjae Lee, UMD). Predecessor + primary
    baseline throughout µ0. CVPR 2026, so a full source can be built.
  - **New entities held at 1 source (below threshold):** Seungjae Lee
    (UMD; would be 2 sources once TraceGen ingested), Yoonkyo Jung
    (UMD, µ0 equal), Jia-Bin Huang (UMD advisor), Furong Huang (UMD
    advisor), H. Jin Kim (SNU), UMD org, SNU org. First UMD paper in
    the wiki; TraceGen would immediately promote UMD to 2-source.
  - **Priority ingest bumps:** (1) TraceGen (Lee et al. CVPR 2026),
    (2) NovaFlow (Li 2026a — Dex4D's baseline, cited again here),
    (3) BootsTAP / BootsTAPIR (still #1 TAP hole overall),
    (4) Dream2Flow (Dharmarajan 2026 — µ0's 3D baseline).

- **2026-07-02 (ingest 15, single):** [[wu-2025-point3r]] (Tsinghua
  Automation, NeurIPS 2025) — third distinct streaming memory design
  for DUSt3R-lineage recon. Each memory unit is a `(3D position ∈ R^3,
  feature ∈ R^768)` pair; distance-based fusion caps growth; 3D
  hierarchical RoPE injects continuous 3D position into
  pointer-image attention. Implications:
  - **(D) bumps to 5 sources.** Streaming-3D thread now covers three
    memory paradigms: fixed implicit state ([[cut3r]]), growing KV
    cache ([[streamvggt]]), and explicit 3D-indexed pointers
    ([[point3r]]). Design axis is what memory scales with — nothing
    (fixed budget), time (KV cache), or explored space (pointer set).
  - **Long-sequence result is the standout.** Point3R vs [[cut3r]] on
    500-1000-frame 7-scenes: Acc mean 0.071 vs 0.238; NRGBD 400-900
    frames: 0.110 vs 0.372. Fixed implicit state bleeds; geometry-
    indexed memory doesn't.
  - **Tsinghua Automation as a streaming-recon center.** Zheng
    (Wenzhao) is 2nd author on Point3R and project lead on StreamVGGT
    — same group producing two complementary streaming designs in the
    same year. Bump Tsinghua to a first-org candidate (2 sources
    threshold met; consider org page next batch).
  - **Camera pose gap remains.** Point3R lags [[cut3r]] and MonST3R-GA
    on ScanNet/Sintel/TUM-dynamics ATE/RPE. Authors flag as future
    work — growing pointer count may add spatial interference to pose
    head. Suggests explicit pointer memory needs a pose-aware
    disentanglement.
  - **Priority ingest bumps.** Spann3R (cache-style memory — the third
    corner of the (D) design space still non-ingested) is now
    higher-priority.

- **2026-06-27 (ingest 14, batch of 5):** Five-paper batch opening
  the new **(E) Manipulation via point tracks** thread:
  [[bharadhwaj-2024-track2act]] (CMU+Meta, ECCV 2024 — founding
  paper), [[xu-2024-im2flow2act]] (Stanford+Columbia, CoRL 2024),
  [[zhi-2025-3dflowaction]] (SCUT+Tencent, Jun 2025),
  [[kuang-2026-dex4d]] (**CMU** Fragkiadaki×Tulsiani, Feb 2026 —
  first dexterous), and [[kim-2026-pri4r]] (KAIST+LG+CMU, Mar 2026 —
  privileged-supervision inversion). Implications:
  - **CMU bumps to 9 sources** (was 6) — now the institutional center
    not just of deep TAP but of TAP-driven manipulation. **The CMU
    TAP-line directly feeds the CMU manipulation line**: PIPs (2022,
    Fragkiadaki) → TAPIP3D (2025, Fragkiadaki) → Dex4D (2026,
    Fragkiadaki + Tulsiani) is a single 4-year arc inside one
    institution.
  - **[[shubham-tulsiani]] bumps to 3 sources** (Point4D + Track2Act
    + Dex4D); **[[katerina-fragkiadaki]] bumps to 3** (PIPs +
    TAPIP3D + Dex4D). Both at "growing" status.
  - **6 new person pages:** [[homanga-bharadhwaj]] (Track2Act lead;
    Tulsiani PhD), [[yuxuan-kuang]] (Dex4D lead), [[sungjae-park]]
    (Dex4D co-lead), [[shuran-song]] (Im2Flow2Act senior),
    [[cheng-chi]] (Diffusion Policy + Im2Flow2Act),
    [[laszlo-jeni]] (CMU faculty; Pri4R co-senior).
  - **Dex4D explicitly acknowledges the user** (Minsik Jeon) for
    presentation feedback — strong evidence the user is integrated
    into the Tulsiani/Fragkiadaki manipulation cluster.
  - **The previously-listed "no robotics application" gap closes.**
    Remaining gap: no source combines a 4D-reconstruction backbone
    (VGGT/Point4D/STRIDE) with this policy family — all five use
    TAP-style trackers instead.
  - Two design axes structure the family: **conditioning vs.
    privileged supervision** (Track2Act/Im2Flow2Act/3DFlowAction/
    Dex4D vs. Pri4R) and **heuristic/optimization vs. learned action
    policy** (Track2Act-OL/3DFlowAction vs. Im2Flow2Act/Dex4D/
    Track2Act-residual).
  - 5 new source pages, 5 new method pages, 1 new concept page
    ([[point-tracks-as-manipulation-interface]]), 1 new comparison
    ([[cmp-point-track-manipulation]]), 6 new person pages, updates
    to [[shubham-tulsiani]], [[katerina-fragkiadaki]], [[cmu-ri]],
    [[meta-ai]], [[vla]], [[point-tracking]], [[index]], [[log]],
    this overview.

- **2026-06-26 (ingest 13, batch of 2):** **PIPs (Harley/Fang/
  Fragkiadaki, CMU, ECCV 2022)** + **TAPIR (Doersch et al.,
  DeepMind+Oxford, ICCV 2023)** — backfilling the **two foundation
  papers of the modern deep TAP sub-field**, both flagged as
  "predecessors not yet ingested" across many existing wiki pages.
  PIPs introduces the **independent-per-point + iterative-MLP-Mixer
  refinement over 8-frame correlation pyramids** template (the
  architectural ancestor of CoTracker, TAPIR, BootsTAPIR, and
  arguably TAPNext) plus the **FlyingThings++** synthetic-with-
  injected-occluders recipe. TAPIR fuses PIPs' refinement with
  **TAP-Net's global per-frame initialization**, replaces the
  fixed-length MLP-Mixer with **depthwise conv over time** (any
  sequence length, parallel inference: ~150 FPS vs. PIPs' chained
  ~3 FPS), and adds **self-supervised position uncertainty**.
  Implications for the wiki's existing threads:
  - **Thread (A) grows from 7 → 9 sources**, and the lineage diagrams
    on [[point-tracking]], [[joint-point-tracking]],
    [[online-vs-offline-tracking]], [[synthetic-to-real-gap]],
    [[trajectory-chaining]], and [[cmp-tap-methods]] now have their
    proper roots.
  - **[[katerina-fragkiadaki]] is now 2-source** (PIPs + TAPIP3D);
    **[[adam-w-harley]] is 2-source** (same line). The CMU TAP-line is
    **2022 PIPs → 2025 TAPIP3D**, one continuous research thread —
    making CMU (via Fragkiadaki) the **institutional origin** of the
    modern deep TAP sub-field. [[cmu-ri]] bumps to 6 sources.
  - **[[carl-doersch]] is now 3-source** (TAPIR + TAPNext + TAPNext++);
    the DeepMind TAP-line's "step 2" finally has its primary page.
  - **[[oxford-vgg]] bumps to 7 sources**, retaining most-prolific-org
    status (Zisserman senior on TAPIR).
  - **Panning MOVi-E** (TAPIR's Kubric-camera-panning fix) — now
    explicit on [[kubric-dataset]] and [[synthetic-to-real-gap]] as
    the canonical training-data variant inherited by BootsTAP, TAPNext,
    TAPNext++.
  - **[[q-emergent-tracking-heuristics]]** now grounds its "control
    condition" properly: PIPs/TAPIR hard-code the cost-volume +
    iterative-refinement + uncertainty patterns that TAPNext argues
    re-emerge for free.
  Created [[harley-2022-pips]], [[doersch-2023-tapir]], [[pips]],
  [[tapir]]. Touched [[cotracker]], [[tapnext]],
  [[karaev-2024-cotracker]], [[point-tracking]],
  [[joint-point-tracking]], [[online-vs-offline-tracking]],
  [[synthetic-to-real-gap]], [[trajectory-chaining]],
  [[adam-w-harley]], [[katerina-fragkiadaki]], [[carl-doersch]],
  [[ignacio-rocco]], [[cmu-ri]], [[google-deepmind]], [[oxford-vgg]],
  [[tap-vid-dataset]], [[kubric-dataset]], [[cmp-tap-methods]],
  [[q-emergent-tracking-heuristics]], [[index]], this overview.

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
