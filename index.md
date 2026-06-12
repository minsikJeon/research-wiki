# Index

Content-oriented catalog of the wiki. Updated on every ingest. The agent reads
this first when answering queries.

See [[AGENTS]] for schema, [[log]] for chronology, [[overview]] for the
evolving thesis.

---

## Sources
*One entry per ingested raw item. Format: `[[slug]] — one-line takeaway (tags)`*

### Point Tracking (TAP)
- [[zholus-2025-tapnext]] — TAP recast as masked-token decoding; SOTA online tracker with no tracking-specific inductive biases (point-tracking, state-space-model, online-tracking)
- [[karaev-2024-cotracker]] — joint multi-point tracking via cross-track attention + proxy tokens; 70K tracks on one GPU (point-tracking, joint-tracking)
- [[karaev-2024-cotracker3]] — simplified arch + random-teacher pseudo-labels on 15K real videos beats BootsTAPIR (15M) (point-tracking, pseudo-labeling)
- [[aydemir-2025-track-on2]] — memory-based causal tracker with classification-first head; synthetic-only beats real-data competitors (point-tracking, online-tracking, memory)
- [[jung-2026-tapnext-plus-plus]] — same TAPNext arch + 1024-frame training + roll aug; SOTA online, introduces AJRD metric (point-tracking, long-sequence)
- [[zhang-2025-tapip3d]] — world-coord 3D tracking via N2N attention on lifted feature cloud; beats 2D SOTA with depth (3d-point-tracking)
- [[xiao-2025-spatialtracker-v2]] — fully end-to-end 3D tracker: jointly trained depth + pose + tracking with differentiable BA (3d-point-tracking, end-to-end)

### Streaming perception & real-time control
- [[li-2020-streaming-perception]] — foundational ECCV 2020 paper; streaming-accuracy metric; offline AP 38.0 → streaming 6.2; tracking + forecasting emerge as necessary (streaming-perception, latency, deva-ramanan)
- [[zhao-2023-act]] — ALOHA + ACT; founding action-chunking paper; CVAE transformer over 4 cams + joint state; 80–90% SR on fine bimanual tasks (robotics, manipulation, action-chunking, imitation-learning)
- [[chi-2024-diffusion-policy]] — visuomotor policy as conditional DDPM over action chunks; +46.9% over prior SOTA; foundation of every modern diffusion-VLA action head (robotics, manipulation, diffusion, action-chunking)
- [[chen-2024-diffusion-forcing]] — MIT CSAIL NeurIPS 2024; per-token-noise sequence modeling; bridges teacher forcing and full-sequence diffusion; underlies πR² and streaming control (sequence-modeling, diffusion, training-paradigm)
- [[song-2025-history-guided-video-diffusion]] — MIT CSAIL ICML 2025; Diffusion Forcing Transformer (DFoT) extends DF to non-causal video DiT; introduces History Guidance (HG-v / HG-t / HG-f / HG-tf); 862-frame rollouts from one image (video-diffusion, diffusion-forcing, classifier-free-guidance)
- [[black-2025-rtc]] — Physical Intelligence; inference-time inpainting for VLA action chunks; freezes committed prefix + soft-masks the rest; robust to >300ms latency (robotics, vla, flow-matching, action-chunking)
- [[black-2025-training-time-rtc]] — Physical Intelligence; drop-in replacement for RTC that moves prefix conditioning to training; zero inference overhead; wins for d ≥ 2 (robotics, vla, training-recipe)
- [[anon-2026-pi-r-squared]] — CoRL 2026 (anon); πR² adds diffusion-forcing staircase schedule + slow/fast channel split; closed-loop 25 Hz on a 7 Hz VLA; +50% relative gain on reactivity-sensitive real-world tasks (robotics, vla, fast-slow-policy, latency-adaptive)

### 3D / 4D Reconstruction (feed-forward foundation models)
- [[wang-2025-vggt]] — foundational feed-forward multi-view 3D transformer; predicts cameras + depth + pointmaps + tracks in one pass (3d-reconstruction, foundation-model)
- [[keetha-2025-mapanything]] — universal feed-forward 3D model; 12+ task configs + metric scale + multi-modal inputs (3d-reconstruction, metric-scale, multi-modal)
- [[lin-2025-depth-anything-3]] — minimalist single-transformer depth+ray prediction; +35.7%/+23.6% over VGGT (depth-estimation, 3d-reconstruction)
- [[wang-2025-cut3r]] — stateful recurrent online 3D reconstruction + virtual-view querying; handles dynamic scenes (3d-reconstruction, online, recurrent)
- [[karhade-2025-any4d]] — feed-forward metric 4D (geometry + scene flow + pose) with multi-modal inputs; 15× faster than next-best (4d-reconstruction, multi-modal, robotics)
- [[sucar-2026-v-dpm]] — VGGT fine-tuned to dynamic 4D via Dynamic Point Maps; halves error vs prior 4D methods (4d-reconstruction, dynamic-point-maps)
- [[luo-2026-4rc]] — encode-once query-anywhere/anytime 4D reconstruction; base geometry + relative motion (4d-reconstruction, conditional-query)
- [[anon-2026-trace-anything]] — Trajectory Fields: dense per-pixel parametric 3D trajectories in one feed-forward pass (4d-reconstruction, trajectory-fields, dense-tracking)
- [[anon-2026-point4d]] — long-range (200+ frame) 4D via 3D-query decoder + chunk chaining; first feed-forward 4D method that scales past 100 frames (4d-reconstruction, long-sequence, trajectory-chaining)
- [[zhang-2025-d4rt]] — canonical 2D-pixel-query 4D decoder; one model + 6 tasks; SOTA on TAPVid-3D + Sintel depth/pose; 9× faster than VGGT (4d-reconstruction, feed-forward, query-based)

### Long-context 3D reconstruction (linear-time / streaming)
- [[sun-2024-ttt]] — UCSD+Stanford+CMU; foundational TTT paper; hidden state = a parametric model `f_W` trained by self-supervised gradient descent on the input stream; linear-time, Transformer-quality long-context; the substrate the 3D-recon TTT papers below port from (sequence-modeling, rnn, test-time-training, fast-weights, long-context)
- [[elflein-2026-vgg-t3]] — NVIDIA; replaces VGGT's variable-length KV with fixed-size MLP via TTT; `O(n)` global/offline/unordered; 11.6× faster at 1k images; feed-forward visual localization for free (3d-reconstruction, test-time-training, linear-complexity, scalability)
- [[zhang-2026-loger]] — DeepMind+Berkeley; chunked feed-forward 3D with SWA + TTT hybrid memory; trained on 128 frames, generalizes to 19k; 74% ATE reduction over TTT3R on KITTI (3d-reconstruction, long-context, hybrid-memory, chunk-wise)

## Concepts
*Abstract ideas synthesized across sources.*

### Point tracking
- [[point-tracking]] — TAP problem family: predict per-point (x,y)+visibility on every frame
- [[joint-point-tracking]] — design choice: tracks exchange info vs. independent estimation
- [[3d-point-tracking]] — recover XYZ trajectories, not just xy; modular vs end-to-end design philosophies
- [[online-vs-offline-tracking]] — frame / window / video latency framing; the gap is closing in 2025-26
- [[synthetic-to-real-gap]] — Kubric-trained transfer to real; two camps (pseudo-label-real vs. better-synthetic)
- [[pseudo-labeling-point-tracking]] — heavy (BootsTAP, 15M+EMA) vs. light (CoTracker3, 15K+random-teacher) recipes

### 3D / 4D reconstruction
- [[feed-forward-3d-reconstruction]] — neural-network 3D in one pass; DUSt3R → VGGT lineage and the 2025-26 wave
- [[4d-reconstruction]] — dynamic 3D + time; current feed-forward 4D wave (V-DPM, Any4D, 4RC, Trace Anything)
- [[pointmap-representation]] — DUSt3R's per-pixel-3D-point representation; foundation of the sub-field
- [[dynamic-point-maps]] — pointmaps parameterized by (viewpoint, time); generalize shape + scene flow + camera
- [[trajectory-fields]] — per-pixel parametric splines as the 4D primitive (TraceAnything)
- [[trajectory-chaining]] — chunk-based extension to long sequences; 2D-pixel vs 3D-coordinate queries determines whether occlusion breaks chaining
- [[test-time-training]] — fast-weight associative memory as a `O(n)` substitute for softmax attention; VGG-T3 (offline-global), LoGeR (streaming-chunked), TTT3R (per-frame) cover the design space

### Streaming perception, real-time control & cross-domain patterns
- [[streaming-perception]] — Li/Ramanan ECCV 2020 framing; ancestor of online-vs-offline-tracking
- [[streaming-accuracy]] — the formal metric: zero-order-hold output stream vs continuous GT
- [[action-chunking]] — robot policy paradigm: predict H actions per inference; chunk boundaries cause mode jumps; origin ACT, refined through DP / RTC / Training-Time RTC / πR²
- [[asynchronous-control]] — closed-loop control under inference delay; control-side analogue of streaming perception
- [[diffusion-forcing]] — per-token independent noise levels; bridges next-token + full-sequence diffusion; sampling grid is the deployment-time DoF; now has three flavors (CDF causal RNN / DFoT non-causal DiT / πR² causal head)
- [[history-guidance]] — CFG generalization enabled by DFoT; HG-v / HG-t / HG-f / HG-tf compose scores conditioned on history at different noise levels; resolves CFG's static-video failure (video-diffusion, classifier-free-guidance)
- [[fast-slow-policy]] — split policy into slow VLM channel + fast proprio channel; πR² is the first single-model instance
- [[vla]] — vision-language-action models; the large-policy class RTC targets
- [[flow-matching]] — generative-model training objective used by π0 / π0.5 VLAs
- [[pi-gdm]] — pseudoinverse-guided diffusion (Song et al. 2023); inference-time guidance engine of RTC; linearization+pseudoinverse approximations degrade as constrained-dim count grows
- [[train-inference-mismatch]] — recurring pattern: SpaTrackerV2 / Point4D / RTC / Training-Time RTC / πR² / Diffusion Forcing / streaming-perception all instances; structural vs heuristic; fixes migrate inference → training → joint

## Methods
*Concrete algorithms, models, pipelines.*

### Point tracking
- [[tapnext]] — TRecViT (SSM+ViT) + masked trajectory tokens + classification coord head; causal per-frame TAP
- [[tapnext-plus-plus]] — TAPNext arch + Kubric-1024 + parallel SSM scan + roll aug
- [[cotracker]] — sliding-window transformer with cross-track attention + proxy tokens + unrolled training
- [[cotracker3]] — simplified arch + random-teacher distillation; online + offline from one recipe
- [[track-on2]] — causal frame-by-frame tracker with single expandable memory + classification-first head
- [[tapip3d]] — 3D tracking via depth+pose lifting → world-coord feature cloud → 3D N2N attention
- [[spatialtracker-v2]] — end-to-end joint depth + pose + 3D tracking with SyncFormer + differentiable BA

### 3D / 4D reconstruction
- [[dust3r]] — pairwise feed-forward 3D + pointmap representation; ancestor of the sub-field
- [[vggt]] — foundational multi-view transformer; alternating attention; over-complete heads
- [[mapanything]] — universal feed-forward 3D; factored repr; multi-modal inputs; metric scale
- [[depth-anything-3]] — minimalist single transformer + depth+ray output; teacher-student
- [[cut3r]] — recurrent online 3D + virtual-view query
- [[v-dpm]] — fine-tunes VGGT for dynamic 4D via DPM heads
- [[any4d]] — feed-forward dense metric 4D + multi-modal sensors
- [[4rc]] — encode-once, query-anywhere/anytime decoder
- [[trace-anything]] — feed-forward trajectory fields (per-pixel splines)
- [[vgg-t3]] — VGGT + TTT-MLP global-attention replacement; `O(n)` offline/unordered; queryable scene → visual localization
- [[loger]] — chunked feed-forward with hybrid memory (SWA + TTT); π³ backbone; 128 → 19k frame generalization

### Streaming perception & robot control
- [[streamer-meta-detector]] — detector + association + Kalman forecaster + dynamic scheduler; turns any detector streaming
- [[act]] — Action Chunking with Transformers; CVAE-style chunked policy + temporal ensembling; origin of chunking
- [[diffusion-policy]] — conditional DDPM over action chunks; chunked DiT denoiser; standard action head in modern VLAs
- [[diffusion-forcing]] — per-position noise schedule paradigm; CDF (causal variant); 2D sampling grid as inference-time DoF
- [[dfot]] — Diffusion Forcing Transformer; non-causal video DiT instance of DF; per-frame `k_t ∈ [0,1]`; enables History Guidance and 862-frame rollouts
- [[rtc]] — real-time chunking via inference-time inpainting; ΠGDM guidance + soft masking
- [[training-time-rtc]] — drop-in RTC replacement that moves prefix conditioning to training; zero inference overhead
- [[pi-r-squared]] — staircase diffusion-forcing schedule + slow/fast channel split; 1 NFE per call; per-tick reactivity on a slow VLA
- [[point4d]] — long-range 4D via 3D-coordinate queries + Sim(3) chunk chaining; extends D4RT to 3D queries
- [[d4rt]] — 2D-pixel-query 4D decoder (SRT lineage); encode-once, query-anywhere; ancestor of Point4D and 4RC

## Entities

### People
- [[andrea-vedaldi]] — Oxford VGG senior; **4 sources** (most recurring senior in wiki)
- [[carl-doersch]] — DeepMind; TAP-Vid → TAPIR → BootsTAP → TAPNext → TAPNext++ trunk
- [[christian-rupprecht]] — Oxford VGG senior; 3 sources (CoTracker series + VGGT)
- [[nikita-karaev]] — Meta AI / Oxford VGG; CoTracker, CoTracker3, SpatialTrackerV2
- [[jianyuan-wang]] — Oxford VGG + Meta AI; VGGT first author + SpatialTrackerV2 co-author
- [[artem-zholus]] — Mila / DeepMind (intern); co-first on TAPNext + TAPNext++
- [[nikhil-keetha]] — **CMU** + Meta Reality Labs; MapAnything + Any4D
- [[deva-ramanan]] — **CMU** faculty; senior author MapAnything + Any4D (user's school)
- [[sebastian-scherer]] — **CMU** AirLab faculty; senior author MapAnything + Any4D
- [[jay-karhade]] — **CMU**; Any4D lead
- [[katerina-fragkiadaki]] — **CMU** faculty; senior author TAPIP3D
- [[shubham-tulsiani]] — **CMU** faculty; **user's advisor**; visual geometry / scene flow line (Flow3R)
- [[adam-w-harley]] — Stanford; PIPs originator; co-author TAPIP3D
- [[gorkay-aydemir]] — Koç University; Track-On / Track-On2 lead
- [[qianqian-wang]] — UC Berkeley + DeepMind; CUT3R lead
- [[bingyi-kang]] — ByteDance Seed; DA3 lead + SpatialTrackerV2 co-author
- [[mehdi-sajjadi]] — DeepMind; D4RT lead + TAPNext co-author; SRT line
- [[skanda-koppula]] — DeepMind; TAPVid-3D introducer + TAPNext + D4RT co-author
- [[ignacio-rocco]] — DeepMind (prev. Meta AI); CoTracker + TAPNext + D4RT co-author
- [[mengtian-li]] — CMU / Ramanan group; streaming-perception ECCV 2020
- [[yu-xiong-wang]] — UIUC; streaming-perception co-author
- [[kevin-black]] — Physical Intelligence / Berkeley; RTC + Training-Time RTC lead; π0/π0.5/π0.6 collaborator
- [[sergey-levine]] — UC Berkeley + Physical Intelligence co-founder; senior author on ACT + RTC + Training-Time RTC (3 sources)
- [[chelsea-finn]] — Stanford IRIS; ACT senior co-author; cited throughout the IL / VLA line
- [[russ-tedrake]] — MIT CSAIL + TRI; co-author on Diffusion Policy + Diffusion Forcing + DFoT (3 sources)
- [[boyuan-chen]] — MIT CSAIL PhD; lead on Diffusion Forcing (NeurIPS 2024) + co-lead on DFoT (ICML 2025); the diffusion-forcing trajectory's anchor (2 sources)
- [[junyi-zhang]] — UC Berkeley + Google DeepMind; LoGeR lead; also MonST3R author (cited 5+ times across wiki)

### Organizations
- [[google-deepmind]] — TAP-line continuity (TAP-Vid → TAPNext++)
- [[meta-ai]] — CoTracker series + VGGT (joint with Oxford VGG)
- [[oxford-vgg]] — **6 sources**, most prolific org; CoTracker, VGGT, V-DPM lineage
- [[meta-reality-labs]] — MapAnything (AR/VR-focused, distinct from FAIR)
- [[cmu-ri]] — **user's institution**; TAPIP3D, MapAnything, Any4D (3 sources)
- [[uc-berkeley]] — CUT3R + RTC
- [[bytedance-seed]] — Depth Anything line; SpatialTrackerV2
- [[physical-intelligence]] — Sergey Levine's robotics foundation-model lab; π0 / π0.5 / π0.6 / RTC / Training-Time RTC
- [[mit-csail]] — Diffusion Policy (Tedrake/Du) + Diffusion Forcing (Chen/Sitzmann/Tedrake)
- [[uiuc]] — Yu-Xiong Wang group; streaming-perception co-author
- [[nvidia]] — VGG-T3 (with Vector Institute + U. Toronto)

### Tools
_(empty — defer foundation-model tools until 2nd source promotes them: DINOv3, DepthAnything 1/2, MegaSaM, MoGe, RecurrentGemma, TRecViT, MonST3R (already cited 5+ times — promote next batch), MASt3R as standalone tools. Pending: SRT (Sajjadi 2022), π³ (Pi3 — VGGT successor; **now cited by LoGeR**), Flow4R, DELTA, St4RTrack, InfiniteVGGT, VGGT-Long, StreamingVGGT, TTT3R (cited by both VGG-T3 and LoGeR — promote next batch), Kauldron training framework.)_

### Venues
_(empty — defer until enough source pages cluster by venue)_

### Datasets
- [[tap-vid-dataset]] — the TAP benchmark (Kubric train + DAVIS/Kinetics eval)
- [[tapvid-3d-dataset]] — 3D extension (Aria + DriveTrack + PStudio subsets)
- [[kubric-dataset]] — synthetic video generator; training-data foundation
- [[pointodyssey-dataset]] — long-form (~2400-frame) benchmark
- [[dynamic-replica-dataset]] — synthetic indoor scenes with RGB-D
- [[argoverse-hd-dataset]] — re-annotated Argoverse 1.1 at 30 FPS; first streaming-perception benchmark

## Questions & theses
*Open research questions tracked over time.*

- [[q-emergent-tracking-heuristics]] — do classical heuristics (cost volume, motion cluster) emerge from end-to-end training?
- [[q-sim2real-for-point-tracking]] — is real-data fine-tuning necessary, or is better synthetic training enough?
- [[q-tracking-vs-4d-reconstruction]] — are sparse point tracking and dense 4D reconstruction the same problem under different names?
- [[q-perception-control-symmetry]] — is the πR² staircase shape right for online 3D point tracking, or does perception prefer a bidirectional / no-tail variant?
- [[q-tt-rtc-vs-rtc-tradeoff]] — when does ΠGDM-based RTC remain preferable to training-time RTC? (untrainable checkpoints, varying constraints, multi-tenant infra)
- [[q-fast-slow-perception]] — does πR²'s slow/fast split (cached VLM + fresh proprio) transfer to perception as cached scene-feat + fresh image patches?

## Comparisons
*Cross-source syntheses and tables.*

- [[cmp-tap-methods]] — comparison of TAP methods (online/window/video, 2D/3D, training)
- [[cmp-3d-4d-reconstruction]] — comparison of feed-forward 3D/4D reconstruction methods (VGGT family + DA3 + CUT3R + 4D wave)

---

## Tag set
*Canonical tag list. Check before inventing a new tag.*

### Point tracking
- `point-tracking`, `3d-point-tracking`, `video-understanding`, `optical-flow`
- `online-tracking`, `offline-tracking`, `joint-tracking`, `online`
- `state-space-model`, `transformer`, `vit`, `dinov3`
- `memory`, `classification-first`, `masked-decoding`
- `pseudo-labeling`, `semi-supervised`, `synthetic-data`, `synthetic-training`, `sim2real`
- `long-sequence`, `re-detection`, `augmentation`, `dense-tracking`

### 3D / 4D reconstruction
- `3d-reconstruction`, `4d-reconstruction`, `video-reconstruction`
- `feed-forward`, `recurrent`, `online`
- `pointmap`, `dynamic-point-maps`, `trajectory-fields`, `scene-flow`
- `metric-scale`, `multi-modal`, `multi-view`
- `monocular`, `depth-estimation`, `camera-pose`, `bundle-adjustment`
- `sfm`, `slam`, `mvs`, `world-coordinates`, `virtual-view-query`
- `dust3r`, `vggt-fine-tune`, `conditional-query`, `teacher-student`
- `test-time-training`, `fast-weights`, `kv-compression`, `linear-complexity`, `hybrid-memory`, `sliding-window-attention`, `long-context`, `chunk-wise`

### Cross-cutting
- `transformer`, `attention`, `foundation-model`
- `benchmark`, `training-data`
- `robotics`, `embodied-ai`, `aerial-robotics`, `slam`, `perception`
- `ar`, `vr`, `latency`, `end-to-end`
- `interpretability`, `emergent-behavior`, `convergence`
- `computer-vision`, `self-supervised`, `representation`
- `pairwise`, `window-based`
- `meta`

### Robotics policy
- `vla`, `manipulation`, `imitation-learning`, `bimanual`, `aloha`
- `action-chunking`, `flow-matching`, `diffusion`, `diffusion-policy`, `diffusion-forcing`
- `asynchronous-control`, `fast-slow-policy`, `latency-adaptive`
- `train-inference-mismatch`, `training-recipe`, `training-paradigm`
- `inference-time-algorithm`, `inpainting`, `multimodal-distribution`
