# Academic Literature Review — Automatic Volleyball Event Detection (STEP 5, REBUILT)

**Rebuilt 12 September 2026.** The original `04-academic-literature.md` was not
on disk when this session began; this file replaces it from a fresh pass across
four sub-literatures.

**Methodology.** ❌ WebSearch returned HTTP 403 (organisation egress policy) on
every query. ❌ Semantic Scholar and OpenAlex APIs returned HTTP 429 on every
attempt. ❌ arXiv's search endpoints, `paperswithcode.com` and `dblp` are
robots-disallowed. ❌ CVF Open Access began returning 403 partway through.
Verification was therefore done by **named-paper lookup and Crossref
bibliographic queries** against arXiv `/abs/`, CVF Open Access, BMVC
proceedings, J-STAGE, MDPI, PLOS, SciTePress and GitHub.

**This review is recall-limited — a floor, not a census.** There may be
2024–2026 work, particularly volleyball-specific work, that could not be
surfaced without a search engine.

**Markers.** ✅ verified on a page actually fetched · 📚 believed real, not
confirmed this session · ❌ looked for, not found · ⚠️ caveat

---

## PART 1 — Volleyball-specific CV and group activity recognition

### 1.1 The Volleyball Dataset lineage

| Title | Authors | Venue/Year | Status | What it does |
|---|---|---|---|---|
| A Hierarchical Deep Temporal Model for Group Activity Recognition | Ibrahim, Muralidharan, Deng, Vahdat, Mori | CVPR 2016 | ✅ | Two-stage LSTM (person → group). **Introduces the Volleyball Dataset** |
| Social Scene Understanding (SSU) | Bagautdinov, Alahi, Fleuret, Fua, Savarese | CVPR 2017 | ✅ | Jointly detects people + individual actions + group activity, no external detector |
| Learning Actor Relation Graphs (ARG) | Wu, Wang, Wang, Guo, Wu | CVPR 2019 | ✅ | GCN over actor-relation graph |
| Actor-Transformers (AT) | Gavrilyuk, Sanford, Javan, Snoek | CVPR 2020 | ✅ | Transformer over actor tokens from I3D + HRNet pose |
| SACRF | Pramono, Chen, Fang | ECCV 2020 (+ TIP 2021) | ✅ | Self-attention-augmented CRF over actor relations |
| GroupFormer | Li, Cao, Liu, Yang, Liu, Hou, Yi | ICCV 2021 | ✅ | Clustered spatio-temporal transformer |
| Spatio-Temporal Dynamic Inference Network (DIN) | Yuan, Ni, Wang | ICCV 2021 | ✅ | Person-specific dynamic interaction graph; explicitly cheaper |
| Dual-AI | Han, Zhang, Wang, Yan, Yao, Chang, Qiao | CVPR 2022 | ✅ | Two complementary transformer orderings + contrastive actor loss |
| **Detector-Free Weakly Supervised GAR (DFWSGAR)** | Kim, Lee, Cho, Kwak | CVPR 2022 | ✅ | **Most important for us** — needs neither GT boxes nor a detector |
| COMPOSER | Zhou, Kadav, Shamsian et al. | ECCV 2022 | ✅ | Group activity from **skeletons only**, no RGB |
| RelPosGAR | Li, Qing, Tao et al. | Pattern Recognition 2026 | ✅ metadata | Current-generation weakly supervised skeleton GAR |
| CERN, stagNet, HRN, CRM, SBGAR, SAM, PCTDM | — | 2017–2020 | 📚 | Appear as row labels inside fetched comparison tables only |

**Verified accuracy progression** (from GroupFormer's ICCV 2021 table and
Dual-AI's CVPR 2022 Table 1): HDTM 81.9 → ARG 92.5 → Gavrilyuk et al. 93.0–94.4
→ Pramono et al. 94.1–95.0 → GroupFormer 95.7 (I3D+Pose); Dual-AI 94.4 (95.4
with flow), and **92.7 on 50% of the training data**.

**⚠️ The protocol caveat that deflates all of the above.** ✅ DFWSGAR (CVPR 2022)
re-evaluates under a **matched ResNet-18 backbone with no ground-truth boxes at
test time**: ARG 87.4, DIN 86.5, SAM 86.3, AT 84.3, SACRF 83.3; DFWSGAR itself
90.5. The paper states plainly that existing models *"demand ground-truth
bounding box labels of actors even in testing."* A further "Merged MCA" column
(left/right collapsed) reaches 94.4 vs 90.5 unmerged — **a meaningful share of
the residual error is simply which side of the net**, the easiest thing to fix
with a calibrated camera and the least interesting.

### 1.2 ⚠️ Dataset statistics — an unreconciled conflict

Two verification passes returned different numbers and both were read from
primary sources:

| Source | Frames | Videos | Labels |
|---|---|---|---|
| GitHub `mostafa-saad/deep-activity-rec` | 4,830 | 55 | 8 group + 9 individual |
| CVPR 2016 paper page | 1,525 | 15 | 6 team + 7 player; 51.1% accuracy |

Almost certainly an original and an extended release. **Do not quote either
without stating which.**

From the repo ✅: group labels are Right/Left × {set, spike, pass, winpoint};
individual labels are Waiting 3,601 · Setting 1,332 · Digging 2,333 · Falling
1,241 · Spiking 1,216 · Blocking 2,458 · Jumping 341 · Moving 5,121 ·
**Standing 38,696**. Annotated frame = centre of a 41-frame window. Code is
BSD-2-Clause; ⚠️ **the dataset itself has no stated licence** and derives from
Olympic broadcast footage — a live legal question for commercial use.

### 1.3 Why ~95% on this benchmark does not mean per-touch events work

1. **8 labels = 4 events × 2 sides.** Strip the side: `pass / set / spike /
   winpoint`, and "winpoint" is a celebration, not a contact. Effectively a
   3-way choice on a pre-isolated clip.
2. **One label per 41-frame clip, not per touch.** A rally has 6–12 contacts.
3. **No serve, no reception, no free ball, no dink, no cover.** Four contact
   classes total; a DataVolley skill alphabet is not representable.
4. **Two-thirds of individual labels are "standing"** (38,696 of ~56,000).
5. **No ball annotation at all** — a model can score 95% without ever localising
   the ball, so it cannot decide who touched it.
6. **No player identity** — no jersey, no rotation slot, no persistent track ID.
7. **No quality grading** — half of VolleyVerse's schema is unrepresentable.

### 1.4 The honest benchmark: MultiSports

✅ **MultiSports** (Li, Chen, He, Wang, Wu, Wang — ICCV 2021): 4 sports, 3,200
clips, 37,701 action instances, ~902k boxes, frame-wise at 25 fps. Its **12
volleyball classes are genuine per-touch classes**: serve, defend, protect,
block, first hit pass, spike, second hit pass, adjust, dink, no offensive
attack, save, second attack.

**Verified baselines:** SlowFast Det. **27.72 frame-mAP@0.5 · 24.18
video-mAP@0.2 · 9.65 video-mAP@0.5**; MOC (K=11) 25.22 / 12.88 / 0.62.

⚠️ Licence **CC BY-NC 4.0 — non-commercial.** ⚠️ One pass could not confirm
volleyball is among the four sports; the other read the 12-class list from the
ICCV PDF.

### 1.5 Volleyball-specific papers

| Title | Authors | Venue/Year | Status |
|---|---|---|---|
| Ball tracking and 3D trajectory approximation … single-camera volleyball | Chen, Tsai, Lee, Yu | MTAP 2011 | ✅ metadata |
| Physics-Based Ball Tracking in Volleyball Videos … Set Type Recognition | Chen, Chen, Lee | ICASSP 2007 | ✅ metadata |
| Trajectory-based ball detection … shot-type identification in volleyball | Chakraborty, Meher | SPCOM 2012 | ✅ metadata |
| Player tracking using prediction-after-intersection particle filter | Cheng, Shiina, Zhuang, Ikenaga | APSIPA 2014 | ✅ metadata |
| **Tracking of Ball and Players in Beach Volleyball Videos** | Gomez, Herrera López, Link, Eskofier | **PLoS ONE 2014** | ✅ **full text** |
| Particle Filter with Ball Size Adaptive Tracking Window … 3D position | Cheng, Zhuang, Wang, Honda, Ikenaga | PCM 2015 | ✅ metadata |
| **Ball State Based Parallel Ball Tracking and Event Detection** | Cheng, Ikoma, Honda, Ikenaga | **IEICE E100.A(11) 2017** | ✅ |
| Court-Based Volleyball Video Summarization … Rally Scene | Itazuri, Fukusato, Yamaguchi, Morishima | CVPRW 2017 | ✅ |
| **View Priority Based Threads Allocation … Real-Time 3D Ball Tracking** | Hou, Deng, Cheng, Ikenaga | **IEICE E101.D(12) 2018** | ✅ |
| Ball Motion State and Abrupt Pose Features … Qualitative Action Recognition | Liu, Cheng, Ikenaga | ICISIP 2018 | ✅ metadata |
| 3D Ball Motion and Relative Position … Real-time Start Scene Detection on GPU | Liang, Cheng, Ikenaga | ICISIP 2018 | ✅ metadata |
| Accurate ball tracking in volleyball actions to support referees | Kurowski, Szeląg, Załuski, Sitnik | Opto-Electronics Review 2018 | ✅ metadata |
| Multi-physical and Temporal Feature … **Monocular 3D Volleyball** Trajectory | Dong, Cheng, Ikenaga | MVA 2021 | ✅ metadata |
| **A Hierarchical Classification for Automatic Assessment of the Reception Quality … Deep Learning** | Nako, Ogata, Matsui, Hamada, Ohya | **ICPRAM 2025** | ✅ metadata |
| Performance Automatic Evaluation of Volleyball Reception by Video Analysis | Mori, Sawano | IEEJ Trans. EIS 145(10) 2025 | ✅ metadata |
| A Proposal of a Spike Event Classification Method Based on Ball Trajectory | Mori, Masuda, Sawano | IIAI LIIR 2023 | ✅ metadata |
| Transformer-Based Multi-Player Tracking and Skill Recognition | Jiang, Yang, Gang | IEEE Access 2025 | ✅ metadata |
| Volleyball technical action recognition based on CNN-LSTM | Zhang, Tian, Qi | Discover AI 2025 | ✅ metadata |
| Volleyball Action Recognition based on Skeleton Data | Liang, Isakunovich | FCIS 2023 | ✅ metadata |
| ⚠️ Sun & Zhang (Mobile Information Systems 2022); Qin (Scientific Programming 2021) | — | — | ✅ metadata — **venues with poor integrity records** |

**❌ NOT FOUND, and these absences are findings:**
- Any dedicated paper on **player–ball touch/contact detection in volleyball**.
  The nearest analogue in any sport is **audio-based hit detection in table
  tennis** (Zhang, Dou, Chen — ICPR 2006 ✅).
- Any CV-based **automated volleyball officiating** beyond Kurowski 2018.
  Soccer has VARS (Held et al., CVPRW 2023 ✅); volleyball has nothing modern.
- Any paper that **emits DataVolley scout codes end to end from video**.
- Any **volleyball action-spotting benchmark**.

### 1.6 The two 2025 grading papers — the most relevant work found

✅ **Nako et al. (ICPRAM 2025)** trains a TimeSformer on single-view match video
**against manually recorded DataVolley labels**, with a hierarchical front/back →
A/B vs C → A vs B pass grading. ✅ **Mori & Sawano (IEEJ 2025)** does automatic
reception evaluation from video. ⚠️ **No accuracy figures could be read** from
either (SciTePress metadata confirmed; J-STAGE returned HTTP 500).

**The strategic point:** both groups built their training corpus by **aligning
video to manually scouted files**. That is the only route anyone has found to
per-touch volleyball grades, and it is exactly what VolleyVerse can generate as
a by-product.

---

## PART 2 — Ball tracking and 3D reconstruction

### 2.1 The TrackNet lineage

All generations stack N consecutive RGB frames into an encoder-decoder that
regresses a **2D Gaussian heatmap** whose argmax is the ball. The stacking is
what makes an 11-pixel blurry blob findable — the network learns the flight
pattern, not the appearance.

| Model | Venue | Status | Numbers |
|---|---|---|---|
| TrackNet | Huang et al., AVSS 2019 (arXiv 1907.03698) | ✅ | Universiade final P 99.7 / R 97.3 / **F1 98.5**; 10-fold CV **F1 84.3** |
| TrackNetV2 | Sun et al., ICPAI 2020 | ✅ metadata | ⚠️ Own metrics never read — all V2 numbers here are third-party and they disagree |
| TrackNetV3 | Chen & Wang, ACM MM Asia 2023 | ✅ | Authors' table: **Acc 97.51 / F1 98.56 @ 25.11 FPS**; V2 97.03 F1; YOLOv7 68.00 |
| TrackNetV4 | Raj, Wang, Gedeon, arXiv 2409.14543 | ✅ existence, ❌ numbers | ⚠️ "Research report", not peer-reviewed; no numbers in abstract; no code link |

⚠️ **WASB's independent re-implementation produces much lower numbers than the
original papers** — TrackNetV2 scores F1 83.6 on volleyball under a common
protocol. **Treat single-paper TrackNet numbers as optimistic.**

### 2.2 WASB — the single most reusable asset found

✅ **"Widely Applicable Strong Baseline for Sports Ball Detection and Tracking"**
— Tarashima, Haq, Wang, Tagawa (NTT Com / Tokyo Metropolitan Univ.), **BMVC
2023**, paper 0310. HRNet-style high-res backbone, N=3 stacked frames →
H×W×9 at 288×512, position-aware training, temporally-consistent inference.
**MIT licence, model zoo includes pre-trained volleyball weights.**

**Volleyball split:** 143,213 train / 54,817 test frames, 1280×720, 39 train
games / 16 test games. Clips from Ibrahim et al. (CVPR 2016) with ball
annotations from Perez, Liu & Kot (Pattern Recognition 2022).

**Volleyball results at τ = 4 px:**

| Method | F1 | Acc | FPS |
|---|---|---|---|
| DeepBall | 60.0 | 57.1 | — |
| DeepBall-Large | 59.5 | 53.0 | — |
| BallSeg | 68.4 | 75.0 | — |
| TrackNetV2 | 83.6 | 72.3 | 17.6 |
| ResTrackNetV2 | 84.2 | 74.7 | — |
| MonoTrack | 85.1 | 75.9 | 19.7 |
| WASB (step=3) | 86.5 | 77.9 | 18.0 |
| **WASB (step=1)** | **88.0** | **80.0** | 15.8 |

**Cross-sport WASB F1:** soccer 88.2 · tennis 95.6 · badminton 93.1 ·
**volleyball 88.0** · basketball 82.6. **Volleyball is the second-hardest of
the five.**

### 2.3 The reality check

✅ **Gomez et al. (PLoS ONE 2014)**, single camera, 25 fps, real beach volleyball
matches: ball tracking **54.2%** (trajectory growth) / 42.1% (Hough) over 535
frames; **ball-contact frame correct in 48.9%** of 190 contacts within a
±10-frame tolerance. The authors wrote, in print:

> *"Tracking results of over 90% from the literature could not be confirmed."*

**The gap between 88 and 54 is the gap between curated broadcast clips and a
camera someone put on a tripod.**

### 2.4 3D ball position

**Multi-view — effectively solved for volleyball:**
- ✅ Cheng et al. (IEICE 2017): multi-view HDTV volleyball, joint 3D tracking +
  contact classification. **Event detection 88.61%, 3D tracking >99%, 606 real
  events**, 2014 Inter-High School Men's Volleyball, Japan.
- ✅ Hou et al. (IEICE 2018): **4 cameras at the four corners** of Tokyo
  Metropolitan Gymnasium, particle filter. **99.23% success; 75.1 ms/frame CPU →
  3.05 ms/frame GPU (24.6×).**
- ⚠️ "Success rate" is the authors' own criterion; **no metric error threshold
  was retrievable.** Do not read it as "99% of frames within X cm".

**Monocular — not there:**
- ✅ **3D Ball Localization From a Single Calibrated Image** (Van Zandycke & De
  Vleeschouwer, CVPRW 2022): depth from apparent ball diameter. Diameter MAE
  **1.6 ± .1 px** → **3D position MAE 2.3 ± .2 m** / 1.8 ± .2 m; ~10% relative
  distance error. **Unusable for deciding who touched the ball.**
- ✅ **MonoTrack** (Liu & Wang, CVSports 2022), badminton: ballistic model with
  quadratic drag, DLT from 4 court corners + 2 net-post tips, priors on hit
  position (feet + assumed 2 m contact height), landing point and court bounds.
  2D track **88.6%**; court **85.5%**, IoU 0.97; **hit detection 89.7% acc /
  94.3% R / 94.9% P / F1 0.946**; **synthetic 3D error 8.0 cm with priors, 14.9
  cm without**; real video reports only reprojection error 23.3–37.1 px.
- ⚠️ **MonoTrack's contact-height prior does not transfer to volleyball**, where
  contact ranges from a floor dig to a 3.2 m spike.
- **❌ There is no verified metric 3D error on real monocular video anywhere in
  this review.**

### 2.5 Contact detection — the pattern

| Setting | Contact / event accuracy |
|---|---|
| Table tennis, 120 fps, controlled (TTNet, CVSports 2020) | ✅ **97.0%** (ball 2 px RMSE, <6 ms inference) |
| Badminton, monocular, curated (MonoTrack HitNet) | ✅ **89.7%**, F1 0.946 (derivative-test baseline: 53.8%) |
| Volleyball, 4-camera HDTV, 606 real events | ✅ **88.61%** |
| Beach volleyball, 1 camera, 25 fps, real matches | ✅ **48.9%** (±10 frames) |

Mechanically, every approach detects contact as a **discontinuity in the fitted
trajectory**. ✅ SoccerNet frames the same idea as *"retrieving all timestamps
related to the soccer ball change of state."* At 30 fps a millisecond-scale
contact is never observed directly — it is **interpolated**, so temporal
resolution is inherently ±1 frame and the 3D contact point is an extrapolation.

### 2.6 Fast Moving Objects — the missing link for gym footage

✅ **The World of Fast Moving Objects** (Rozumnyi et al., CVPR 2017) defines the
FMO regime: an object moving further than its own size within one exposure,
*"very common in sports videos."* ✅ **DeFMO** (CVPR 2021) recovers sub-frame
sharp appearance and trajectory from a single blurred frame — **volleyballs are
in its object set.** ✅ **FMODetect** (ICCV 2021) is the first learning-based FMO
detector. ✅ **Recognizing and Recovering Ball Motion Based on Low-Frame-Rate
Monocular Camera** (Zhang et al., Applied Sciences 2023) fits semi-elliptical
intensity profiles to **motion streaks**, reporting **streak-detection F1 0.97**.

🔢 At 130 km/h the ball travels **1.2 m per frame at 30 fps (~5.7 ball
diameters)**; at 1/120 s shutter it smears ~0.3 m. **That is unambiguously the
FMO regime — and none of the heatmap trackers (TrackNet 1–4, WASB, MonoTrack)
model streaks**, having been trained on well-lit broadcast footage.

❌ **The FMO literature and the ball-tracking literature have not been joined for
volleyball.** This is a genuine open research direction with direct product
relevance for badly-lit gyms.

Also noted: ✅ event-camera ball detection exists (Glover & Bartolozzi, IROS
2016; Gao et al., Chinese Optics Letters 2022) but nothing volleyball-specific.
❌ No independent accuracy validation of Hawk-Eye could be found — only
applications of its data.

---

## PART 3 — Temporal action spotting and online detection

### 3.1 Task definitions (verified — this is where most misreading happens)

- ✅ **SoccerNet Action Spotting**: a prediction is correct if its timestamp
  falls within tolerance δ. **Loose Average-mAP averages δ = 5–60 s. Tight
  Average-mAP averages δ = 1–5 s** (introduced 2022). 17 classes.
- ✅ **SoccerNet Ball Action Spotting**: **12 dense ball-contact classes** (Pass,
  Drive, Header, High Pass, Out, Cross, Throw In, Shot, Block, Tackle, Free
  Kick, Goal), metric **mAP@1 (1-second tolerance)**, **only 7 annotated games**.
- ✅ **E2E-Spot / T-DEED "precise spotting"** uses **δ = 1 and 2 frames**
  (≈33–80 ms) — two orders of magnitude tighter than SoccerNet loose.
- ⚠️ **Online Action Detection** on THUMOS/TVSeries uses **per-frame mAP /
  mcAP**, which is **not comparable** to segment mAP@tIoU. Comparing MiniROAD's
  71.8 to TriDet's 69.3 is a category error.

### 3.2 Key papers

| Title | Venue | Status | Offline/Online | Verified numbers |
|---|---|---|---|---|
| SoccerNet | CVPRW 2018 | ✅ | Offline | Avg-mAP 49.7 (δ 5–60 s); **~1 event per 6.9 min** |
| SoccerNet-v2 | CVPRW 2021 | ✅ | Offline | ~300,000 annotations, 500 games |
| SoccerNet 2023 Challenges | 2023 | ✅ | — | AS winner **71.31 tight avg-mAP**; **BAS winner 86.47 mAP@1** (baseline 62.72) |
| NetVLAD++ | CVPRW 2021 | ✅ content | Offline | **53.4 Avg-mAP, +12.7** — explicitly splits context *before* vs *after* the spot |
| **E2E-Spot** (Hong, Zhang, Gharbi, Fisher, Fatahalian) | **ECCV 2022** | ✅ | Offline (short-clip, bidirectional) | mAP@δ=1 frame: **Tennis 96.1 / 96.9**; FS-Comp 81.0/83.4; FineDiving 68.4; **FineGym-Full 47.9–51.8**; SoccerNet-v2 61.82 tight |
| **T-DEED** (Xarles, Escalera, Moeslund, Clapés) | **CVPRW 2024** | ✅ | Offline | mAP@δ=1/δ=2: FS-Comp **85.15 / 91.70**; FS-Perf 88.17 / 95.87; FineDiving 73.23 / 88.88. **1st place 2024 SoccerNet BAS** |
| ActionFormer | ECCV 2022 | ✅ | **Offline** | THUMOS14 avg **66.83** |
| TriDet | CVPR 2023 | ✅ | **Offline** | THUMOS14 avg **69.3** |
| LSTR | NeurIPS 2021 (Spotlight) | ✅ | **Online** | THUMOS per-frame 69.5; TVSeries 88.1/89.1 |
| TeSTra | ECCV 2022 | ✅ | **Online/streaming** | THUMOS **71.2**; **6× faster** than sliding-window equivalent |
| MAT | ICCV 2023 | ✅ | **Online** | **72.6 FPS**, 94.6M params |
| **MiniROAD** | **ICCV 2023** | ✅ | **Online** | THUMOS **71.8** at **0.0158 GFLOPs / 15.8M params** (~0.4% of prior SOTA compute); TVSeries 89.6 |
| CAG-QIL | ICCV 2021 | ✅ | **Online (On-TAL)** | Formalises emitting whole instances from a stream |
| **MATR** | **ECCV 2024** | ✅ | **Online** | **The key online-vs-offline table — see below** |
| TTNet | CVSports 2020 | ✅ | **Online/real-time** | **97.0%** event spotting; ball 2 px RMSE; **<6 ms** |
| P2ANet | arXiv 2022/2024 | ✅ | Offline | 14 classes, 25 fps broadcast: **48% AUC localisation, 82% top-1 recognition**; authors call it still-challenging |
| OadTR, GateHUB, De Geest et al. (TVSeries) | ICCV'21 / — / ECCV'16 | 📚 | Online | Numbers read only inside MiniROAD's tables |

### 3.3 The cost of going causal — the central number

✅ **MATR (ECCV 2024), Table 1 — THUMOS14 segment mAP, same metric, same data:**

| Setting | Method | mAP @ 0.3/0.4/0.5/0.6/0.7 | Average |
|---|---|---|---|
| Offline | **TriDet** | 83.6 / 80.1 / 72.9 / 62.4 / 47.4 | **69.3** |
| Offline | ActionFormer | 82.1 / 77.8 / 71.0 / 59.4 / 43.9 | 66.8 |
| Offline | MUSES | 68.9 / 64.0 / 56.9 / 46.3 / 31.0 | 53.4 |
| Online | **MATR** | 70.3 / 62.7 / 52.1 / 38.6 / 23.7 | **49.5** |
| Online | OAT-OSN | 63.0 / 56.7 / 47.1 / 36.3 / 20.0 | 44.6 |

**≈20 mAP points, ~29% relative.** On MUSES the gap is 4.2 points (~23%).
MATR beats *older* offline methods but not modern offline SOTA — exactly as its
abstract claims.

✅ Corroboration: **NetVLAD++** gains +12.7 Avg-mAP specifically by separating
context before and after the spot, arguing that treating surrounding context as
one undifferentiated block is *"sub-optimal"* — direct evidence that post-event
frames carry real signal, which is what a causal system gives up.

✅ **Compute is not the blocker.** MiniROAD: 71.8 per-frame mAP at 0.0158
GFLOPs. TeSTra: 6× speedup. MAT: 72.6 FPS. **Accuracy-without-lookahead is.**

⚠️ **The untested hypothesis that matters most to us:** E2E-Spot and T-DEED are
*not* full-video methods — they run bidirectional temporal modules over short
clips. A live system could likely buy back most of the causal penalty with a
fixed 1–2 s lookahead buffer. **❌ No paper measuring the accuracy-vs-lookahead
curve could be found.** This is the single most decision-relevant missing number
in the literature for a live volleyball product.

### 3.4 Achievable temporal precision

✅ Best verified: **±1 frame (≈33–40 ms)** — E2E-Spot 96.1–96.9 mAP@δ=1 on
tennis. ✅ Relaxing by one frame is worth **+7 to +16 mAP** (T-DEED) — precision
is expensive at the margin. ✅ TTNet demonstrates sub-10 ms causal spotting at
97.0% *when you control the camera and frame rate* (120 fps). ✅ P2ANet is the
counterweight: at **25 fps broadcast**, dense fine-grained detection yields only
48% AUC.

---

## PART 4 — Player identity, pose, calibration

### 4.1 Jersey number recognition — the wall

✅ **A General Framework for Jersey Number Recognition in Sports Video** —
Koshkina & Elder, CVPRW 2024. Legibility classifier → pose-guided crop → scene-
text recognition → tracklet vote.

**End-to-end: 91.4% hockey images · 87.45% SoccerNet test · 79.31% SoccerNet
challenge** (prior SOTA 68.53 / 73.77).

**The load-bearing number, from their data description:**

> **Legible player crops: hockey train 4,706 / 94,036 (5.0%); val 923 / 14,138
> (6.5%); test 2,158 / 24,809 (8.7%).**

✅ SoccerNet's own organisers: *"the jersey numbers might be visible on a very
small subset of the whole tracklet."* ✅ SoccerNet jersey task: 2,853 tracklets;
classes 1–99 **plus a "−1" class for no visible number**; plain tracklet
accuracy. ✅ 2023 challenge: ZZPM 92.85%, UniBw 90.95%, random baseline 3.93%.

**Three reasons the 92.85% headline does not apply to event attribution:**
1. ✅ The metric **credits correct "−1 / not visible" predictions.** For
   attribution, "−1" is a miss.
2. ✅ SoccerNet-GSR labels a tracklet if the number is visible **in at least one
   frame of a 30-second clip.** An event log needs identity at a 40 ms instant.
3. Even the best end-to-end soccer figure (79.31%) means **one tracklet in five
   is wrong.**

✅ Additional verified failure modes: *"when the jersey number of the occluding
player is visible it can affect both legibility and number predictions for the
tracklet"*; *"only one digit may be visible even when the jersey number consists
of two digits."*

⚠️ `mkoshkina/jersey-number-pipeline` is **CC BY-NC 3.0 — non-commercial.**

### 4.2 The cascade — SoccerNet Game State Reconstruction

✅ **SoccerNet GSR** (Somers, Joos, Cioppa et al., CVPRW 2024) — video → each
person's pitch position + role + team + jersey on a minimap. Metric GS-HOTA.

| | Score |
|---|---|
| Baseline (test / val / challenge) | **22.26 / 18.05 / 23.36** |
| 2024 challenge winner (Constructor tech) | **63.81** |
| 2nd–3rd | 43.15 / 34.40 |

**Oracle ablation** (all other modules = ground truth): Team Side **92.00** ·
ReID 87.42 · **Jersey 56.75** · Calibration 51.39 · Pitch 49.99 · BBox detection
35.28. Image-plane MOT: 57.64 HOTA / 67.42 DetA / 49.42 AssA / 80.79 MOTA.

✅ **Why the baseline collapses:** GS-HOTA's identity term is binary — *"failing
to correctly predict at least one attribute turns the corresponding detection
into a False Positive."*

⚠️ **Two caveats that make 63.81 generous:** the winner used a **closed-set
TeamID model trained on 111 specific uniforms**, and **GS-HOTA allows a 5-metre
localisation tolerance** — 🔢 over half a volleyball court.

⚠️ **A volleyball-specific problem:** the GSR work assumes the bottom-centre of a
box lies on the ground plane and notes this *"limits the precision of the
estimated locations in the case of jumps."* Spikes and blocks are the worst case.

### 4.3 Tracking and re-ID

✅ **SportsMOT** (ICCV 2023) — 240 sequences, >150k frames, >1.6M boxes,
**includes volleyball**. Per-sport (MixSort-Byte): **volleyball 72.5 HOTA / 87.0
IDF1 / 66.8 AssA / 78.7 DetA**; basketball 60.8 / 67.8 / 46.8; football 66.4 /
73.6 / 56.3. **Volleyball is the best split.** Domain characterised as *"similar
yet distinguishable appearance."*

⚠️ AssA 66.8 means roughly a third of the association work is still wrong, and
errors cluster at the net during a block and on the floor after a dig.

✅ SoccerNet Re-ID: 340,993 thumbnails; 2023 winner 93.26 mAP / 91.26 R-1.
✅ DeepSportradar re-ID (basketball): baseline ~72.7 mAP test / ~61.1 challenge;
2022 winner (CLIP ViT-L/14) 98.44% mAP. ✅ BPBreID (WACV 2023) — part-based
re-ID robust to occlusion, ancestor of the GSR baseline's PRTReID.

⚠️ **Within-team re-ID is not what these benchmarks measure.** They are
cross-viewpoint *retrieval against a gallery*, not telling teammate A from
teammate B in identical kit after a 3-second occlusion.

### 4.4 Pose and skeleton action recognition

✅ **ViTPose** (NeurIPS 2022): 81.1 mAP COCO test-dev; **92.8 AP OCHuman; 78.3 AP
CrowdPose**. ✅ **RTMPose** (2023): RTMPose-m 75.8% AP COCO at 90+ FPS CPU /
430+ FPS GTX1660Ti. ✅ **RTMO** (CVPR 2024): one-stage, cost does not grow with
person count — **74.8% AP COCO val at 141 FPS (V100)**. All Apache-2.0.

✅ **PoseConv3D / "Revisiting Skeleton-based Action Recognition"** (CVPR 2022
oral): **Volleyball dataset 91.3% top-1** (group activity, validation, tracking
boxes) vs 89.2 for GCN. Robustness: dropping one limb keypoint per frame costs
**<1% for PoseConv3D vs 14.3% for GCN**.

⚠️ **Two warnings from the same paper.** **Box quality propagates:** on FineGYM,
GT boxes 92.0 → tracking boxes 85.3 → **detection boxes 75.8**. And skeleton-only
methods are, by the authors' framing, *"immune to contextual nuisances"* —
**including the ball.** Pose tells you a player made an overhead two-handed
motion; it does not tell you the ball was there. ✅ The original Volleyball paper's
observation still stands: *"confusion between set and pass activities, as these
activities often may look similar"* — precisely the set-vs-reception distinction
an event log needs.

**Conclusion: pose alone is insufficient. Ball tracking must be fused with pose.**

### 4.5 Calibration — easier for us than the literature suggests

✅ SoccerNet Camera Calibration: 20,028 train / 2,104 test images; 2023 winner
Sportlight **ACC@5 73.22% at 75.59% completeness** (baseline 0.08 / 13.54%).
✅ **TVCalib** (WACV 2023), ✅ **No Bells Just Whistles** (CVPRW 2024, GPL-2.0),
✅ **ProCC** (CVPRW 2024) — which argues existing calibration benchmarks are
misleading because *"progress is impeded by outdated benchmarking criteria."*

**All of these address a moving broadcast PTZ camera.** A fixed gym camera on a
rigid 18×9 m court is a one-time setup task, not a per-frame estimation problem.
**Do not import this risk.**

✅ Team assignment is the best-behaved sub-problem: 92.00 oracle GS-HOTA, and the
GSR baseline does it with k-means over two clusters of tracklet-averaged re-ID
embeddings plus a left/right heuristic.

---

## PART 5 — Synthesis

### Genuinely solved
- Player detection and short-horizon tracking on a bounded volleyball court
  (best SportsMOT split, 72.5 HOTA).
- Real-time multi-person 2D pose for 12+ people (RTMO 74.8 AP @141 FPS).
- 2D ball detection on well-lit, well-framed volleyball video — **WASB, MIT,
  volleyball weights, F1 88.0**.
- 3D ball position with 4 calibrated corner cameras — >99% success, real-time on
  GPU, demonstrated in a real gymnasium.
- Rally / game-state segmentation (Itazuri CVPRW 2017; Liang ICISIP 2018
  real-time on GPU).
- Team assignment into two clusters.
- Fixed-camera court calibration (a setup task, not an ML problem).
- Task formalisation and metrics for action spotting.
- Real-time inference cost at every stage — **compute is not the bottleneck.**

### Partially solved
- Contact-moment detection: ~89% high-frame-rate/multi-camera, **~49% at 25 fps
  monocular**.
- Monocular 3D: 8 cm *in simulation*; **no published real-video metric error**.
- Jersey recognition at tracklet level with a legibility gate — but only
  **5–9% of frames legible** and a metric that rewards "not visible".
- Coarse volleyball action classification from pose (91.3% on 6 group classes)
  — with documented set-vs-pass confusion.
- Online/causal instance-level localisation — works, at **~20 mAP below offline**.
- End-to-end game-state reconstruction: 22.26 → 63.81 GS-HOTA at 5 m tolerance
  with closed-set uniforms.
- Quality grading: **reception only**, two 2025 papers, no verified accuracy.

### Open
- ❌ **Touch/contact detection in volleyball** — no dedicated paper exists. The
  atomic primitive of every scout code is essentially unaddressed.
- ❌ **Per-touch attribution to a named player** — no paper reports an accuracy
  for "which specific player touched the ball".
- ❌ **Ordered per-rally event sequences** with correct touch counts. No
  published model exploits the 3-touch rule as a structural constraint.
- ❌ **End-to-end DataVolley code generation from video.**
- ❌ **Any volleyball action-spotting benchmark.**
- ❌ **Automated volleyball officiating** beyond one 2018 paper.
- ❌ **FMO/motion-streak methods joined to sports ball tracking** — the correct
  technical family for a badly-lit gym, never benchmarked on volleyball.
- ❌ **Audio for volleyball** — the one modality that is occlusion-immune,
  lighting-immune and frame-rate-independent, and nobody has tried it. Table
  tennis solved the analogue with audio in 2006.
- ❌ **The accuracy-vs-lookahead curve** for precise event spotting.
- ❌ **Volleyball-specific jersey legibility statistics.**
- ❌ Beach volleyball — effectively greenfield (one substantial paper, 2014).

### The binding constraint is data, not architecture

The architectures needed — detection, tracking, pose, temporal action detection,
transformers over actor tokens — are mature and well-published. **What does not
exist publicly is a volleyball corpus with per-touch timestamps, contacting-
player identity, a full skill alphabet and DataVolley-style grades.**

Every 2025 paper that gets near grading built its own private dataset by aligning
video to manually scouted files. ✅ The closest public analogue — SoccerNet Ball
Action Spotting — trains on **7 annotated games**. That is the gap, and it is
the one an operating league-management product is uniquely placed to close.

### Where our problem is harder than every published benchmark

1. **Event density** — ✅ SoccerNet: ~1 event per 6.9 min. Volleyball: 🔢
   0.5–1.5 events/second. **200–500× denser.**
2. **Attribution is a separate pipeline** — ✅ SoccerNet treats re-ID, tracking
   and jersey numbers as three distinct tasks.
3. **Quality grading has no benchmark analogue** outside two 2025 reception papers.
4. **Data volume** — the closest task has 7 games.
5. **Simultaneity and occlusion** — blocks and digs involve near-simultaneous
   contacts by multiple players, a failure mode absent from tennis, diving and
   figure skating where the best numbers come from.
6. **Gym conditions** — 🔢 at 30 fps the ball moves 1.2 m between frames and
   motion-blurs 0.3 m at 1/120 s. Expect real-gym ball tracking closer to the
   beach paper's 54% than to WASB's 88% unless capture is controlled.
