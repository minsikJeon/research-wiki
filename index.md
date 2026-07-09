# Index

Content-oriented catalog of the wiki. Updated on every ingest. The agent reads
this first when answering queries.

See [[AGENTS]] for schema, [[log]] for chronology, [[overview]] for the
evolving thesis.

---

## Sources
*One entry per ingested raw item. Format: `[[slug]] — one-line takeaway (tags)`*

### Point Tracking (TAP)
- [[harley-2022-pips]] — Particle Video Revisited (ECCV 2022); the foundational deep TAP method; 8-frame MLP-Mixer iterative refinement over correlation pyramids; trained on FlyingThings++ (point-tracking, optical-flow, foundation-paper)
- [[doersch-2023-tapir]] — TAPIR (ICCV 2023); fuses TAP-Net global init with PIPs iterative refinement; depthwise conv (any-length) + self-supervised position uncertainty; canonical TAP method of 2023 (point-tracking, two-stage, uncertainty-estimation, foundation-paper)
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
- [[song-2026-gvs]] — MIT CSAIL + Runway ML ICLR 2026; Generative View Stitching turns any DFoT model into a training-free long-horizon camera-guided video generator via Omni Guidance + cyclic conditioning; handles loops, collisions, Impossible Staircase (video-diffusion, diffusion-stitching, training-free, omni-guidance)
- [[black-2025-rtc]] — Physical Intelligence; inference-time inpainting for VLA action chunks; freezes committed prefix + soft-masks the rest; robust to >300ms latency (robotics, vla, flow-matching, action-chunking)
- [[black-2025-training-time-rtc]] — Physical Intelligence; drop-in replacement for RTC that moves prefix conditioning to training; zero inference overhead; wins for d ≥ 2 (robotics, vla, training-recipe)
- [[anon-2026-pi-r-squared]] — CoRL 2026 (anon); πR² adds diffusion-forcing staircase schedule + slow/fast channel split; closed-loop 25 Hz on a 7 Hz VLA; +50% relative gain on reactivity-sensitive real-world tasks (robotics, vla, fast-slow-policy, latency-adaptive)

### 3D / 4D Reconstruction (feed-forward foundation models)
- [[wang-2025-vggt]] — foundational feed-forward multi-view 3D transformer; predicts cameras + depth + pointmaps + tracks in one pass (3d-reconstruction, foundation-model)
- [[keetha-2025-mapanything]] — universal feed-forward 3D model; 12+ task configs + metric scale + multi-modal inputs (3d-reconstruction, metric-scale, multi-modal)
- [[lin-2025-depth-anything-3]] — minimalist single-transformer depth+ray prediction; +35.7%/+23.6% over VGGT (depth-estimation, 3d-reconstruction)
- [[wang-2025-cut3r]] — stateful recurrent online 3D reconstruction + virtual-view querying; handles dynamic scenes (3d-reconstruction, online, recurrent)
- [[wu-2025-point3r]] — Tsinghua NeurIPS 2025; streaming DUSt3R-family recon with **explicit spatial pointer memory** (each unit = 3D position + feature); 3D hierarchical RoPE; distance-based fusion; beats CUT3R on long sequences (500-1000 frames, Acc 0.071 vs 0.238) (3d-reconstruction, streaming, spatial-memory, 3d-rope, dust3r-family)
- [[karhade-2025-any4d]] — feed-forward metric 4D (geometry + scene flow + pose) with multi-modal inputs; 15× faster than next-best (4d-reconstruction, multi-modal, robotics)
- [[sucar-2026-v-dpm]] — VGGT fine-tuned to dynamic 4D via Dynamic Point Maps; halves error vs prior 4D methods (4d-reconstruction, dynamic-point-maps)
- [[luo-2026-4rc]] — encode-once query-anywhere/anytime 4D reconstruction; base geometry + relative motion (4d-reconstruction, conditional-query)
- [[anon-2026-trace-anything]] — Trajectory Fields: dense per-pixel parametric 3D trajectories in one feed-forward pass (4d-reconstruction, trajectory-fields, dense-tracking)
- [[anon-2026-point4d]] — long-range (200+ frame) 4D via 3D-query decoder + chunk chaining; first feed-forward 4D method that scales past 100 frames (4d-reconstruction, long-sequence, trajectory-chaining)
- [[zhang-2025-d4rt]] — canonical 2D-pixel-query 4D decoder; one model + 6 tasks; SOTA on TAPVid-3D + Sintel depth/pose; 9× faster than VGGT (4d-reconstruction, feed-forward, query-based)
- [[anon-2026-stride]] — NeurIPS 2026 anon; first feed-forward 4D driving-scene method that fuses camera+LiDAR via PTv3 *and* learns instance decomposition without annotations (4d-reconstruction, driving-scenes, multi-modal, lidar, instance-decomposition)

### Manipulation via point tracks (manipulation-from-tracks)
- [[bharadhwaj-2024-track2act]] — Track2Act (ECCV 2024); CMU + Meta FAIR; DiT denoises future 2D point tracks from web videos → PnP rigid transforms → end-effector + residual policy (~400 Spot demos); the founding "predict tracks → drive manipulation" paper (manipulation, point-tracking, web-video, residual-policy, foundation-paper, imitation-learning)
- [[xu-2024-im2flow2act]] — Im2Flow2Act (CoRL 2024); Stanford+Columbia+JPM; object-only flow as cross-domain interface; flow generation on human videos + flow-conditioned diffusion policy trained on sim play; 81% real-world SR with zero real-robot data (manipulation, flow, sim-to-real, cross-embodiment, diffusion-policy)
- [[zhi-2025-3dflowaction]] — 3DFlowAction (Jun 2025); SCUT+Tencent+HKUST; lifts Im2Flow2Act to 3D flow (u,v,z,vis); GPT-4o closed-loop verifier; ManiFlow-110k cross-source dataset (110K); optimization action policy needing no action labels; 70% vs 20-50% baselines (manipulation, 3d-point-tracking, video-diffusion, world-model, vlm-verifier)
- [[kuang-2026-dex4d]] — Dex4D (Feb 2026); **CMU** Fragkiadaki×Tulsiani; Anypose-to-Anypose sim-to-real RL + Paired Point Encoding; first to apply point-tracks-as-interface to 22-DoF dexterous manipulation; Wan2.6 video gen → CoTracker3 → 3D tracks → policy; +22.5% SR over NovaFlow-CL (manipulation, dexterous, sim-to-real, reinforcement-learning, world-model)
- [[kim-2026-pri4r]] — Pri4R (Mar 2026); KAIST+LG+SNU+Yonsei+**CMU** (Jeni); 3D point tracks as **privileged supervision** for π₀ / π₀.₅ / OpenVLA-OFT; auxiliary head discarded at inference; +13.2% RoboCasa for OpenVLA-OFT, +9.8% LIBERO-Long; ablation: 3D tracks beat 2D / depth / goal-only supervision (vla, manipulation, privileged-information, auxiliary-supervision)
- [[lee-2026-mu0]] — µ0 (Jun 2026); UMD (F. Huang + J.-B. Huang) + SNU (H. J. Kim); **query-conditioned 3D trace-space world model**; DINOv2 semantic keypoint sampling + globally aligned 3D + event-centric captioning (TraceExtract, ~8× TraceGen scale); SmolVLM2 backbone + permutation-equivariant Trace Expert on B-spline control points + flow matching (+validity +semantic rigidity); frozen µ0 features feed action expert; beats π₀ on RoboCasa365 sim (30.25 vs 25.25%); 91.7% real-world UR3 avg vs 80% π₀.₅ / 81.7% TraceGen (manipulation, world-model, 3d-point-tracking, flow-matching, b-spline, semantic-keypoint)
- [[huang-2026-pointworld]] — PointWorld (Jan 2026); Stanford (W. Huang + Fei-Fei) + NVIDIA (Chao + Mousavian + Liu + Fox + Mo); **action-conditioned 3D dynamics WM** — predicts full-scene 3D point flows from RGB-D + robot action point flows (URDF forward kinematics); PTv3 backbone (50M–1B params) + frozen DINOv3 features; movement-weighted + aleatoric-uncertainty + Huber loss; chunked 10-step @ 0.1s; **~2M trajectories** DROID+BEHAVIOR-1K with custom FoundationStereo+VGGT+CoTracker3 annotation pipeline; **published log-linear scaling laws**; deployed via MPPI for zero-shot rigid pushing / deformable / articulated / tool use on real Franka from single RGB-D (manipulation, world-model, 3d-point-tracking, dense-tracking, point-transformer, dinov3, mpc, foundation-model)

### Humanoid whole-body control from video
- [[allshire-2025-videomimic]] — VideoMimic (CoRL 2025 Best Student Paper); UC Berkeley (Allshire × Choi × Zhang × McAllister equal + Darrell + Abbeel + Malik + Kanazawa senior); **real-to-sim-to-real** pipeline turning monocular smartphone videos into contextual humanoid skills; joint 4D human-scene reconstruction (SMPL + MegaSAM/MonST3R + JAX-Levenberg-Marquardt joint optimization with metric SMPL height prior); retarget to Unitree G1; 4-stage RL curriculum (MoCap-pretrain → scene-conditioned DeepMimic tracking → DAgger distill → under-conditioned PPO fine-tune); single distilled policy conditioned only on proprio + 11×11 heightmap + root direction; onboard 50 Hz on 23-DoF Unitree G1; stair ascent/descent + chair sit-stand + kerbs + rough terrain from context alone; 123 smartphone videos as full training corpus (humanoid, whole-body-control, real-to-sim-to-real, imitation-learning, reinforcement-learning, smpl, 4d-reconstruction, deepmimic)

### Long-context 3D reconstruction (linear-time / streaming)
- [[sun-2024-ttt]] — UCSD+Stanford+CMU; foundational TTT paper; hidden state = a parametric model `f_W` trained by self-supervised gradient descent on the input stream; linear-time, Transformer-quality long-context; the substrate the 3D-recon TTT papers below port from (sequence-modeling, rnn, test-time-training, fast-weights, long-context)
- [[elflein-2026-vgg-t3]] — NVIDIA; replaces VGGT's variable-length KV with fixed-size MLP via TTT; `O(n)` global/offline/unordered; 11.6× faster at 1k images; feed-forward visual localization for free (3d-reconstruction, test-time-training, linear-complexity, scalability)
- [[zhang-2026-loger]] — DeepMind+Berkeley; chunked feed-forward 3D with SWA + TTT hybrid memory; trained on 128 frames, generalizes to 19k; 74% ATE reduction over TTT3R on KITTI (3d-reconstruction, long-context, hybrid-memory, chunk-wise)
- [[zhang-2025-lact]] — MIT+Adobe; Large-Chunk TTT (2K–1M tokens) lifts TTT hardware utilization from 5% → 70%, enables nonlinear SwiGLU fast weights @ 40% of model params + Muon optimizer; 14B AR video diffusion @ 56K tokens; 1M-token NVS contexts (test-time-training, long-context, video-diffusion, scalability)
- [[zhuo-2026-stream-vggt]] — Tsinghua; StreamVGGT = VGGT with global attention → temporal causal + KV-cached memory tokens, distilled from VGGT; outperforms CUT3R on reconstruction / depth / camera pose; 5× faster current-frame inference at 40 frames (3d-reconstruction, streaming, causal-attention, kv-cache, distillation)
- [[ma-2026-fsm]] — MIT-IBM/UMich/UMass; LaCET = LaCT + EWC elastic consolidation; FSM = first TTT-based 4D NVS model; SOTA feed-forward 4D on Stereo4D (32.16 PSNR) (test-time-training, elastic-weight-consolidation, 4d-reconstruction, novel-view-synthesis)

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

### Manipulation interfaces
- [[point-tracks-as-manipulation-interface]] — point tracks (2D/3D) as the intermediate representation between cross-embodiment planning and robot execution; the binding concept of Track2Act / Im2Flow2Act / 3DFlowAction / Dex4D / Pri4R / µ0 / PointWorld

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
- [[pips]] — Particle Video Revisited; CNN + multi-scale corr pyramids + 12-block MLP-Mixer iterative refinement over 8 frames + visibility-thresholded chaining; the founding deep TAP architecture
- [[tapir]] — TAP-Net global per-frame init + PIPs-style local-pyramid refinement; 12-block depthwise-conv-over-time (any-length) + self-supervised position uncertainty
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
- [[point3r]] — streaming DUSt3R-family recon with explicit 3D-indexed pointer memory + 3D hierarchical RoPE; beats CUT3R at long sequences
- [[streamvggt]] — causal-attention + KV-cached restructuring of VGGT; distilled from VGGT teacher; streaming alternative to CUT3R that *beats* it on reconstruction + camera pose
- [[v-dpm]] — fine-tunes VGGT for dynamic 4D via DPM heads
- [[any4d]] — feed-forward dense metric 4D + multi-modal sensors
- [[4rc]] — encode-once, query-anywhere/anytime decoder
- [[trace-anything]] — feed-forward trajectory fields (per-pixel splines)
- [[stride]] — feed-forward 4D for driving scenes; camera+LiDAR fused in 3D via PTv3; outputs 3DGS + per-Gaussian velocity + learnable instance decomposition
- [[vgg-t3]] — VGGT + TTT-MLP global-attention replacement; `O(n)` offline/unordered; queryable scene → visual localization
- [[loger]] — chunked feed-forward with hybrid memory (SWA + TTT); π³ backbone; 128 → 19k frame generalization
- [[lact]] — Large-Chunk TTT block (window-attn + SwiGLU MLP fast weights + Muon test-time optimizer); 2K–1M tokens per chunk; 70% GPU util in pure PyTorch; NVS, LM, and AR video diffusion
- [[lacet]] — LaCT + Fisher-weighted elastic consolidation (EWC); streaming-EMA anchors; fixes multi-chunk drift
- [[fsm]] — Fast Spatial Memory; LaCET-based 4D NVS model with LVSM-style and LRM-style decoders; SOTA feed-forward 4D

### Manipulation via point tracks
- [[track2act]] — DiT denoiser of full 2D point trajectories from web videos + PnP rigid-transform fitting + residual BC policy on Spot demos; the founding method
- [[im2flow2act]] — object-only flow generator (AnimateDiff) on human videos + flow-conditioned diffusion policy trained on sim play; temporal alignment module; 81% real-world success rate with zero real-robot data
- [[3d-flow-action]] — 3D flow (u,v,z,vis) world model + GPT-4o closed-loop verifier + IK-aware grasp + optimization-based action policy (no action labels); 70% on rotation-heavy tasks
- [[dex4d]] — Anypose-to-Anypose sim-to-real RL on dexterous hand + Paired Point Encoding; first dexterous application; Wan2.6 video gen → CoTracker3 → 3D point tracks → policy
- [[pri4r]] — auxiliary 3D point-track head on VLAs (OpenVLA-OFT / π₀ / π₀.₅); privileged supervision; head discarded at inference; +13.2% RoboCasa
- [[mu0]] — query-conditioned 3D trace-space **world model** (SmolVLM2 + Trace Expert); B-spline control points + flow matching; TraceExtract data engine (semantic keypoints + global 3D + event captions); frozen µ0 features feed action expert; beats action-labeled π₀ with zero action pretraining
- [[pointworld]] — action-conditioned **3D dynamics world model** (learned physics simulator); PTv3 + DINOv3 + URDF-derived robot point flows; chunked 10-step prediction @ 0.1s; deployed via MPPI for zero-shot in-the-wild Franka manipulation across rigid / deformable / articulated / tool-use classes; published scaling laws (50M → 1B params, 5% → 100% data, log-linear)

### Humanoid whole-body from video
- [[videomimic]] — real-to-sim-to-real; monocular RGB video → joint 4D human-scene reconstruction (SMPL + MegaSAM + JAX joint-opt) → G1 retarget → 4-stage RL curriculum (MPT → scene-cond tracking → DAgger → PPO fine-tune) → single distilled policy on proprio + 11×11 heightmap + root direction; 50 Hz onboard Unitree G1; single policy handles stair up/down, chair sit-stand, rough terrain

### Streaming perception & robot control
- [[streamer-meta-detector]] — detector + association + Kalman forecaster + dynamic scheduler; turns any detector streaming
- [[act]] — Action Chunking with Transformers; CVAE-style chunked policy + temporal ensembling; origin of chunking
- [[diffusion-policy]] — conditional DDPM over action chunks; chunked DiT denoiser; standard action head in modern VLAs
- [[diffusion-forcing]] — per-position noise schedule paradigm; CDF (causal variant); 2D sampling grid as inference-time DoF
- [[dfot]] — Diffusion Forcing Transformer; non-causal video DiT instance of DF; per-frame `k_t ∈ [0,1]`; enables History Guidance and 862-frame rollouts
- [[gvs]] — Generative View Stitching; training-free, off-the-shelf DFoT → long-horizon camera-guided video via Omni Guidance + cyclic conditioning
- [[rtc]] — real-time chunking via inference-time inpainting; ΠGDM guidance + soft masking
- [[training-time-rtc]] — drop-in RTC replacement that moves prefix conditioning to training; zero inference overhead
- [[pi-r-squared]] — staircase diffusion-forcing schedule + slow/fast channel split; 1 NFE per call; per-tick reactivity on a slow VLA
- [[point4d]] — long-range 4D via 3D-coordinate queries + Sim(3) chunk chaining; extends D4RT to 3D queries
- [[d4rt]] — 2D-pixel-query 4D decoder (SRT lineage); encode-once, query-anywhere; ancestor of Point4D and 4RC

## Entities

### People
- [[andrea-vedaldi]] — Oxford VGG senior; **4 sources** (most recurring senior in wiki)
- [[carl-doersch]] — DeepMind; TAP-Vid → TAPIR → BootsTAP → TAPNext → TAPNext++ trunk; **3 sources**
- [[christian-rupprecht]] — Oxford VGG senior; 3 sources (CoTracker series + VGGT)
- [[nikita-karaev]] — Meta AI / Oxford VGG; CoTracker, CoTracker3, SpatialTrackerV2
- [[jianyuan-wang]] — Oxford VGG + Meta AI; VGGT first author + SpatialTrackerV2 co-author
- [[artem-zholus]] — Mila / DeepMind (intern); co-first on TAPNext + TAPNext++
- [[nikhil-keetha]] — **CMU** + Meta Reality Labs; MapAnything + Any4D
- [[deva-ramanan]] — **CMU** faculty; senior author MapAnything + Any4D (user's school)
- [[sebastian-scherer]] — **CMU** AirLab faculty; senior author MapAnything + Any4D
- [[jay-karhade]] — **CMU**; Any4D lead
- [[katerina-fragkiadaki]] — **CMU** faculty; senior author PIPs + TAPIP3D (the CMU TAP origin line) + co-senior Dex4D; **3 sources**
- [[shubham-tulsiani]] — **CMU** faculty; **user's advisor**; visual geometry / scene flow line + manipulation-from-tracks line; senior on Point4D / Track2Act / Dex4D; **3 sources**
- [[adam-w-harley]] — Stanford (prior CMU PhD with Fragkiadaki); **PIPs lead** + co-author TAPIP3D; **2 sources**
- [[homanga-bharadhwaj]] — **CMU** PhD (Tulsiani group); Track2Act first author; DemoDiffusion / Gen2Act collaborator
- [[yuxuan-kuang]] — **CMU** (visiting; Tulsiani × Fragkiadaki); Dex4D first author; StopNet / RAM / SkillBlender / FetchBot author
- [[sungjae-park]] — **CMU** (Tulsiani group); Dex4D co-first author; DemoDiffusion co-author
- [[shuran-song]] — Stanford+Columbia; Im2Flow2Act + Diffusion Policy senior
- [[cheng-chi]] — Stanford+Columbia; Diffusion Policy first author + Im2Flow2Act co-author
- [[laszlo-jeni]] — **CMU** faculty; Pri4R co-senior; first KAIST × LG × CMU collaboration in the wiki
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
- [[boyuan-chen]] — MIT CSAIL PhD; lead on Diffusion Forcing (NeurIPS 2024) + co-lead on DFoT (ICML 2025) + co-author on GVS (ICLR 2026); the diffusion-forcing trajectory's anchor (3 sources)
- [[chonghyuk-song]] — MIT CSAIL PhD; GVS lead (1 source). Distinct from Kiwhan Song (DFoT lead)
- [[junyi-zhang]] — UC Berkeley (Darrell) + Google DeepMind; LoGeR lead + VideoMimic co-first; also MonST3R author (cited 8+ times across wiki); **2 sources**, spans long-context 3D recon and humanoid-video threads

### Organizations
- [[google-deepmind]] — TAP-line continuity (TAP-Vid → TAPIR → BootsTAP → TAPNext → TAPNext++); **3 sources** + D4RT + CUT3R co-affiliation
- [[meta-ai]] — CoTracker series + VGGT + Track2Act (Mottaghi at FAIR; **4 sources**)
- [[oxford-vgg]] — **7 sources**, most prolific org; TAPIR (Zisserman), CoTracker, VGGT, V-DPM lineage
- [[meta-reality-labs]] — MapAnything (AR/VR-focused, distinct from FAIR)
- [[cmu-ri]] — **user's institution**; PIPs (Fragkiadaki, 2022), streaming-perception, TAPIP3D, MapAnything, Any4D, Point4D, **Track2Act**, **Dex4D**, **Pri4R** (**9 sources** — both the institutional origin of the modern deep TAP sub-field AND now its largest manipulation-from-tracks cluster)
- [[uc-berkeley]] — **5 sources**; CUT3R + RTC + Training-Time RTC + LoGeR + **VideoMimic** (CoRL 2025 Best Student Paper); spans feed-forward 3D + real-time control + long-context 3D + humanoid-from-video across BAIR / Efros / Kanazawa / Malik / Darrell / Levine labs
- [[bytedance-seed]] — Depth Anything line; SpatialTrackerV2
- [[physical-intelligence]] — Sergey Levine's robotics foundation-model lab; π0 / π0.5 / π0.6 / RTC / Training-Time RTC
- [[mit-csail]] — Diffusion Policy + Diffusion Forcing + DFoT + GVS (Chen/Song/Sitzmann/Tedrake cluster) + LaCT (Zhang/Yang/Freeman) — **5 sources, the de facto center of diffusion-forcing-for-video and TTT-for-long-context**
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
- [[cmp-point-track-manipulation]] — comparison of the 7 manipulation-via-point-tracks methods (Track2Act / Im2Flow2Act / 3DFlowAction / Dex4D / Pri4R / µ0 / PointWorld)

---

## Tag set
*Canonical tag list. Check before inventing a new tag.*

### Point tracking
- `point-tracking`, `3d-point-tracking`, `video-understanding`, `optical-flow`, `semantic-keypoint`, `b-spline`, `dense-tracking`, `point-transformer`
- `online-tracking`, `offline-tracking`, `joint-tracking`, `online`, `window-based`
- `state-space-model`, `transformer`, `vit`, `dinov3`, `mlp-mixer`, `depthwise-conv`
- `memory`, `classification-first`, `masked-decoding`, `iterative-refinement`, `two-stage`
- `occlusion`, `uncertainty-estimation`, `foundation-paper`
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
- `test-time-training`, `fast-weights`, `kv-compression`, `linear-complexity`, `hybrid-memory`, `sliding-window-attention`, `long-context`, `chunk-wise`, `elastic-weight-consolidation`, `novel-view-synthesis`, `4dgs`
- `driving-scenes`, `lidar`, `point-transformer`, `gaussian-splatting`, `instance-decomposition`, `self-supervised`

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
- `vla`, `manipulation`, `imitation-learning`, `bimanual`, `aloha`, `mpc`, `humanoid`, `whole-body-control`, `real-to-sim-to-real`, `smpl`, `deepmimic`, `unitree-g1`, `monocular-video`
- `action-chunking`, `flow-matching`, `diffusion`, `diffusion-policy`, `diffusion-forcing`
- `asynchronous-control`, `fast-slow-policy`, `latency-adaptive`
- `train-inference-mismatch`, `training-recipe`, `training-paradigm`
- `inference-time-algorithm`, `inpainting`, `multimodal-distribution`
- `dexterous`, `cross-embodiment`, `sim-to-real`, `reinforcement-learning`
- `web-video`, `world-model`, `video-diffusion`, `flow`, `residual-policy`
- `privileged-information`, `auxiliary-supervision`, `vlm-verifier`
- `paired-point-encoding`
