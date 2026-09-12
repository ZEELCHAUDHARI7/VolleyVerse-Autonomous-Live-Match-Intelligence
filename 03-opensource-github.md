# Open Source / GitHub Scan (STEP 4)

**Methodology:** GitHub web search was blocked (org egress, HTTP 403). Findings
come from the GitHub REST/Search API plus raw README/LICENSE fetches. Every repo
listed was actually returned by one of those calls. "Unverified" = could not
confirm that specific field this session.

**Skepticism check, confirmed empirically:** volleyball-specific searches
returned dozens of 0–1★ repos last touched once by a student and abandoned.
**Treat any volleyball CV repo under ~10★ as coursework unless proven otherwise.**

---

## A) Volleyball-specific

| Repo | What it does | Tech | Licence | Activity | Verdict |
|---|---|---|---|---|---|
| [shukkkur/VolleyVision](https://github.com/shukkkur/VolleyVision) | 3 stages: ball tracking, player+action recognition (block/defense/serve/set/spike), court segmentation | YOLOv7-tiny / YOLOv8m / Roboflow / DaSiamRPN / OpenCV | ⚠️ **DISCREPANCY** — README states **CC BY-NC-ND** (non-commercial, no derivatives); GitHub's licence detection reported **AGPL-3.0**. **Either reading blocks commercial reuse.** Verify before touching. | ~326★, last push 2025-06-20 | **STUDY ONLY** |
| [masouduut94/volleyball_analytics](https://github.com/masouduut94/volleyball_analytics) | End-to-end: **VideoMAE** classifies SERVICE/PLAY/NO-PLAY → **YOLOv8** detects 6 classes (ball, serve, receive, set, spike, block) → FastAPI storage | VideoMAE (HF), YOLOv8, FastAPI, OpenCV | **GPL-2.0** ✅ verified | ~122★, pushed **2026-09-10** | **STUDY CLOSELY, do not vendor.** Most actively maintained volleyball CV repo found; closest existing thing to VolleyVerse's goal. **Zero published accuracy numbers.** |
| [asigatchov/fast-volleyball-tracking-inference](https://github.com/asigatchov/fast-volleyball-tracking-inference) | Real-time volleyball ball detection/tracking; benchmarks 4 models | ONNX + OpenVINO, OpenCV | **MIT** ✅ verified | ~60★, pushed 2026-06-17 ✅ | **USE — the single most important repo in this scan** |
| [asigatchov/vball-net](https://github.com/asigatchov/vball-net) | VballNetV1/FastV1 — U-Net + depthwise-separable convs + motion attention, TrackNetV4 lineage | TF/Keras, ONNX, OpenVINO | MIT (per metadata) | 11★, 2026-06-08 | **USE/FORK** (training side) |
| [asigatchov/vball-net-pytorch](https://github.com/asigatchov/vball-net-pytorch) | PyTorch port, 768×432, 5-frame sequences, ONNX export | PyTorch | Unverified | 4★ | FORK/STUDY |
| [asigatchov/RAVEL-VB-beachvolleyball-tracking](https://github.com/asigatchov/RAVEL-VB-beachvolleyball-tracking) | Beach volleyball player+ball tracking; dual PyTorch(CUDA)/OpenVINO(CPU) | PyTorch, OpenVINO | Unverified | 14★, 2026-08-07 | STUDY — `asigatchov` is the most serious solo volleyball-CV dev on GitHub; **worth contacting** |
| [mostafa-saad/deep-activity-rec](https://github.com/mostafa-saad/deep-activity-rec) | Original CVPR-2016 code; introduced **The Volleyball Dataset** (4,830 frames, 8 group-activity classes) | Caffe/Theano-era dual-LSTM | **BSD-2-Clause** ✅ | 224★, old | STUDY ONLY (dataset provenance) |
| [openvolley/ovml](https://github.com/openvolley/ovml) · [ovcourt](https://github.com/openvolley/ovcourt) | R wrappers around YOLOv3/4/7 + pattern-based volleyball court detection | R + torch-for-R | Unverified | 31★ / 2★, ovscout2 pushed 2026-09-12 | STUDY ONLY — real long-running volleyball OSS community; CV layer is a thin, self-admittedly slow R wrapper. **Worth a data/domain partnership, not a code dependency.** |
| [GYdevy/Volleylitics](https://github.com/GYdevy/Volleylitics) | "Modular CV system for automated volleyball rally segmentation, ball tracking, tactical analytics" | Python | Unverified | 1★, 2026-08-10 | AVOID today; **re-check periodically** |
| [victorwctsang/volleyball-rally-segmentation](https://github.com/victorwctsang/volleyball-rally-segmentation) · [kairsato/Volleyball-Metrics](https://github.com/kairsato/Volleyball-Metrics) · [ZeynepNilayYazici/volleyball-detection-tracking](https://github.com/ZeynepNilayYazici/volleyball-detection-tracking) | Rally dead-time removal / court-calibrated stats+reels / YOLOv8+ByteTrack+jersey-OCR feasibility | Various | Unverified | 0–1★, all pushed within 2 weeks of this scan | AVOID as dependencies — **but their existence is a market signal**: several independent people are building VolleyVerse's MVP right now |
| [Cli98/volleyball-tracking-demo](https://github.com/Cli98/volleyball-tracking-demo) | Classification-net ball tracking; author calls the backbone "not the strongest" | — | Unverified | 16★ | **AVOID** |

### Benchmarked volleyball ball-tracking numbers (from `fast-volleyball-tracking-inference`, on 886 labelled 720p/1080p frames)

| Model | F1 | CPU FPS |
|---|---|---|
| VballNetV4c | **0.902** | 149.6 |
| VballNetGridV2b | 0.892 | 122.9 |
| VballNetGridV3 | 0.867 | 229.3 |
| VballNetFastV1 | 0.799 | **1140** |

A second table from the same author reports accuracy@5px: best **87.25% @
138.7 fps**, fastest **73.20% @ 271.9 fps** (CPU, Intel i5-10400F).

**Bottom line: CPU-only, sub-$1000-hardware, real-time volleyball ball
detection at ~F1 0.9 already exists, MIT-licensed.** This de-risks the ball
leg of the roadmap more than anything else found.

### Datasets
**Roboflow Universe** has ~300 volleyball-tagged community datasets. Largest
observed: "volleyball (RC2025)" 16,100 images / 1 class (ball); "volleyball
(sportrackr)" 7,430 images / 1 class (serve); "Volleyball (myarmy)" 2,860
images / 3 classes. Fragmented, no canonical benchmark. **Each project sets its
own licence** (commonly CC BY 4.0, not guaranteed) — check per-dataset before
commercial training.

---

## B) Ball tracking (the core hard problem)

| Repo | What | Licence | Activity | Verdict |
|---|---|---|---|---|
| [nttcom/WASB-SBDT](https://github.com/nttcom/WASB-SBDT) | "Widely Applicable Strong Baseline for Sports Ball Detection and Tracking" (BMVC 2023, NTT). README **explicitly lists volleyball** among covered sports | **MIT** ✅ | 190★, last push 2023-11-23 | **USE/STUDY.** Caveat: ships evaluation code for pretrained models, not the full training pipeline; ~3 yrs stale |
| [qaz812345/TrackNetV3](https://github.com/qaz812345/TrackNetV3) | Shuttlecock tracking + augmentation + trajectory rectification | **MIT** ✅ (explicitly commercial-friendly) | 300★, pushed **2026-09-03** | **FORK** — best-maintained heatmap tracker; architecturally what VballNet transplanted to volleyball |
| [alenzenx/TrackNetV3](https://github.com/alenzenx/TrackNetV3) | Attention-based TrackNet variant | MIT ✅ | 160★, 2025-06-02 | FORK/STUDY |
| [ChgygLin/TrackNetV2-pytorch](https://github.com/ChgygLin/TrackNetV2-pytorch) | PyTorch port of TF TrackNetV2 | Unverified | 63★ | STUDY ONLY |
| 3D ball reconstruction | Only hit was a cricket repo (0★) | — | — | **No usable volleyball/general 3D ball-reconstruction repo found.** Remains a DIY multi-camera triangulation problem |

---

## C) Player detection / tracking / re-ID

**Detectors**

| Repo | Licence | Activity | Verdict |
|---|---|---|---|
| [ultralytics/ultralytics](https://github.com/ultralytics/ultralytics) (YOLOv8/11/12) | **AGPL-3.0** ✅ | 61,535★, pushed 2026-09-12 | Dominant, but **SaaS use requires a paid Enterprise licence** — see landmines |
| [Megvii-BaseDetection/YOLOX](https://github.com/Megvii-BaseDetection/YOLOX) | **Apache-2.0** ✅ | 10,517★, 2025-06-08 | USE — mature Apache alternative |
| [lyuwenyu/RT-DETR](https://github.com/lyuwenyu/RT-DETR) | **Apache-2.0** ✅ | 5,517★, 2026-09-07 | **USE** — best actively-developed Apache YOLO alternative (CVPR 2024) |
| [open-mmlab/mmdetection](https://github.com/open-mmlab/mmdetection) | **Apache-2.0** ✅ | 32,920★ | USE/STUDY — includes RTMDet |

**Trackers**

| Repo | Licence | Activity | Verdict |
|---|---|---|---|
| [ifzhang/ByteTrack](https://github.com/ifzhang/ByteTrack) | **MIT** ✅ | high | USE (preferably via a wrapper) |
| [NirAharon/BoT-SORT](https://github.com/NirAharon/BoT-SORT) | **MIT** ✅ | 1,537★ | USE |
| [noahcao/OC_SORT](https://github.com/noahcao/OC_SORT) | **MIT** ✅ | 1,141★, 2026-04-21 | USE |
| [GerardMaggiolino/Deep-OC-SORT](https://github.com/GerardMaggiolino/Deep-OC-SORT) | **MIT** ✅ | 284★ | USE/STUDY |
| [mikel-brostrom/boxmot](https://github.com/mikel-brostrom/boxmot) | **AGPL-3.0** ✅ | 8,293★, 2026-09-12 | Most popular all-in-one MOT collection — **but AGPL** |
| [roboflow/trackers](https://github.com/roboflow/trackers) | **Apache-2.0** ✅ | 3,766★, 2026-09-11 | **USE** — the commercially-safe answer to boxmot |
| [tryolabs/norfair](https://github.com/tryolabs/norfair) | **BSD-3-Clause** ✅ | 2,681★ | USE — lightweight, detector-agnostic |
| [open-mmlab/mmtracking](https://github.com/open-mmlab/mmtracking) | Apache-2.0 ✅ | 3,892★, stale since 2023-09 | **AVOID** as a new dependency |

StrongSORT and BoostTrack have no well-starred standalone repos — reach them via
`roboflow/trackers` (Apache) rather than `boxmot` (AGPL).

**Re-ID / jersey numbers**

| Repo | Licence | Activity | Verdict |
|---|---|---|---|
| [mkoshkina/jersey-number-pipeline](https://github.com/mkoshkina/jersey-number-pipeline) | **CC BY-NC 3.0** ✅ — non-commercial only | 67★ | **STUDY ONLY** — cannot embed commercially |
| [SoccerNet/sn-jersey](https://github.com/SoccerNet/sn-jersey) | Unverified | 30★ | STUDY — challenge scaffold |
| [baudm/parseq](https://github.com/baudm/parseq) | **Apache-2.0** ✅ | 739★ | **USE** — the OCR backbone jersey pipelines build on, and it is commercially clean. Build your own crop→PARSeq pipeline |
| [shallowlearn/sportsreid](https://github.com/shallowlearn/sportsreid) | **MIT** ✅ | 26★ | USE/STUDY — #2 SoccerNet 2022 Re-ID |
| [SoccerNet/sn-reid](https://github.com/SoccerNet/sn-reid) | **MIT** ✅ | 90★ | STUDY/USE |
| [DeepSportradar/player-reidentification-challenge](https://github.com/DeepSportradar/player-reidentification-challenge) | Unverified | 53★ | STUDY (basketball, EPFL/Sportradar) |

---

## D) Pose

| Repo | Licence | Verdict |
|---|---|---|
| [open-mmlab/mmpose](https://github.com/open-mmlab/mmpose) | **Apache-2.0** ✅ | **USE** — RTMPose and RTMO ship as configs *inside* this repo |
| [ViTAE-Transformer/ViTPose](https://github.com/ViTAE-Transformer/ViTPose) | **Apache-2.0** ✅ | USE — highest accuracy, heavier |
| [JunkyByte/easy_ViTPose](https://github.com/JunkyByte/easy_ViTPose) | **Apache-2.0** ✅ | USE — deployment wrapper |
| [google-ai-edge/mediapipe](https://github.com/google-ai-edge/mediapipe) | **Apache-2.0** ✅ | USE — BlazePose; **single-person by default**, needs per-player crops for a court |
| [CMU-Perceptual-Computing-Lab/openpose](https://github.com/CMU-Perceptual-Computing-Lab/openpose) | **Non-commercial CMU academic licence** ✅ | **AVOID commercially** — and technically obsolete vs RTMPose/ViTPose |
| YOLO-pose | ships inside Ultralytics | Same **AGPL-3.0** — no licence escape |

---

## E) Action recognition / temporal event spotting

| Repo | Licence | Activity | Verdict |
|---|---|---|---|
| [open-mmlab/mmaction2](https://github.com/open-mmlab/mmaction2) | Apache-2.0 ✅ | 5,100★, last tag Oct 2023 | STUDY ONLY — visibly slowing |
| [facebookresearch/SlowFast](https://github.com/facebookresearch/SlowFast) | Apache-2.0 ✅ | 7,400★ | STUDY/USE — also the official home of X3D |
| [OpenGVLab/VideoMAEv2](https://github.com/OpenGVLab/VideoMAEv2) | **MIT** ✅ | 819★ | **USE** — permissive self-supervised video backbone; good for fine-tuning serve/set/spike/block/dig on limited clips |
| [facebookresearch/TimeSformer](https://github.com/facebookresearch/TimeSformer) | "Other" — **not re-verified** | 1,862★ | STUDY ONLY pending licence check |
| [OpenGVLab/InternVideo](https://github.com/OpenGVLab/InternVideo) | Apache-2.0 ✅ | 2,385★ | STUDY — too heavy for real-time; useful offline |
| [OpenGVLab/VideoMamba](https://github.com/OpenGVLab/VideoMamba) | Apache-2.0 ✅ | 1,128★ | STUDY — efficient, less battle-tested |
| [dingfengshi/TriDet](https://github.com/dingfengshi/TriDet) | **MIT** ✅ | 220★ (CVPR 2023) | USE/STUDY — temporal boundaries → rally segmentation |
| [happyharrycn/actionformer_release](https://github.com/happyharrycn/actionformer_release) | **MIT** ✅ | 575★ (ECCV 2022) | USE/STUDY |
| SoccerNet spotting family: [sn-spotting](https://github.com/SoccerNet/sn-spotting), [sn-teamspotting](https://github.com/SoccerNet/sn-teamspotting), [PTS-baseline](https://github.com/SoccerNet/PTS-baseline) | Unverified individually | active org | STUDY |
| [ZJLAB-AMMI/E2E-Spot-MBS](https://github.com/ZJLAB-AMMI/E2E-Spot-MBS) | MIT ✅ | 10★ | STUDY ONLY — a challenge *solution*, **not** the original E2E-Spot |
| Original E2E-Spot / T-DEED repos | — | — | ⚠️ **Could not verify exact URLs.** Do not cite a URL for either without a manual check |

---

## F) Sports frameworks / tooling

| Repo | Licence | Verdict |
|---|---|---|
| [roboflow/sports](https://github.com/roboflow/sports) | **MIT** ✅, 5.4k★ | **USE / best FORK candidate** — soccer/basketball detection, ball tracking, jersey OCR, player tracking/re-id, pitch keypoint homography, calibration. Swap in volleyball-trained models |
| [roboflow/supervision](https://github.com/roboflow/supervision) | **MIT** ✅, 48.4k★ | **USE** — annotation/tracking/dataset glue |
| [TrackingLaboratory/tracklab](https://github.com/TrackingLaboratory/tracklab) | **MIT** ✅, 249★ | **USE** — the modular framework sn-gamestate is built on; MIT is materially better than sn-gamestate's GPL for the orchestration layer |
| [SoccerNet/sn-gamestate](https://github.com/SoccerNet/sn-gamestate) | **GPL-3.0** ✅, 452★ | **STUDY CLOSELY** — architecturally almost exactly what VolleyVerse wants (detect → track → jersey OCR → team/role → pitch projection → minimap). GPL-3.0 needs legal review |
| [mguti97/No-Bells-Just-Whistles](https://github.com/mguti97/No-Bells-Just-Whistles) | **GPL-2.0** ✅, 61★ | USE/STUDY — real camera calibration/homography from field lines |
| [SoccerNet/sn-calibration](https://github.com/SoccerNet/sn-calibration) | Unverified, 109★ | STUDY |
| [Spiideo/soccersegcal](https://github.com/Spiideo/soccersegcal) | **MIT** ✅, 42★ | **USE/STUDY** — from a commercial sports-camera company; pitch-segmentation→homography transfers well to a volleyball court's simpler lines |
| [tryolabs/soccer-video-analytics](https://github.com/tryolabs/soccer-video-analytics) | MIT ✅, 307★ | STUDY — worked "ball possession from video" example |

**No dedicated deep-learning volleyball court-keypoint repo exists** beyond
`ovcourt`'s small pattern-based R function. That's a build-it-yourself gap, not
a landmine — and volleyball court geometry is simpler than a soccer pitch.

---

## G) Inference / edge

| Repo | Licence | Verdict |
|---|---|---|
| [microsoft/onnxruntime](https://github.com/microsoft/onnxruntime) | MIT ✅, 21.6k★ | USE — already the export target for the volleyball models above |
| [openvinotoolkit/openvino](https://github.com/openvinotoolkit/openvino) | Apache-2.0 ✅, 10.7k★ | **USE** — "cheap CPU box + OpenVINO" is a *validated* volleyball deployment path, not a generic claim |
| [NVIDIA/TensorRT](https://github.com/NVIDIA/TensorRT) | Apache-2.0 ✅, 13.1k★ | USE for GPU/Jetson |
| [NVIDIA-AI-IOT](https://github.com/NVIDIA-AI-IOT) org (`torch2trt`, `deepstream_python_apps`) | Mixed | STUDY — DeepStream/Jetson reference pipelines |
| [blakeblackshear/frigate](https://github.com/blakeblackshear/frigate) | MIT ✅, 35.8k★ | **STUDY (architecture)** — the most mature OSS "always-on camera → real-time local detection → event/clip generation → API". Structurally very close to a live-match capture box |

---

## H) Datasets

| Dataset | Licence | Note |
|---|---|---|
| The Volleyball Dataset (CVPR16) via [deep-activity-rec](https://github.com/mostafa-saad/deep-activity-rec) | **BSD-2-Clause** ✅ | 4,830 frames, broadcast clips, dated |
| [MCG-NJU/SportsMOT](https://github.com/MCG-NJU/SportsMOT) + [MixSort](https://github.com/MCG-NJU/MixSort) | Unverified / MIT ✅ | Multi-sport MOT (ICCV 2023) — **includes volleyball** |
| DeepSportradar challenges ([calibration](https://github.com/DeepSportradar/camera-calibration-challenge), [reid](https://github.com/DeepSportradar/player-reidentification-challenge), [ball-3d](https://github.com/DeepSportradar/ball-3d-localization-challenge)) | Unverified | Basketball/cricket, dormant since ~2023 |
| [SDOlivia/FineGym](https://github.com/SDOlivia/FineGym), [xujinglin/FineDiving](https://github.com/xujinglin/FineDiving) | Unverified / MIT ✅ | Fine-grained taxonomy references |
| [MCG-NJU/MultiSports](https://github.com/MCG-NJU/MultiSports) | "Other"/unverified | Spatio-temporal action; volleyball inclusion unconfirmed |
| SoccerNet ecosystem (19 repos) | Mixed | The best model of how to run a multi-task sports-video benchmark programme |
| VideoBadminton, P2A/P2ANet | — | **Not found.** Do not assume accessible repos exist |

---

## The 6–8 repos I'd actually `pip install` in week 1

1. **[asigatchov/fast-volleyball-tracking-inference](https://github.com/asigatchov/fast-volleyball-tracking-inference)** (MIT) — working volleyball ball detector on day one, F1≈0.9, 100+ FPS on CPU.
2. **[asigatchov/vball-net-pytorch](https://github.com/asigatchov/vball-net-pytorch)** — the training counterpart, for retraining on your own camera angles.
3. **[qaz812345/TrackNetV3](https://github.com/qaz812345/TrackNetV3)** (MIT) — the better-resourced architecture to benchmark against.
4. **[roboflow/supervision](https://github.com/roboflow/supervision)** (MIT) — saves weeks of bbox/tracking-ID/annotation plumbing.
5. **[roboflow/trackers](https://github.com/roboflow/trackers)** (Apache-2.0) — ByteTrack/BoT-SORT without touching AGPL.
6. **[lyuwenyu/RT-DETR](https://github.com/lyuwenyu/RT-DETR)** (Apache-2.0) — train the player/ball/net detector here so the stack starts commercially clean.
7. **[open-mmlab/mmpose](https://github.com/open-mmlab/mmpose)** (Apache-2.0) RTMPose/RTMO — serve vs set vs attack often hinges on arm/torso pose, not ball position alone.
8. **[masouduut94/volleyball_analytics](https://github.com/masouduut94/volleyball_analytics)** (GPL-2.0, **read-only reference**) — clone to study the 6-class taxonomy and the VideoMAE play/no-play gating idea. Do not vendor.

---

## Licensing landmines for a commercial product

- **Ultralytics YOLOv8/11/12 — AGPL-3.0.** Ultralytics' own licensing page states
  *"SaaS platforms, APIs, or cloud systems that use YOLO behind the scenes"*
  require a paid **Enterprise Licence** — triggered by **SaaS delivery itself**,
  not just redistribution. Either budget for it, or standardise on
  **RT-DETR / YOLOX / RTMDet (Apache-2.0)** and keep Ultralytics to prototyping.
- **mikel-brostrom/boxmot — AGPL-3.0.** Same trigger. Use `roboflow/trackers`.
- **OpenPose (CMU) — non-commercial academic licence.** No sale, no sublicence,
  no commercial derivative, trademarked name. Avoid; it's obsolete anyway.
- **mkoshkina/jersey-number-pipeline — CC BY-NC 3.0**, explicitly
  non-commercial. Build jersey OCR from Apache-2.0 **PARSeq** directly.
- **shukkkur/VolleyVision** — README says CC BY-NC-ND, GitHub metadata says
  AGPL-3.0. **Both block commercial reuse.** Study only.
- **GPL-2.0/3.0 copyleft** (no network trigger, still real):
  `masouduut94/volleyball_analytics`, `SoccerNet/sn-gamestate`,
  `No-Bells-Just-Whistles`, several YOLOv7+StrongSORT glue repos. A *distributed*
  product linking GPL code generally must release the combined work under the
  same licence. Get counsel before vendoring.
- **"Other"/blank licences** (MultiSports, MonoTrack, others) — this does **not**
  mean permissive. It defaults to **full copyright reserved**. Treat as AVOID
  until someone reads the actual LICENSE text.
- **Roboflow Universe datasets** — each of ~300 volleyball projects sets its own
  licence. Check per-dataset before training anything you ship.
- **General pattern:** much of the exciting sports-CV research code is
  university thesis output under NC or ambiguous terms. Budget engineering time
  to **reimplement architectures under clean licences** rather than assuming
  reference code is reusable.
