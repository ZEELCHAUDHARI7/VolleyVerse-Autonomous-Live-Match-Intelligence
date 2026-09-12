# VolleyVerse Hardware R&D: Cameras, Edge Compute, Sensors & Venue Reality (Sept 2026)

**Methodology note:** WebSearch was unavailable for this entire research session (proxy returned HTTP 403 on every query — a session-level restriction, not a retry-able error). All data below comes from direct WebFetch of manufacturer, distributor, and reference pages. Every fetched fact is marked ✅ with its URL. Where a page renders prices via JavaScript (common on GoPro/DJI/Insta360/Veo/Hudl/Pixellot/Basler/Spiideo storefronts) or a distributor blocked the request, I say so explicitly and label the figure 📚 (recalled from training, not re-verified live today — confirm before budgeting) or 🚫 (genuinely not public; quote-only). Do not treat 📚 figures as current quotes.

---

## 1. CAMERA OPTIONS

### 1a. IP/PoE security-style cameras — the 25/30fps wall

Direct spec-page fetches confirm the core problem stated in the brief: **budget/mid-range security cameras are frame-rate-capped well below what a 100+ km/h volleyball needs**, regardless of resolution.

| Model | Resolution | Max FPS | Price | Source |
|---|---|---|---|---|
| Reolink RLC-1224A | 12MP (4512×2512) | **20fps** (default; mainstream stream only goes 2–20fps) | Not public on product page | ✅ [reolink.com/product/rlc-1224a](https://www.reolink.com/product/rlc-1224a/) |
| Reolink RLC-810WA | 4K (3840×2160, 8MP) | **20fps fixed** — no 1080p or 60fps mode offered at all | Not public | ✅ [reolink.com/us/product/rlc-810wa](https://www.reolink.com/us/product/rlc-810wa/) |
| Axis M3216-LVE | 4MP (2688×1512) | **25/30fps max**, no higher-fps mode at any resolution documented | Not public | ✅ [axis.com/products/axis-m3216-lve](https://www.axis.com/products/axis-m3216-lve) |
| Axis P3737-PLE (4-sensor panoramic) | 4×5MP per channel | **20fps per channel** | Not public | ✅ [axis.com/products/axis-p3737-ple](https://www.axis.com/products/axis-p3737-ple) |

Three independent camera datasheets (two Reolink, one Axis), across three different resolution classes, all cap out at 20–30fps. This is not a coincidence — it's the ISP/encoder pipeline these vendors build for *surveillance* (motion-triggered recall of license plates/faces), not fast-motion sports, and it holds even for their newest panoramic multi-sensor unit. UniFi Protect (Ubiquiti) and Amcrest product pages could not be scraped for hard fps numbers (JS-rendered storefronts — [store.ui.com](https://store.ui.com), [amcrest.com](https://amcrest.com) both returned no spec text on fetch), but Ubiquiti's own category page only advertises "4K" without an fps claim above 30 on any Protect model, consistent with the pattern above. 📚 From general product knowledge, UniFi G4/G5/G6 Protect cameras record at 24–30fps at rated resolution; I could not confirm this live today.

**Practical read:** if you want a genuine 60fps at 1080p or better from an IP/PoE camera, you are almost certainly leaving the consumer-security-camera category and moving into machine-vision (below) or a sports-specific unit that hides a machine-vision sensor behind a nicer enclosure (Veo, Pixellot, Hudl). Hikvision and Dahua technically make higher-fps panoramic/PTZ lines, but see the legal note below before considering either.

**Legal/sourcing note (important for an India-based team):** ✅ Hikvision is on the US Entity List, barred from US federal contracts and (per an FCC Secure Equipment Act action) from new US retail sale/import; the FCC opened a further investigation in March 2025; Canada ordered Hikvision to cease Canadian operations in June 2025 (upheld in court Sept 2025); **India banned Chinese surveillance equipment including Hikvision from government buildings, critical infrastructure, and public-sector projects in April 2026** ([en.wikipedia.org/wiki/Hikvision](https://en.wikipedia.org/wiki/Hikvision)). Dahua is under a similar FCC Interim Freeze Order (Nov 2022) barring new-equipment sale/import, an FCC investigation opened March 2025, and its former Lorex brand is under a Texas AG lawsuit (Feb 2026) over continuing Dahua ties ([en.wikipedia.org/wiki/Dahua_Technology](https://en.wikipedia.org/wiki/Dahua_Technology)). **If VolleyVerse ever sells into university/government-linked sports facilities in India, Hikvision/Dahua-family hardware is a real compliance risk, not just a brand preference** — verify against the actual April 2026 Indian notification text before specifying either brand for any public-sector venue.

### 1b. Machine-vision / industrial cameras — where real 60fps+global shutter actually lives, cheaply

✅ Confirmed via two independent Teledyne/FLIR pages and Mouser (Basler distributor):

| Model | Resolution | FPS | Shutter | Price | Source |
|---|---|---|---|---|---|
| FLIR/Teledyne BFS-PGE-04S2C-CS | 0.4MP (720×540) | 291fps std / 350fps Turbodrive | Global (Sony IMX287) | **$361** | ✅ [flir.com/products/blackfly-s-gige](https://www.flir.com/products/blackfly-s-gige/) |
| FLIR/Teledyne BFS-PGE-19S4C-C | 2MP | **60fps** | Global | **$655** | ✅ [teledynevisionsolutions.com/products/blackfly-s-gige](https://www.teledynevisionsolutions.com/products/blackfly-s-gige/) |
| FLIR/Teledyne BFS-PGE-23S3C-C | 2.3MP | 53fps | Global | $421 | ✅ same page |
| FLIR/Teledyne BFS-PGE-50S4C-C | 5MP | 24fps | Global | $740 | ✅ same page |
| FLIR/Teledyne BFS-PGE-244S8C-C | 24.5MP | 5fps | Global | $2,225 | ✅ same page |
| Basler ace 2 (Mouser #405-109485) | — | — | — | **$1,784.29** | ✅ [mouser.com](https://www.mouser.com) (Basler ace 2 search; +10% possible US tariff noted on listing) |
| Basler ace 2 (Mouser #405-109578) | — | — | — | **$2,035.72** | ✅ same |

Full Blackfly S GigE catalog spans **$361–$4,638** across 52 color/mono variants (per the fetched listing). This is the sweet spot for a genuine ball-tracking camera: true global shutter (no rolling-shutter skew on a fast ball), 60fps+ at resolutions plenty for a 21cm ball across 18m, and *cheaper per unit* than the Basler equivalents I could reach live. Allied Vision Alvium and The Imaging Source pages could not be fetched this session (blocked mid-session); 📚 from general knowledge both are directly comparable GigE/USB3 global-shutter lines in a similar $300–$1,500 band — treat as a plausible alternative sourcing path, not a confirmed price.

**Why/why not machine vision for VolleyVerse:** Genuine global shutter + high fps + far better low-light photon collection per pixel than a security-cam sensor (larger pixels — e.g., Sony IMX273 in the BFS-19S4C-C uses 3.45µm pixels vs. ~2.0–2.9µm typical in a security-cam ISP). The cost is real system-integration burden: **no PoE on most of these** (GigE Vision cameras need either a separate 12V supply or a PoE-injector/PoE-capable GigE card — a real BOM line item most teams forget), no built-in encoder/RTSP/NVR ecosystem (you're writing your own capture pipeline against the GigE Vision / Pylon / Spinnaker SDK), and a bare C/CS-mount body needs a lens purchased separately (add $50–$300+ depending on focal length/aperture). This is the right call for a *dedicated ball-tracking camera*, wrong call for "cheap venue-wide coverage camera" where a PoE security cam's turnkey NVR/RTSP ecosystem still wins on integration time even with the fps penalty.

### 1c. Action cameras — cannot do 8+ hour unattended capture, full stop

✅ Confirmed specs (DJI); model lineup confirmed live (GoPro, Insta360); exact current prices could not be scraped from JS-rendered storefronts (📚 recalled figures flagged below).

- **DJI Osmo Action 5 Pro** ✅ [dji.com/osmo-action-5-pro/specs](https://www.dji.com/osmo-action-5-pro/specs): battery life **240 minutes at 1080p/24fps** with stabilization on, Wi-Fi off — i.e., the *best case* is 4 hours, and that's at a resolution/fps you don't want for ball tracking. At its useful modes — 4K@100/120fps, 1080p@100/120/200/240fps — expect meaningfully less than 4 hours before battery depletion, well short of an 8-hour tournament day. 📚 Price recalled ~$349–399 at launch (not re-verified live).
- **GoPro HERO13 Black** ✅ confirmed still in the current lineup via [gopro.com/en/us/shop/cameras](https://www.gopro.com/en/us/shop/cameras) (Sept 2026 fetch), alongside HERO12 Black, MAX2, MAX, LIT HERO, HERO, and a new **MISSION 1 / MISSION 1 Pro / MISSION 1 Pro ILS** cinema-camera line launched May 2026 at **$599.99 / $699.99** ✅ [en.wikipedia.org/wiki/GoPro](https://en.wikipedia.org/wiki/GoPro). 📚 HERO13 Black price recalled ~$399.99 (not re-verified live). **Notable finding, not asked for but material to sourcing risk:** the same Wikipedia page reports GoPro in serious financial distress — 2024 net income −$432M, stock down from an $86 peak to $1.33 (June 2024), and Reuters reporting in June 2026 that GoPro was weighing "a sale of the company or a merger" (not completed as of the fetched content); in August 2026 YouTuber Markiplier bought an 8.5% stake, becoming the largest shareholder after founder Nick Woodman. **If you're betting an MVP on GoPro as a component supplier, company continuity is a real, current risk** — worth a line in any procurement risk register.
- **Insta360**: confirmed current lineup ✅ [en.wikipedia.org/wiki/Insta360](https://en.wikipedia.org/wiki/Insta360) — X5 (Apr 2025, larger 1/1.28" sensors, swappable lenses), X4 (Apr 2024, 8K30/5.7K60/4K100), Ace Pro (Leica co-engineered, 4K120, 1/1.3" sensor). No live prices obtained; 📚 recalled ballpark X4 ~$499, Ace Pro ~$429.

**The structural problem for all three, independent of battery:** none of them are PoE, none natively speak RTSP/NDI/SRT (GoPro/Insta360 have proprietary live-stream apps, not a standard NVR-compatible stream), and all use consumer-camcorder thermal designs that **throttle or shut down on continuous high-resolution/high-fps recording** — this is a widely reported GoPro HERO-line issue for years and is architecturally true of all small-body action cams (no fan, sealed waterproof body, battery physically touching the hot SoC). Even with a USB-power "always plugged in" hack, thermal shutdown risk remains because the failure mode is heat, not battery. **Verdict: fine for a volunteer's supplementary highlight-reel angle, not viable as a primary 8-hour-unattended sensor for VolleyVerse's core pipeline.**

### 1d. Sports-specific cameras — the pricing is deliberately hidden, and that itself is a finding

| Product | Confirmed to exist (Sept 2026)? | Pricing model | Source |
|---|---|---|---|
| Veo Cam 3 | ✅ Yes — primary product; "Veo Cam 2" no longer referenced | Subscription-gated; page literally renders "From {price}" placeholders — figure is dynamically inserted by region/currency and not visible on static fetch | ✅ [veo.com/en-us](https://www.veo.com/en-us) |
| Veo Go | ✅ Yes — "turn two iPhones into a match camera" | "Subscription {price}/month; free hardware with annual plans" | ✅ same |
| Hudl Focus / Focus Flex / Focus Point / Focus Indoor / Focus Outdoor | ✅ All five confirmed live, incl. explicit volleyball mention in supported sports | 🚫 No price anywhere on the page; page explicitly routes to "contact sales" | ✅ [hudl.com/en_gb/products/focus](https://www.hudl.com/en_gb/products/focus) |
| Pixellot Show S3 / Air / Air NXT / DoublePlay | ✅ All confirmed; 19 sports incl. volleyball | 🚫 Explicitly "Camera-as-a-Service" — Pixellot's own language is becoming "financial partners" with clubs, i.e., revenue-share/subscription, not a BOM line item at all in many deployments | ✅ [pixellot.tv](https://pixellot.tv/) |
| Spiideo (Portable SmartCam 3 + Spiideo EDGE) | ✅ Confirmed | 🚫 No pricing, resolution or fps published; "Talk to sales" only | ✅ [spiideo.com/products](https://spiideo.com/products/) |

**Read this as a real finding, not a gap in my research:** every sports-specific vendor in this category has converged on **hidden, quote-only, subscription-first pricing** — none publish a hardware BOM cost the way a security-camera or machine-vision vendor does. That's a deliberate go-to-market choice (bundle hardware cost into a recurring SaaS fee so the sticker shock never appears), and it tells you something about the category's actual margin structure: the hardware is not the expensive part, the *service* is. For VolleyVerse's competitive-positioning purposes, that also means there is real headroom to compete on transparent, unbundled pricing if that's a market gap worth exploiting.

📚 From general market reporting (not vendor-published, treat as rough color): Veo Cam hardware has historically been reported around $1,000–1,300 one-time plus a required annual "Team" subscription in the ~$1,000–1,500/yr range; Hudl Focus and Pixellot deployments are frequently reported in the low-to-mid **$2,000–5,000 install + $1,000–3,000+/yr** range depending on tier — none of this is vendor-confirmed and you should not quote it externally.

### 1e. Smartphone/tablet as capture device (Balltime-style)

No vendor page needed here — this is a reasoned assessment of what these apps structurally do: a phone/tablet on a tripod or clamp runs an app that either records locally (with on-device or later cloud ML) or streams. **Pros:** effectively $0 incremental hardware (BYO device), modern phone sensors + computational photography meaningfully outperform a budget security-camera sensor in dim light, most 2024+ phones do 4K60 or 1080p240 natively, built-in connectivity (Wi-Fi/cellular) means no network engineering. **Cons, and they're serious for an 8-hour tournament day:** battery/thermal limits are *at least as bad* as action cameras (thinner unventilated body, bright screen often left on), no PoE/always-on power without a cable physically taped down (trip hazard in a gym), on-device storage fills fast at high bitrate, **a phone left unattended on a tripod will get picked up** by someone checking notifications mid-match unless physically locked down, and placement/leveling is never consistent match-to-match, which breaks any calibration you tried to establish. This is the right answer for a **$0-hardware pilot with a single club testing whether VolleyVerse's software is worth paying for at all** — it is not a scalable, unattended production sensor.

### 1f. Panoramic/multi-sensor cameras

✅ Axis P3737-PLE confirmed: 4 independent sensor heads (varifocal 3.18–8.12mm each), 4×5MP @ 20fps/channel, 360° IR — [axis.com/products/axis-p3737-ple](https://www.axis.com/products/axis-p3737-ple). This class of camera (one head unit, multiple fixed sensors stitched or run independently) is architecturally exactly what Pixellot/Spiideo-style "single-unit whole-court" products use internally, but Axis's own security-market version tops out at 20fps/channel — same frame-rate ceiling problem as section 1a, just spread across more sensors. Hikvision's panoramic line could not be fetched this session (blocked mid-session) — treat as existing in the same 20–30fps class based on the pattern above, and subject to the same legal-sourcing caveats.

### 1g. KEY SPEC DISCUSSION

**Motion blur vs. shutter speed.** FIVB ball diameter is ~21cm (65–67cm circumference); elite men's serves are commonly cited up to ~130 km/h (36.1 m/s), club/school-level serves much slower (40–90 km/h). Motion-blur distance = velocity × exposure time:

| Shutter speed | Blur distance @ 130 km/h (36.1 m/s) | Blur as % of ball diameter |
|---|---|---|
| 1/250s | 14.4 cm | ~69% — ball is a smear, not a circle |
| 1/500s | 7.2 cm | ~34% — usable for coarse presence detection only |
| 1/1000s | 3.6 cm | ~17% — workable for centroid tracking |
| 1/2000s | 1.8 cm | ~9% — good, minimal shape distortion |
| 1/4000s | 0.9 cm | ~4% — excellent, but needs a LOT of light |

**Rolling vs. global shutter — a separate problem from blur.** Blur is about exposure *duration*; rolling-shutter distortion is about sensor *readout*, where each row of pixels is captured at a slightly different instant. A ball crossing the frame fast during a multi-millisecond row-by-row readout doesn't just blur — it **skews into an egg/oval shape or a diagonal streak**, which corrupts centroid estimation in a way a longer/shorter exposure can't fix. This is *the* reason machine-vision cameras (Blackfly S, Basler ace, both confirmed global-shutter above) are the correct choice for a dedicated ball-tracking camera, even though rolling-shutter security cameras remain fine for player-position tracking (players move at ~5–8 m/s max, an order of magnitude slower than a served ball).

**The lux problem.** A 1/1000–1/2000s shutter lets in 1/16–1/33 the light of a typical rolling security-cam exposure (~1/60–1/125s). Combine that with a small-pixel security sensor and you get exactly the noise problem the brief describes: not enough photons per pixel, forcing high sensor gain, which shows up as grain/noise that actively hurts a detector's ability to find ball edges — **this is a case where "more fps/faster shutter" directly fights "clean enough image for the model," and gym lighting quality becomes a hard external variable you don't control.** Typical unrenovated school/club gym lighting runs roughly 300–500 lux (uneven, with shadowed zones under galleries); a well-renovated LED gym can reach 750–1000+ lux; FIVB-broadcast-grade venues target 1000–1500+ lux at floor level specifically because of this exact problem. Machine-vision sensors with larger pixels (e.g., 3.45µm on the BFS-19S4C-C vs. ~2µm typical security-cam pixels) and the option to pair a fast (f/1.4–f/1.8) lens meaningfully outperform a security camera's fixed small-aperture "megapixel" lens here — this is the real, non-marketing reason to pay the machine-vision premium for the ball camera specifically.

**Flicker/banding at mains frequency — an India-specific detail worth flagging explicitly.** Indoor gyms are commonly lit by fluorescent tubes (magnetic ballast flickers at 2× mains frequency) or budget LED retrofits with cheap non-PWM-dimmed drivers (flicker at mains frequency or its low multiples). **India runs 50Hz mains**, so flicker-safe shutter speeds are multiples of 1/100s (1/100, 1/200, 1/400, 1/800…) — **not** the 1/60-family multiples (1/60, 1/120, 1/240…) that US-market camera defaults and most US-authored tutorials assume. This is a genuine, easy-to-miss trap for a US-influenced hardware stack deployed in Indian gyms: **the fast shutter speeds needed for blur control (1/1000, 1/2000) are not clean multiples of either 1/100 or 1/120**, so banding can still appear under bad lighting regardless of region; the only reliable fixes are (a) camera-level "flicker-free"/anti-banding modes (common on both security and machine-vision cameras — enable and set to 50Hz explicitly for India), or (b) verifying/upgrading venue lighting to high-frequency (>2kHz PWM or true DC) LED fixtures, which is a venue-side capex item VolleyVerse cannot control and must test on-site before committing to a shutter setting.

**Resolution needed for a 21cm ball across an 18m court.** If one camera's horizontal frame spans the court's 18m length (plus ideally the 3m free zones each end = 24m), pixel footprint = span/horizontal-resolution:

| Resolution | Pixel footprint (naive, 24m span) | Ball diameter in pixels |
|---|---|---|
| 1080p (1920px) | 1.25 cm/px | ~17 px |
| 4K (3840px) | 0.625 cm/px | ~34 px |

1080p is numerically fine in this idealized straight-on case. The catch: a real installation is rarely a perfectly perpendicular, undistorted view — it's an oblique angle from a sideline or corner, where far-court pixels cover meaningfully more real-world distance than near-court pixels (perspective foreshortening), and wide-FOV lenses needed to fit the whole court add distortion that further degrades edge resolution. In practice, expect effective far-corner resolution roughly half the naive number above — meaning 1080p (~8–9px ball at the far corner) is workable for detection but marginal for precise contact-point/line calls, while 4K (~17px at the far corner) gives real headroom. **This is why multi-camera layouts (each camera covering less court at a tighter, less-distorted FOV) usually beat one heroic wide-angle 1080p camera for tracking accuracy**, even before fps is considered.

**30 vs 50 vs 60 vs 120 fps — distance traveled between frames at a 36.1 m/s serve:**

| FPS | Time between frames | Ball travel between frames |
|---|---|---|
| 30 | 33.3 ms | **1.20 m** |
| 50 | 20.0 ms | 0.72 m |
| 60 | 16.7 ms | 0.60 m |
| 120 | 8.3 ms | 0.30 m |

At 30fps the ball can move over a meter between samples — more than a body-width — which is coarse for trajectory reconstruction and risks a critical contact frame being missed entirely during a fast exchange. **60fps roughly halves that gap for a manageable cost/complexity step up and is the realistic sweet spot for ball tracking**; 120fps+ meaningfully tightens contact-moment/line-call precision but doubles data rate, storage, and compute load, and is best reserved for a narrow-FOV camera dedicated to a specific zone (net, lines) rather than whole-court coverage — directly analogous to how Hawk-Eye-style officiating systems use dedicated high-speed cameras at specific zones, not one high-speed camera for the whole venue. Note 50Hz-native regions (India included) get a secondary practical benefit from 50fps/25fps-family rates: easier to align frame timing cleanly against local mains-frequency lighting flicker than 60fps-family rates are.

---

## 2. EDGE COMPUTE

### 2a. NVIDIA Jetson lineup — current, verified

✅ [nvidia.com/.../jetson-orin](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/), ✅ [nvidia.com/.../jetson-thor](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-thor/), ✅ [en.wikipedia.org/wiki/Nvidia_Jetson](https://en.wikipedia.org/wiki/Nvidia_Jetson):

| Module | TOPS (INT8) | RAM | Power | Devkit/module price |
|---|---|---|---|---|
| Orin Nano (4GB) | 17–20 | 4GB LPDDR5 | 7–25W | Module: **$199** (1,000-unit) |
| Orin Nano (8GB) | 33–40 | 8GB LPDDR5 | 7–25W | Module: **$299** (1,000-unit) |
| **Orin Nano Super Dev Kit** | 67 | 8GB LPDDR5 | 7–25W | **$249** (cut from $499 in Dec 2024) |
| Orin NX (8/16GB) | 67–117 | 8/16GB LPDDR5 | 10–40W | Module price not public; Arrow lists the 16GB module (900-13767-0000-000) **out of stock, no price shown** ✅ [arrow.com](https://www.arrow.com/en/products/900-13767-0000-000/nvidia) |
| AGX Orin (32/64GB) | 157–275 | 32/64GB LPDDR5 | 15–60W | Not public this session; 📚 devkit historically ~$1,999 at 2023 launch, subject to since-unconfirmed price cuts |
| **Jetson Thor T4000** | 1,200 TFLOPS (FP4) | 64GB | 40–70W | Not public |
| **Jetson Thor T5000** | 2,070 TFLOPS (FP4, sparse) | 128GB LPDDR5X, 273GB/s | 40–130W | **Devkit: $3,499** (Aug 2025) |

**Real-world integrator prices** (Mouser, Sept 2026 fetch — i.e., what you'd actually pay for a ready-to-run box, not a bare devkit): Seeed reComputer J3011 (8GB Orin Nano) **$813.75**; Asus PE1100N (8GB Orin Nano) **$1,249**; ADLINK DLAP-211 (8GB Orin Nano) **$1,461.40** ✅ [mouser.com](https://www.mouser.com). These fully-cased, fan-cooled, industrial-I/O integrator boxes cost 3–6× the bare $249 devkit — that gap is the real cost of "something you can bolt into an equipment closet and forget," not a markup to avoid.

**Product-line change worth flagging:** Jetson Thor ships as **T4000/T5000** modules (confirmed on NVIDIA's own page), not "T5000" alone as sometimes referenced — and NVIDIA's Orin page separately references newer "T3000 and T2000" Blackwell-family modules without public specs yet. If the brief's "T5000" assumption was based on older info, it's essentially correct but incomplete — there's a 4-tier Thor family forming, not a single SKU.

### 2b. Mini-PC with consumer GPU

✅ Newegg (Sept 2026 fetch): ASUS ROG NUC 14 (Ultra 7 155H, barebones) **$999.99**, with 16GB/512GB **$1,499.99**; Khadas Mind 1 (i7-1360P + RTX 4060 Ti eGPU dock) **$1,848** (down from $2,198); MXZ builds (Ryzen 7/5 + RTX 5060 Ti, 16GB DDR5, 1TB NVMe) **$1,449–$1,469**; various RTX 4060 small-form desktops **$1,044–$1,502** ([newegg.com](https://www.newegg.com), mini-PC/RTX-4060 search). Note "Intel NUC" is now **ASUS-branded** (Intel sold the NUC business to Asus in 2023) — the Newegg results literally show "ASUS ROG NUC," confirming this transition in the market today.

**Why a mini-PC + discrete GPU can beat Jetson on $/throughput:** a desktop RTX 4060/5060-class GPU delivers vastly more raw FP16/INT8 throughput than any Jetson module at a comparable or lower price point, and runs the exact same CUDA/TensorRT stack with none of Jetson's embedded-ARM software quirks — for a **fixed installation with wall power available (the normal case in a gym)**, $/throughput usually favors the mini-PC. The trade you're accepting: a discrete GPU alone draws 115–160W vs. Jetson's 7–40W total system power, there's no genuinely fanless/rugged embedded chassis option at this price, and physical size/heat output is much less friendly to a locked equipment cupboard with no HVAC — which matters in Indian gym facilities where equipment rooms are often poorly ventilated and can run hot for months at a time. For a **portable/battery-adjacent rig moved between venues**, Jetson's power envelope and embedded-grade thermal design still win regardless of the $/throughput gap.

### 2c. Intel NUC + OpenVINO, Coral, Hailo

- **"Intel NUC"** as a brand no longer exists in the way the brief assumes — see above, it's Asus now. **OpenVINO** is a separate thing: Intel's own inference-optimization SDK, still actively maintained, and it runs on Intel CPU/iGPU/discrete Arc GPUs generally (including the mini-PCs above), not tied to a discontinued NUC brand.
- **Hailo-8 M.2** ✅ [mouser.com](https://www.mouser.com) (Hailo-8 search): **$169** standard module (26 TOPS per Hailo's own page ✅ [hailo.ai/products/hailo-8](https://hailo.ai/products/hailo-8/)), **$191.25** starter kit, **Hailo-8L $85**. This is a genuinely cheap way to add real inference throughput to an otherwise-weak mini-PC or NUC-class board via M.2, and is worth serious evaluation against a bare Jetson for a fixed-camera-count install.
- **Coral TPU**: [coral.ai/products](https://coral.ai/products/) now 302-redirects to [developers.google.com/coral](https://developers.google.com/coral) ✅, and that landing page gives no clear current availability/discontinuation statement. Treat Coral as **status-uncertain** for new-design purchasing — don't build a 2027+ roadmap around it without directly confirming stock with Google/a distributor first.

### 2d. Realistic throughput — what a Jetson can actually do

✅ Two independent benchmark sources:
- **Ultralytics's own Jetson guide**, YOLO26n (their newest, smallest model), TensorRT, *inference-only* (excludes pre/post-processing): Orin Nano Super **~219–263 fps**; Orin NX 16GB **~242–286 fps**; AGX Orin 64GB **~382–435 fps** ✅ [docs.ultralytics.com/guides/nvidia-jetson](https://docs.ultralytics.com/guides/nvidia-jetson/).
- **NVIDIA's own MLPerf v3.1 numbers**, Retinanet (a heavier, more accurate detector, not YOLO): AGX Orin **148.71 samples/sec** single-stream; Orin NX **66.5 samples/sec** offline; Orin NX MaxQ **47.59 samples/sec** offline ✅ [developer.nvidia.com/embedded/jetson-benchmarks](https://developer.nvidia.com/embedded/jetson-benchmarks).

**Honest translation to multi-camera reality:** the small-model number (200–400+ fps) looks like it trivially covers several 60fps camera streams on paper. It doesn't, in practice, for three reasons the benchmark explicitly excludes: (1) pre/post-processing (resize, NMS, tracker association, multi-camera fusion) is not counted and typically adds a comparable amount of time to raw inference; (2) a production pipeline runs a bigger, more accurate model than the tiny "n" variant benchmarked, which costs several× the latency; (3) running N camera streams concurrently on one device means real contention for the same GPU/DLA engines, not N independent full-speed pipelines. **A defensible planning number is 1–2 camera streams of real-time 1080p60 YOLO-class detection+tracking per Orin NX-class module**, with the AGX Orin comfortably handling more (3–4), and the small Orin Nano Super realistically good for 1 stream plus light logic — not the 4–8 streams a naive read of the raw fps table might suggest. Multi-camera VolleyVerse installs (4+ cameras) should plan on either an AGX Orin/mini-PC-with-dGPU centrally, or one Jetson per 1–2 cameras rather than one Jetson for the whole rig.

### 2e. Cloud GPU alternative

✅ Confirmed on-demand US pricing via [instances.vantage.sh](https://instances.vantage.sh) (a third-party aggregator built on AWS's own published pricing API — cross-check against [aws.amazon.com/ec2/pricing](https://aws.amazon.com/ec2/pricing/on-demand/) directly before committing budget, since I could not get the raw AWS page to render a table):

| Instance | GPU | vCPU/RAM | $/hour |
|---|---|---|---|
| g4dn.xlarge | 1× T4 (16GB) | 4 / 16GB | **$0.526** |
| g6.xlarge | 1× L4 (24GB) | 4 / 16GB | **$0.805** |
| g5.xlarge | 1× A10G (24GB) | 4 / 16GB | **$1.006** |

**Cost per hour of match video processed** (worked example, stating assumptions plainly): if your model+pipeline runs at ~3× real-time on a T4 for a single camera (a plausible ratio for a lightweight detector doing a second pass, not live), 1 hour of footage costs ≈ 20 minutes of GPU time ≈ **$0.18** on g4dn.xlarge. If you instead need near-real-time (1×) throughput per stream on a heavier/more accurate model, 1 hour of footage costs close to **$0.53–$1.01** per camera-hour depending on instance class, and a 4-camera match multiplies this roughly linearly if processed sequentially on one instance (~$2.10–$4.00/match on g4dn) or requires 4 concurrent instances if you need it live (same total $ if same instance-hours, but real-time requires the parallel spend). **Cloud makes sense for after-the-fact batch analytics or peak-tournament burst capacity; it does not remove the need for *some* on-site compute if you want live, low-latency events, because round-trip network time to a cloud region adds materially to the latency budget (see §4).**

---

## 3. DEPTH / OTHER SENSORS

### 3a. Depth cameras — genuinely range-limited, confirmed prices

| Product | Price | Source |
|---|---|---|
| Luxonis OAK-D S2 | **$329** | ✅ [shop.luxonis.com/products/oak-d-s2](https://shop.luxonis.com/products/oak-d-s2) |
| Luxonis OAK4 S | **$949** | ✅ [shop.luxonis.com/collections/oak4](https://shop.luxonis.com/collections/oak4) |
| Luxonis OAK4 D | **$1,049** | ✅ same |
| Luxonis OAK4 D Pro | **$1,149** | ✅ same |
| Luxonis OAK4 CS (prototype) | **$1,099** | ✅ same |
| Stereolabs ZED 2i | **$669** | ✅ [robotshop.com](https://www.robotshop.com/products/stereolabs-zed-2i-stereo-camera) |
| Stereolabs ZED X | Not confirmed live this session (JS-rendered store; blocked on retry) | 📚 recalled ~$1,099–1,449 depending on sensor variant — verify directly |

**Why these are mostly irrelevant beyond ~10–20m for an 18m court (confirming the brief's framing, with the actual mechanism):** passive/active stereo depth error grows roughly with the square of distance divided by the stereo baseline × focal length. These units have baselines on the order of 6–12cm — fine for the 0.5–15m range they're designed and marketed for (robotics, AR, warehouse), but depth uncertainty at 18m+ balloons to a magnitude too coarse to place a 21cm ball's height usefully. It's not a firmware limitation the vendor could fix with an update — it's baseline geometry. **Intel RealSense — verified status:** ✅ RealSense was **not shut down**; Intel spun it out as an independent company on **July 11, 2025**, with a **$50M Series A** (Intel Capital, MediaTek Innovation Fund, and PE investors), explicitly continuing the existing product roadmap; it announced a **Jetson Thor integration partnership with NVIDIA in October 2025**, and **Dormakaba took a minority stake in November 2025** for biometric access control ([en.wikipedia.org/wiki/RealSense](https://en.wikipedia.org/wiki/RealSense)). RealSense is alive and shipping, just no longer an Intel-badged product line — good news for continuity if you were considering it, but its use case here remains the same short-range limitation as OAK/ZED.

### 3b. LiDAR — not actually range-limited, but wrong tool for a small fast ball

✅ Livox Mid-360: 0.1m minimum range, **40m at 10% reflectivity up to 70m at ≥80% reflectivity**, 360°×59° FOV ([livoxtech.com/mid-360](https://www.livoxtech.com/mid-360)) — this comfortably spans an 18m court; range is *not* the reason LiDAR is overkill here. The real reasons: (1) cost — 📚 Livox Mid-360 recalled ~$1,699, Ouster units historically quote-only and reported in the low-to-high five-figure USD range for higher-channel-count models (not vendor-confirmed today; Ouster's [ouster.com/products](https://www.ouster.com/products) page ✅ confirms OSDome/OS0/OS1/OS1 Max/OS2/REV8 exist but publishes no prices); (2) a 21cm sphere at 18m only intercepts a handful of LiDAR points at typical angular resolutions — far too sparse for reliable small-object classification versus a camera's dense pixel grid; (3) LiDAR returns no color/texture, so it can't disambiguate a ball from a hand, a kneepad, or a stray shoe the way even a cheap camera-based classifier can. **Verdict stands, for the right underlying reason: LiDAR is priced and shaped for whole-scene occupancy/robotics, not for pinpointing one small fast textured object.**

### 3c. Wearables — real rules uncertainty, be honest about it

✅ Catapult Sports confirmed (Vector S7, T6, T7 wearables + G5 goalkeeper monitor; clients include Real Madrid, Chelsea, the Brazil national team, Australian cricket, La Liga, NRL, XFL) — **no volleyball mention found anywhere in the fetched content** ([en.wikipedia.org/wiki/Catapult_Sports](https://en.wikipedia.org/wiki/Catapult_Sports)). Kinexon's own site could not be fetched this session (blocked mid-session); 📚 Kinexon (Munich) is well known generally for UWB real-time-location wearables in NBA/NFL/Bundesliga contexts. Blast Motion is primarily a swing-sensor company (baseball/golf/etc.), not an indoor-team-sport wearable play — lower relevance here.

**The rules question — I could not get a definitive answer and you should not treat this section as settled.** I made four separate attempts to fetch FIVB's official rulebook and NCAA volleyball rules for an explicit wearable/electronic-device provision; every attempt 404'd or was blocked. I did **not** find a citable rule either permitting or forbidding player-worn tracking sensors in official FIVB or NCAA competition. For contrast, ✅ football's governing body (IFAB) explicitly amended Law 4 in 2015 to permit Electronic Performance and Tracking Systems (EPTS) after years of banning them — a real, well-documented precedent that team-sport rulebooks *do* sometimes get formally rewritten for this. Whether FIVB/NCAA volleyball has an equivalent explicit provision is unverified here — **treat "can players legally wear a tracking sensor in an official match" as an open compliance question requiring direct confirmation from the current FIVB Rules of the Game and NCAA volleyball rulebook before any product plan depends on it.** Training/practice use, by contrast, is essentially unregulated and is where Catapult/Kinexon-style wearables are actually deployed today.

### 3d. Ball-embedded sensors — verified to exist in football, not found in volleyball

✅ Adidas Al Rihla (FIFA World Cup ball): an IMU suspended at the center of the bladder, developed jointly by **FIFA and Kinexon (Munich)**, feeding ball-movement data to VAR for offside decisions ([en.wikipedia.org/wiki/Al_Rihla](https://en.wikipedia.org/wiki/Al_Rihla)). **I found no evidence of a commercially available connected/sensor-embedded volleyball equivalent** — Molten's Wikipedia page could not be fetched to check directly, and no other source surfaced one. This reads as a genuine white space rather than a gap in my search: volleyball's ball-contact-detection problem (touches, double-contacts, net touches) is at least as valuable a signal as football's offside-line problem, and nobody has obviously shipped the equivalent. Worth flagging as a possible differentiated R&D bet, with the caveat that FIVB ball specifications and game-approval processes would need direct engagement before any embedded-sensor ball could be used in sanctioned play.

---

## 4. NETWORK / INSTALL / VENUE REALITY

### 4a. Network hardware — confirmed current prices

✅ Netgear ([netgear.com/business/wired/switches/poe](https://www.netgear.com/business/wired/switches/poe/), Sept 2026): GS308LP (8-port unmanaged PoE+) **$52.99**; GS108EP (8-port smart PoE+, 62W budget) **$119.99**; GS108EPP (8-port smart PoE+, 123W) **$159.99**; GS116EP (16-port smart PoE+, 180W) **$279.99**; GS116EPP (16-port smart PoE+, 231W) **$319.99**; GS752TXUP (48-port managed) **$1,259.99**. **Sizing check that actually matters:** GigE Vision machine-vision cameras (Blackfly S/Basler ace) are generally **not PoE-native** — confirm each specific model before assuming a PoE switch alone powers them; you may need PoE-to-GigE injectors or separate 12V supplies, a genuine extra BOM line.

### 4b. NDI vs RTSP vs SRT

✅ NDI (full): **"1080 60P video yields a data rate up to 150 Mbps per stream"** ([docs.ndi.video bandwidth whitepaper](https://docs.ndi.video/all/getting-started/white-paper/bandwidth.md)) — this is a LAN-only, near-lossless production protocol, excellent for zero-latency in-venue switching, unworkable over a normal internet uplink and expensive in storage if recorded raw (see 4c). **RTSP** (well-established, not separately re-verified today): a pull-based streaming protocol using standard H.264/H.265 compression, universally supported by IP cameras and NVRs, typically 200ms–2s latency depending on buffering, the default choice for camera→edge-box transport on a LAN. **SRT** is a transport protocol (not a codec) purpose-built for reliable low-latency streaming over unreliable/public networks — the right choice specifically for the leg where compressed video leaves the venue over a normal broadband/Wi-Fi uplink to a cloud service, since it recovers from packet loss without NDI's bandwidth or RTSP/TCP's re-buffering stalls. **For the "one network port, no IT staff" gym**: use RTSP (or NDI HX, the compressed variant, if using NDI-native gear) camera→edge-box on the LAN, and SRT for anything that has to leave the building.

### 4c. Bitrate and storage per match-hour

Using the fetched NDI figure as the top anchor and standard, well-established H.264/H.265 encoding ratios:

| Format | Typical bitrate | Storage per camera-hour |
|---|---|---|
| NDI full, 1080p60 | ✅ ~150 Mbps | ~67.5 GB |
| 1080p60, H.264 (security-cam typical) | ~8–12 Mbps | ~3.6–5.4 GB |
| 1080p60, H.265/HEVC | ~4–6 Mbps | ~1.8–2.7 GB |
| 4K30, H.264 | ~20–30 Mbps | ~9–13.5 GB |
| 4K30, H.265/HEVC | ~10–15 Mbps | ~4.5–6.75 GB |

**Worked example, 4 cameras, 8-hour tournament day, 1080p60 H.265:** 4 × 2.5GB/hr × 8hr ≈ **80GB/day** — trivially manageable on one consumer NVMe SSD. The same 4 cameras recorded as raw NDI for 8 hours ≈ **2.16 TB/day** — impractical to store without either aggressive on-the-fly transcoding or discarding raw feeds right after live processing. **This is a real architectural decision, not a footnote: decide up front whether raw footage is ever retained for re-training/dispute-review, because the storage cost difference between "process live, keep only H.265 proxies" and "keep NDI-quality raw" is roughly 25×.**

### 4d. Mounting, FOV, and gym geometry

✅ Court/ceiling facts: indoor court **9m × 18m**, free zone **minimum 3m** on all sides, ceiling clearance **minimum 7m (23ft)**, **recommended 8m (26.2ft)** ([en.wikipedia.org/wiki/Volleyball_court](https://en.wikipedia.org/wiki/Volleyball_court)). 📚 (not re-verified live this session, standard volleyball-officiating knowledge) elite FIVB World/Olympic-level competition actually requires a much taller ~12.5m clearance — most school/club gyms sit at or barely above the 7–9m minimum, which genuinely constrains your options.

**FOV math:** to frame the full court + free zone (24m) in one shot, required horizontal FOV half-angle = atan(12m / mounting-distance). A camera mounted only a few meters behind the baseline (common when a gym's free zone runs right up against a wall) needs a fisheye-class FOV to fit the whole court, introducing heavy distortion that must be corrected in software and that degrades effective resolution at the frame edges — exactly where a far-court ball needs the *most* resolution, not the least. A camera mounted higher and further back (ceiling truss at mid-court sideline height, or a raised platform beyond the free zone) needs roughly a 90–100° lens for single-camera whole-court coverage — this is the actual geometry Veo/Pixellot-style single-unit products are built around. **Practical mounting hierarchy for a school/club gym:** ceiling truss (best — out of ball/player reach, stable, but needs a lift/ladder and often facility permission to rig anything to the structure) > elevated side-wall bracket beyond the free zone (good compromise, still needs a permanent mounting point) > tripod at the end of the free zone (fastest to set up for a portable rig, but occupies floor space in the free zone itself and is vulnerable to a stray ball or a bumped tripod mid-match).

### 4e. Calibration

Multi-camera systems need both **intrinsic** calibration (lens distortion/focal length — done once per camera/lens, stable unless focus/zoom changes) and **extrinsic** calibration (each camera's position/orientation in a shared court coordinate frame — must be redone every time a camera physically moves, even by a few millimeters, since that error compounds at 15–20m range). A portable rig **can** self-calibrate reasonably well from the court's own painted lines (attack line, sidelines, center line are standard, high-contrast, known geometry) via a homography — this is standard, proven practice in the Veo/Pixellot/Hawk-Eye family. The catch: a single-camera homography only solves the **ground plane** — it cannot recover ball **height** on its own. Getting real ball height needs either multiple overlapping camera views (proper triangulation) or a physics-based fallback (assume ballistic flight, fit a parabola to the 2D pixel track, use known ball size for a rough monocular depth-from-size estimate) — the latter is fragile at long range/oblique angles and is a genuine, unsolved-by-hardware research problem, not a calibration-routine problem. **Recommended cadence:** full extrinsic recalibration at every physical setup (every venue for a touring rig); an automated line-based sanity check at the start of every match (seconds to a couple of minutes); continuous or periodic drift monitoring for multi-hour tournament days, since thermal drift, HVAC vibration, or someone bumping a tripod can silently degrade accuracy mid-event without an obvious failure signal.

### 4f. Latency budget — realistic per-stage numbers

| Stage | Realistic range | Driver |
|---|---|---|
| Capture (exposure + sensor readout) | 5–20ms | Rolling-shutter readout can add close to a full frame time; global shutter is faster/more deterministic |
| Encode | 20–80ms | Hardware encoder buffer depth; low-latency (few/no B-frame) profiles keep this near 1–2 frame times |
| Network transport (LAN, single hop) | 1–5ms (can spike much higher) | A consumer unmanaged switch or a single congested port — exactly the "one network port, no IT staff" scenario — introduces jitter well beyond this on a bad day |
| Decode + preprocessing at edge | 5–15ms/frame | Assuming compute isn't already the bottleneck (see §2d) |
| Inference (detection+tracking+fusion) | ✅ ~4–5ms raw (per Ultralytics YOLO26n benchmark above) up to 30–100ms+ in a real multi-model, multi-camera pipeline | Real pipelines run bigger models than the benchmarked "n" variant and add tracking/re-ID/triangulation on top |
| Event/rule logic | 1–20ms typical; **100ms–several seconds for ambiguous calls** | Net touches, boundary lines, double-contacts may need multi-frame/multi-camera buffering, or explicit human review |
| Delivery to web UI | 50–300ms | Dominated by backend/websocket round-trip + client render, not usually a hard constraint |

**Realistic end-to-end total:** roughly **150–500ms for a fully automated event on-site edge compute**, likely **1–3+ seconds if routed through cloud GPU processing** (network round-trip + queueing on top of the same stages), and **multiple seconds to tens of seconds whenever a human-in-the-loop confirmation step is inserted** — which, for an early-stage system's reliability, should be assumed as the common case for anything scoring-critical, not the exception.

### 4g. Power, heat, theft/damage, maintenance

PoE simplifies camera power (no separate brick) but the **PoE switch's total power budget must be checked against camera class** — the fetched Netgear switches cap out at 62–231W depending on model, a real constraint once you add several PoE+ cameras. Non-PoE machine-vision cameras need their own supply — an easy-to-forget line item. Gyms are frequently **not climate-controlled**, especially in India where equipment rooms/lofts often have minimal ventilation and can run hot for months — a fanless embedded box (Jetson-class) tolerates this far better than a bare gaming-GPU mini-PC, which risks thermal throttling in a hot closet during peak summer. Ceiling-mounted cameras are relatively theft-resistant but vulnerable to a stray ball or errant serve if mounted low/side-wall; the **compute box is the more attractive theft target** and belongs in a lockable enclosure, not sitting exposed on a shelf. **"Who maintains it" has an honest answer: not the venue.** A school gym/club with no IT staff cannot be relied on to notice a dead camera, a full disk, or an overheating box — VolleyVerse itself must provide remote health monitoring (camera-online heartbeat, free disk space, temperature) plus a simple, labeled physical runbook a volunteer can follow (which cable, which switch port, how to power-cycle) as an actual, ongoing *operational* cost, not a one-time hardware line.

### 4h. Portable vs. permanent

**Permanent** (a club's home gym): calibration stays valid indefinitely once set, cabling can be properly routed/conduit-protected, justified by high weekly utilization — but the equipment is exposed to the venue's environment (temperature swings, dust, unsupervised-hours vandalism risk) 24/7 and needs the venue's buy-in for a capital fixture. **Portable** (tournament organizers touring venues): must recalibrate at every stop — both a time cost and an operational risk if a volunteer sets height/angle wrong — favoring a small number of pre-calibrated, fixed-rig units (closer to the Veo/Pixellot single-or-dual-sensor model) over a from-scratch multi-camera machine-vision array that needs real calibration expertise on-site every single time. **This tension should directly shape the MVP call**: for the "school gym, bad lighting, one network port, no IT staff, volunteer setup" target customer described in this brief, the earliest viable product should bias hard toward the *simplest* rig that still produces useful output — even at some accuracy cost — because that customer genuinely cannot support a complex calibrated multi-camera array without dedicated technical support showing up in person.

---

## 5. COST TABLES (BOM approximations, USD; INR at an approximate **₹88/USD** — this rate is a ballpark estimate, not a live FX fetch; confirm the current rate before budgeting, and separately budget for Indian customs duty/GST if importing hardware directly rather than buying through an India-based distributor, since imported cameras/compute typically land at a meaningful premium over the USD sticker price)

**Legend for "expected accuracy ceiling":** this describes what the *hardware* physically limits, independent of model quality — per the brief's explicit instruction that hardware doesn't give accuracy, the model does, but hardware sets a floor/ceiling the model cannot out-train its way past.

### A) 1 fixed camera

| Dimension | Detail |
|---|---|
| Camera type | Single machine-vision global-shutter camera (e.g., Blackfly S BFS-PGE-19S4C-C class) or, budget version, one 4K PoE security camera |
| Resolution / FPS | 2MP@60fps (machine-vision) or 4K@20–25fps (security-cam budget path) |
| Placement | Elevated end-wall or ceiling-truss mount, ~90–100° FOV lens to fit court+free zone |
| Shutter | Global (machine-vision path only — security-cam path is rolling) |
| Lighting needs | Needs ≥500 lux even lit gym; fast-shutter path needs brighter/more even lighting or will show sensor noise |
| Network | Single PoE run to a switch; no multi-camera sync needed |
| Edge vs cloud | Small edge box (Jetson Orin Nano-class, ~$249–800) sufficient for one stream |
| Storage | ~2–5GB/hour (H.265) |
| Calibration | One-time intrinsic + one-time extrinsic (or per-move for portable) |
| Latency | ~150–400ms automated |
| **Accuracy ceiling** | **No 3D ball height ever recoverable from one camera** — best case is a physics-fit estimate, an inherent geometric limit no model improves past. One camera also cannot resolve occlusion (a player blocking the ball from view) at all. |
| Approx. cost | Camera+lens $650–1,000 + edge box $250–800 + mounting/cabling $100–300 ≈ **$1,000–2,100** (₹88K–185K) |
| Install complexity | Low–moderate (one mount point, one calibration) |
| Maintenance | Low; single point of failure |

### B) 2 cameras

| Dimension | Detail |
|---|---|
| Camera type | 2× machine-vision global-shutter, opposite corners or opposite sidelines |
| Resolution / FPS | 2MP@60fps each |
| Placement | Opposite elevated corners for baseline stereo-like coverage, or one per half-court |
| Shutter | Global |
| Lighting needs | Same as (A), now must be consistent across both fields of view |
| Network | 2× PoE-or-injector runs to one switch; needs frame-sync consideration for triangulation |
| Edge vs cloud | One AGX Orin-class box or one Orin NX per camera |
| Storage | ~4–10GB/hour total |
| Calibration | Extrinsic calibration between the two cameras now required (shared court coordinate frame) — meaningfully harder than (A) |
| Latency | ~200–450ms |
| **Accuracy ceiling** | With 2 overlapping views, **basic triangulated ball height becomes possible** for the first time — but only in the zone both cameras see unoccluded; single-camera-only geometric limits still apply outside overlap |
| Approx. cost | 2× camera/lens ~$1,300–2,000 + edge compute $800–1,460 + network/mounting $200–500 ≈ **$2,300–3,960** (₹202K–348K) |
| Install complexity | Moderate — two-camera extrinsic calibration is a real added step |
| Maintenance | Low–moderate |

### C) 4+ cameras

| Dimension | Detail |
|---|---|
| Camera type | 4× machine-vision global-shutter, one per court quadrant/corner |
| Resolution / FPS | 2MP@60fps each (or mix: 2 ball-tracking @60fps + 2 player-tracking @30fps security-cam to save cost) |
| Placement | Four elevated corners/trusses, overlapping FOVs across the whole court+free zone |
| Shutter | Global (ball cameras), rolling acceptable (player-only cameras) |
| Lighting needs | Even illumination across all four FOVs — hardest lighting requirement so far, shadowed gym corners now matter |
| Network | 4-port+ PoE switch (Netgear GS116EP-class, $279.99 confirmed above), real cable-run planning |
| Edge vs cloud | Realistically 2× Orin NX/AGX Orin-class boxes (per §2d throughput guidance — one Jetson per 1–2 streams), or one mini-PC+RTX4060 class box |
| Storage | ~8–20GB/hour total |
| Calibration | Full multi-camera extrinsic calibration (4-way), most failure-prone step in the whole system |
| Latency | ~250–550ms |
| **Accuracy ceiling** | Full-court triangulated 3D ball trajectory becomes genuinely feasible with good overlap; remaining ceiling is now dominated by calibration drift and compute-throughput contention (§2d), not camera count |
| Approx. cost | 4× camera/lens ~$2,600–4,000 + 2× edge compute ~$1,600–2,920 + switch $280 + cabling/mounting $400–800 ≈ **$4,900–8,000** (₹431K–704K) |
| Install complexity | High — this is where a volunteer setup genuinely struggles without a guided/auto-calibration workflow |
| Maintenance | Moderate — 4 potential points of failure, needs remote health monitoring |

### D) Multi-camera + edge computer (dedicated, higher-spec)

| Dimension | Detail |
|---|---|
| Camera type | 4–6× machine-vision global-shutter |
| Resolution / FPS | 2–5MP@60fps |
| Placement | As (C), plus one dedicated high-fps narrow-FOV camera at the net for contact/line calls |
| Shutter | Global throughout |
| Lighting needs | Same as (C) |
| Network | Managed PoE+ switch, 16-port class ($279.99–$319.99 confirmed) |
| Edge vs cloud | One AGX Orin devkit (📚 ~$1,999, unconfirmed live) or one mini-PC+RTX4060/5060 ($1,000–2,500 confirmed above) as central compute, sized per §2d guidance |
| Storage | 1–2TB local NVMe recommended for an 8-hour tournament day at H.265 |
| Calibration | Full multi-camera + dedicated net-camera calibration |
| Latency | ~250–500ms |
| **Accuracy ceiling** | Compute throughput (not camera count) becomes the binding constraint — if inference can't keep pace with 6 streams at 60fps, frames queue/drop and event timing degrades regardless of model quality (§2d) |
| Approx. cost | Cameras ~$4,000–7,000 + edge compute $1,000–2,500 + switch $300–320 + storage/cabling $500–1,000 ≈ **$5,800–10,800** (₹510K–950K) |
| Install complexity | High |
| Maintenance | Moderate–high; genuinely needs a maintenance contract or remote-support plan, not volunteer-only |

### E) Camera + depth sensor

| Dimension | Detail |
|---|---|
| Camera type | 1–2× RGB (as A/B) + 1× stereo depth camera (OAK-D S2 $329, OAK4 D $1,049, or ZED 2i $669 — all confirmed above) |
| Resolution / FPS | RGB as above; depth sensor typically 15–30fps for depth stream, lower than the RGB ball-tracking need |
| Placement | Depth sensor near-court (≤5–8m from action) — **its useful range does not reach the far side of an 18m court**, so it is only meaningful for a localized zone (e.g., serve contact, near-net play), not whole-court coverage |
| Shutter | Depth sensors' RGB channel is typically rolling shutter (check specific model) |
| Lighting needs | Active-IR depth (structured light/ToF-based units) can be disrupted by strong gym floodlighting or sunlight through windows |
| Network | USB3 (most of these are USB, not PoE) — short cable runs only, a real placement constraint |
| Edge vs cloud | Needs a directly-attached host (mini-PC or Jetson with USB3), not a remote network camera |
| Storage | Depth streams add meaningfully to storage if recorded raw; usually processed live and discarded |
| Calibration | Depth sensor needs its own extrinsic registration into the shared court frame, on top of RGB calibration |
| Latency | ~200–450ms |
| **Accuracy ceiling** | **Hard range ceiling of roughly 10–20m before depth error becomes too coarse to use** (baseline-geometry limit, not fixable by better firmware/software) — genuinely useful only for a sub-zone of the court, never for whole-18m-court depth |
| Approx. cost | RGB camera(s) $650–2,000 + depth sensor $329–1,149 + host compute $250–800 ≈ **$1,230–3,950** (₹108K–348K) |
| Install complexity | Moderate — USB cable-length limits complicate placement versus PoE's flexibility |
| Maintenance | Low–moderate |

### F) Camera + scoreboard integration

**The honest reality check requested:** ✅ I could not fetch a public, self-serve Daktronics DataXchange page, a Colorado Time Systems API page, or a Bodet network-integration page this session — all attempts 404'd or were blocked. This is itself informative: **scoreboard data-feed integration for small venues is not a documented, self-serve API you can sign up for** the way a payment gateway is. 📚 From general industry knowledge, Daktronics and similar vendors do have data-exchange products, but these are typically sold/configured as part of a broader Daktronics/venue contract relationship, aimed at large stadiums with an existing Daktronics install and IT support — not something a school gym with a 15-year-old Daktronics or generic scoreboard can self-enable. **Realistic conclusion: for the actual target venues in this brief (school gyms, club halls), OCR on a camera pointed at the physical scoreboard display is more realistic than a data-feed integration** — most small-gym scoreboards are simple wired-remote-controlled 7-segment/LED displays with no network port at all, so a data feed literally doesn't exist to integrate with, whereas OCR just needs the scoreboard in any camera's FOV (or a small dedicated one) and a digit-recognition model, which is a solved, low-risk CV problem given consistent digit fonts/positions.

| Dimension | Detail |
|---|---|
| Camera type | 1× dedicated low-cost fixed camera (even a basic security PoE cam is fine — this is a static-scene OCR task, not fast-motion tracking) pointed at the physical scoreboard, PLUS the main court camera(s) from A–D |
| Resolution / FPS | 1080p@10–15fps is plenty for digit OCR (score changes are not fast-motion) |
| Placement | Direct line-of-sight to scoreboard face, avoiding glare |
| Shutter | Irrelevant for this camera — static scene |
| Lighting needs | Avoid glare/reflection off the scoreboard's own LEDs — angle matters more than lux here |
| Network | Same PoE run as other cameras, can share the switch |
| Edge vs cloud | OCR model is lightweight, runs easily on the same edge box as ball-tracking |
| Storage | Negligible additional (low fps, small ROI crop) |
| Calibration | One-time framing/crop-region setup; needs re-check if scoreboard or camera moves |
| Latency | Sub-second is easily achievable for a static-digit OCR task |
| **Accuracy ceiling** | Bounded mainly by scoreboard legibility (LED brightness/contrast, viewing angle, glare) rather than by model quality — a good OCR model still fails on a glare-washed-out or partially-occluded scoreboard, a hardware/placement problem, not a software one |
| Approx. cost | +$50–150 for a dedicated cheap scoreboard-facing camera (or $0 if reusing an existing camera's FOV) on top of A–D's cost |
| Install complexity | Low — the easiest incremental addition in this whole list |
| Maintenance | Low |

### G) Camera + additional sensors (wearables/IMU/ball sensor)

| Dimension | Detail |
|---|---|
| Camera type | As C/D, plus player wearables (Catapult/Kinexon-class, 🚫 pricing quote-only, not public) |
| Resolution / FPS | Camera side unchanged; wearables sample IMU/UWB typically at tens–hundreds of Hz, far denser temporally than any camera fps |
| Placement | Wearables worn under jersey/on a vest — **rules status for official FIVB/NCAA competition is unverified, see §3c** — safe to deploy for training/practice today, not confirmed safe for official matches |
| Shutter | N/A for wearables |
| Lighting needs | N/A for wearables (a real advantage — IMU/UWB tracking is lighting-independent, unlike every camera-based approach above) |
| Network | Wearables typically use a separate proprietary RF/UWB base-station system, not your camera PoE network — a second, parallel infrastructure to install and maintain |
| Edge vs cloud | Vendor-proprietary backend in most wearable systems (Catapult/Kinexon), limited raw-data access — a real integration risk for a third party like VolleyVerse |
| Storage | Wearable data is tiny (numeric time-series) compared to video — a non-issue |
| Calibration | Wearable base-stations need their own venue-specific RF calibration, independent of camera calibration |
| Latency | Wearable positional data is often lower-latency than camera-based tracking, but fusing it with camera-derived events adds its own sync-latency problem |
| **Accuracy ceiling** | Wearables solve player position/load extremely well (this is their designed purpose) but say **nothing about the ball** — you still need the camera system fully working for ball events; wearables are additive to, never a substitute for, the camera pipeline |
| Approx. cost | Camera system as C/D (~$4,900–10,800) + wearables **🚫 quote-only, not public** — budget for a per-player-per-season enterprise SaaS-style cost, not a one-time hardware line, and confirm official-competition legality before committing |
| Install complexity | High — two independent systems (camera + RF wearable infrastructure) to install, calibrate, and keep in sync |
| Maintenance | High — battery-charging logistics for every player's wearable every session is a real, recurring operational burden the brief's "no IT staff" venue cannot absorb without VolleyVerse actively managing it |

---

## What still needs a direct vendor quote before any of this becomes a real budget

Genuinely not public anywhere I could reach, confirmed by explicit "contact sales" language or blocked/absent pricing on the vendor's own site: **Veo Cam 3 hardware price, Hudl Focus/Focus Flex, Pixellot (any model), Spiideo (any model), Kinexon, Catapult, Ouster LiDAR, ZED X, current Jetson AGX Orin devkit price, and any Daktronics/Colorado Time Systems/Bodet data-feed product.** Treat every 📚-flagged figure in this report as a planning placeholder, not a quote, and re-verify GoPro/DJI/Insta360/Basler exact current prices directly before purchasing — this session's tooling could not render their JavaScript storefronts to confirm live numbers despite the products themselves being confirmed to exist and to still be in each vendor's current lineup.