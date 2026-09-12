# Commercial Landscape — Who Is Already Doing This (STEP 3)

**Methodology:** WebSearch was blocked for this session (HTTP 403 on every
query). All findings come from ~45 direct WebFetches of vendor domains plus
Wikipedia. Markers: **[VERIFIED]** = confirmed by fetching the vendor's own
current page · **[CLAIM]** = vendor marketing language, not independently
testable · **[UNVERIFIED]** = could not confirm this session.

---

## PART 1 — Volleyball-specific products

### DataVolley 4 — Data Project (now part of Genius Sports)
- URL: https://www.dataproject.com/Products/EU/it/Volleyball/DataVolley4
- **[VERIFIED]** Data Project "is part of Genius Sports"; founded 1982; footer
  entity Genius Sports Italy Srl.
- **Captures:** full scouting code (serve/reception/set/attack/block/dig/error)
  synced to video.
- **Hardware:** laptop only. Win 8.1+, USB dongle licence.
- **AI/CV: NONE.** **[VERIFIED]** explicitly keyboard code-entry — "watch the
  action and write codes with the keyboard". Shortcut "automatisms" help a human
  keep pace; there is no detection.
- **Live/post:** live scouting; post-match video sync.
- **Human:** 100%. **This is the manual baseline VolleyVerse is replacing.**
- **Pricing [VERIFIED]:** Professional €799/yr; Lite €299/yr.
- **Learn:** DataVolley's *code schema* (fundamental / effect / zone) is the de
  facto taxonomy buyers expect. VolleyVerse's AI output should map onto it
  rather than invent a new vocabulary.

### Click&Scout — Data Project
- URL: https://www.dataproject.com/Products/EU/it/Volleyball/ClickAndScout
- Tablet touch input, iPad 5th-gen+/Android 7"+. **AI/CV: NONE [VERIFIED]** —
  "a touch, directly on the virtual playing area".
- Live only; codes cannot be added after the match (must export to DataVolley).
- **Pricing [VERIFIED]: €49/yr.** Free trial capped at 1 set / 200 codes.
- **Learn:** this — plus SoloStats/VBStats — is the real competitive set for
  VolleyVerse's *current* manual product. €49/yr is a very low willingness-to-pay
  anchor. The wedge is removing the human, not undercutting price.

### DataVideo / Video Sharing 4 — Data Project
- Video import/sync to codes, clip-making, in-venue LAN streaming. **No AI
  found [VERIFIED].** Pricing not public. Confirms the whole Data Project suite
  is *synchronisation tooling around human-entered data* — a clean gap for an AI
  layer underneath it.

### VideoCheck — Data Project / Genius Sports (volleyball challenge system)
- URL: https://www.dataproject.com/Products/EU/it/Volleyball/VideoCheck ·
  support contact `videocheck@geniussports.com` **[VERIFIED]**
- **Hardware [VERIFIED]: up to 23 cameras per venue, up to 200fps**, WiFi
  central unit, controlled from tablet/PC.
- Reviews ball in/out, service faults, net contact, four-touch, block touches,
  pancakes. **Not AI** — multi-angle *human* review.
- **Learn:** Genius Sports owns **both** volleyball's manual scouting standard
  **and** its official video-review rigs. The most obvious adjacent incumbent
  already has cameras on pro courts. Also: 23 cameras at 200fps is what
  "officiating-grade certainty about a net touch" actually costs.

### VolleyStation / VolleyStation Pro
- URL: https://volleystation.com — **[UNVERIFIED]**. Pure client-rendered SPA;
  WebFetch returned only generic metadata ("From grassroots junior tournaments
  to Olympic Games…"). No product detail, hardware, AI claim or pricing could be
  extracted. **Do not make competitive claims about VolleyStation without a
  JS-capable re-check.** Generally understood in-market as a live
  scoreboard/scouting/streaming suite competing with DataVolley — unconfirmed.

### Balltime → **Hudl Assist AI (Volleyball)** — the single most important finding
- **[VERIFIED]** `balltime.com` now **302-redirects** to
  https://www.hudl.com/products/assist/volleyball/ai — Balltime has been
  absorbed into Hudl and no longer operates independently. *(Re-verified
  independently: the 302 and its target were confirmed twice.)*
- Balltime's original identity **[VERIFIED via Crunchbase]:** Israeli startup,
  Tel Aviv; founders Dan Banon and Tom Raz; pre-seed from Volo Ventures; the
  original claim was stats/highlights **within ~1 hour** of upload — i.e. even
  its own pre-acquisition pitch was **post-match, not live**.
- **What Hudl claims [CLAIM]:** "every player touch and result" — serve,
  reception, set, attack, dig, outcome, zone, rotation; player + rotational
  stats, ball-trajectory overlays, AI performance summaries; "no coding or
  tagging".
- **Live/post [VERIFIED]:** **post-match.** Users upload games and the system
  then processes them. No real-time claim anywhere.
- **Rollout stage [VERIFIED — the key datum]:** **club** volleyball only (match
  + practice). **College: practice only** — college *match* breakdowns still go
  through **Volleymetrics** (human). **High school: coming 2026** (waitlist).
- **Human-in-loop:** **[UNVERIFIED]** — the page does not say whether a human QA
  pass happens. The silence is itself notable.
- **Pricing:** not public. Demo-gated.
- **Learn — read this carefully.** Hudl is the best-resourced, most obviously
  incentivised company to build exactly VolleyVerse's vision. They *bought* an
  AI team for it. Two-plus years on they ship only a **post-match, club-tier**
  feature and keep **humans** on the higher-stakes tiers. That is strong evidence
  the live/high-accuracy version is genuinely hard — not merely unattempted.

### Volleymetrics (Hudl)
- **[VERIFIED, direct quote]:** "Within 12 hours our professional **analysts**
  track every touch on both sides of the net." Human-coding-as-a-service, 12h SLA.
- **Learn:** the market *already pays humans* for this. An AI system doesn't
  have to be instant to win — it has to beat 12 hours and/or beat live-manual
  accuracy/effort. This is also the cost/quality benchmark to price against.

### SoloStats Live / SoloStats123, VBStats
- SoloStats **[VERIFIED]**: manual button/tap scorekeeping, free tier + paid
  WebReports. No AI. VBStats (vbstats.app) **[thin/UNVERIFIED]**: "record live
  matches, analyze your team, share reports"; framing mirrors the manual tools.
- **Learn:** the "click buttons live" category is a commodity with a near-zero
  price ceiling. Automation is the only durable differentiator.

### Sideout Stats, iVolley — **[UNVERIFIED]**, domains did not resolve. Could not
confirm these still exist.

### FIVB Video Challenge / Hawk-Eye (Sony)
- **[VERIFIED]** hawkeyeinnovations.com lists **FIVB** among officiating
  partners; volleyball sits under their **"SMART Replay"** line.
- **Not verified:** a dedicated volleyball spec page (camera count, scope,
  competitions/years) — direct URL guesses 404'd, and Hawk-Eye's own recent
  news/insights pages list **zero** volleyball articles. One automated extraction
  surfaced the term **"VolleyTrack"** but the URL 404'd with no corroborating
  press — **treat "VolleyTrack" as unconfirmed.**
- **Learn:** Hawk-Eye is FIVB's incumbent for officiating-grade review but
  appears to treat volleyball as a minor/legacy vertical. Thin marketing and no
  recent investment = a soft spot, but also a signal that officiating-grade
  volleyball CV is low-margin even for the best-equipped player.

### FIVB VIS (Volleyball Information System) — **[UNVERIFIED]**, no working page
found. Commonly understood as a competition-management/results database, not a
CV system — unconfirmed.

---

## PART 2 — Automated camera / broadcast production

### Spiideo — https://spiideo.com **[VERIFIED]**
- **Spiideo Play** (automated live production, "no camera operator"),
  **Spiideo Perform** (cloud analysis, auto tag-clips/notes), **Spiideo Replay**
  (officiating), **AutoData™**, **Portable SmartCam 3**.
- **[CLAIM]** automated wide-angle capture + player tracking + auto-tagging.
  Footage "available within seconds".
- No volleyball-specific case study found. **Pricing not public** (/pricing 404s).
- **Learn:** architecturally the closest thing to a VolleyVerse camera rig, but
  AutoData appears to solve *framing and highlights*, not event *semantics*. A
  plausible **capture/production partner** rather than an event-detection
  competitor.

### Pixellot — https://pixellot.tv **[VERIFIED]**
- **Show S3** (fixed AI camera, "19 sports including volleyball"), **Air /
  Air NXT** (portable, 10,000+ clubs), **VidSwap / Pixellot Analytics**.
- **[CLAIM — the boldest volleyball claim found]:** "AI algorithms analyze the
  video feeds and identify the location of the ball, players, and other
  objects", with volleyball- and beach-volleyball-trained ML, including
  **"spike, block, and serve breakdowns"**.
- **Could not verify** whether "breakdown" means a labelled event with an
  outcome or merely a highlight clip. Case studies are **regional/lower-tier
  club** volleyball (Cuenca, Sant Just, Benfica) — no top-tier league reference,
  no accuracy numbers.
- **Pricing:** not public; explicitly **"Camera-as-a-Service"** / revenue-share.
- **Learn:** test this empirically — get a demo and inspect real output. If the
  claim is real they are a direct threat; if the volleyball layer is shallow
  they are a partner who would license a real one.

### Veo (Cam 3) — https://veo.com **[VERIFIED]**
- Football/rugby/lacrosse/basketball/hockey-first. **Volleyball appears only in
  the quote-generator form** — no case study, no analytics. Auto event detection
  is claimed for its core sports only.
- Pricing: hardware purchase + subscription; exact figures rendered as
  `From {price}` placeholders — **not obtainable**.
- **Learn:** essentially zero volleyball investment. Not a near-term threat;
  a low-risk integration/co-marketing conversation.

### Hudl Focus / Focus Flex — https://www.hudl.com/products/focus **[VERIFIED]**
- **[VERIFIED, quoted]** uses "computer vision to follow the flow of the game"
  — i.e. **auto-framing only**, not event detection. Integrates with
  **Sportscode for live coding**, which presumes a human coder.
- **Learn — keep this distinction sharp:** "AI camera" in this segment
  overwhelmingly means **AI cinematography** (follow the play so nobody works a
  joystick). That is a completely different and much easier problem than
  **AI event detection**. Do not let competitors' marketing blur the two.

---

## PART 3 — Elite tracking tech (cost/difficulty ceiling)

### Genius Sports "Data Capture" (Second Spectrum lineage) — the key reference point
- URL: https://www.geniussports.com/data-capture **[VERIFIED]**
- "A lightweight, in-venue **iPhone camera system**" producing **"Mesh
  Tracking"** (positional data) and **"Auto Event Data — real-time play-by-play
  events"**, plus automated multi-angle video and scoreboard integration.
- **[CLAIM]** "100% Automated". **[VERIFIED nuance]** the same page also
  references **"Trained Statisticians"** in the pipeline — a human-hybrid layer
  is acknowledged; the split is not disclosed.
- **Basketball-specific**, "over 1,000 basketball leagues, federations and NCAA
  schools". No volleyball.
- **Learn — the most important strategic fact in this report.** The **same
  parent company that owns volleyball's manual-scouting incumbent** has already
  shipped, at scale, a "lightweight camera + AI + real-time event data +
  human-statistician hybrid" system — **for basketball**. They have the capital,
  the volleyball customers, the officiating rigs already on courts, and a proven
  playbook. They have **not** applied it to volleyball. Either volleyball CV is
  harder (more simultaneous actors, contact ambiguity, no discrete "shot"
  moment), or it isn't prioritised. Either way: the incumbent has the blueprint
  and could enter at any time.

### Sportlogiq — **[VERIFIED]** "patented computer vision and machine learning",
self-described leader in **hockey** analytics. No volleyball. Proof CV-only
tactical analytics can lead a category — in one sport.

### SkillCorner — **[VERIFIED]** "automated player and ball tracking… from **any
single camera source** including broadcast, tactical or wide-angle video", **no
dedicated venue hardware**. Football, basketball, American football. **No
volleyball.**
- **Learn:** strongest evidence that **single-camera, hardware-light tracking is
  commercially viable today** in other team sports — and that nobody has pointed
  it at volleyball. A clear technology-transfer gap, and a candidate approach
  (use existing broadcast/wide-angle feeds rather than installing a rig).

### KINEXON — **[VERIFIED]** volleyball offering is **wearable/IMU** (PERFORM IMU
/ LPS) plus **"xBall"**; ~50 volleyball metrics (jump count, impact force,
landing load). This is **sports-science load monitoring, not rally/event
detection.** Pricing quote-only.
- **Learn:** a plausible **sensor-fusion partner** — IMU jump/impact data could
  disambiguate exactly the contact calls vision struggles with.

### Catapult — https://www.catapult.com/sports/volleyball **[VERIFIED]**
- Wearables (Vector Pro/Core, T7). **[CLAIM]** "Serve, set, and spike tracking
  with real-time insights" and "automatic identification of key volleyball
  movements" — **wearable-based** auto-classification, not CV.
- **Learn:** a wearable competitor *already claims* automatic serve/set/spike
  identification. Different instrumentation, same customer promise. Its cost:
  every player must wear a sensor — which most youth/club markets resist.
  (Note: a separate Wikipedia fetch of Catapult's client list found **no
  volleyball mention**, so the depth of their volleyball business is unclear.)

### Stats Perform AutoStats, ChyronHego TRACAB — **[UNVERIFIED]**, pages 404'd /
timed out. Well known in-industry as broadcast MOT systems; not re-verifiable.

---

## PART 4 — Other sports as a difficulty proxy

**Tennis — the most mature automated-officiating category.**
- Hawk-Eye ELC **[VERIFIED]**: "450+ courts since 2005". Hawk-Eye Live (fully
  automated calling) is established at majors — exact tournament list unverified.
- **PlaySight** **[VERIFIED]**: fixed multi-camera "SmartCourt" (PS Pro / 4K /
  8K) + portable PS GO; **SmartTracker™** auto-follow; "VAR Light" replay;
  tennis/pickleball/padel/squash/table-tennis + NBA/MLB/NHL/NCAA. No pricing.
- **SwingVision** (swing.vision) — **[UNVERIFIED, JS SPA]**. **[CLAIM]** phone-only
  automated scoring, stats, highlights and line calling for tennis/pickleball.
  No accuracy or pricing verifiable.
- **Baseline Vision** **[VERIFIED]**: single portable camera clipped to a
  netpost, no external power or WiFi, ~20s setup; tracks player/ball, shot
  placement, ball speed, net clearance; live in/out cues; post-match stats.
  Pricing not public.
- **Wingfield** **[VERIFIED]**: fixed court-mounted cams for padel/tennis/
  pickleball; auto-highlights, VAR; 700+ clubs incl. Rafa Nadal Academy.
  Pricing "on request".
- "In/Out" app — domain expired/parked; **could not verify it still exists**.
- **Why tennis is the easy case:** one ball, 2–4 players, a single well-defined
  geometric event (ball crosses a line), minimal occlusion, fixed court
  reference. And it still took Hawk-Eye ~20 years (2005 →) to reach trusted
  fully-automated line calling at the top level.

**Basketball.**
- **NEX Team HomeCourt** **[VERIFIED]**: **single iPhone/iPad, no other
  hardware**; detects shot makes/misses, shot location, release angle; free tier;
  NBA partnership. No published accuracy.
- **ShotTracker** **[VERIFIED, thin]**: basketball-only "AI-driven video, stats,
  analytics"; underlying sensing mechanism **could not be confirmed** (tech page
  404'd).
- **Huupe** **[VERIFIED]**: smart **rim/hoop sensor** (not CV). **Public
  pricing:** mini $399 (list $799), ARENA upgrade kit $699, ARENA Portable $995
  (list $1,495), ARENA 60" in-ground $2,995 (list $3,995).
- **Why basketball's "shot" is easy:** a binary, unambiguous outcome at a single
  fixed location. That is why it has the most consumer-priced, phone-only, free
  products of any team sport. **Volleyball has no equivalent binary event** —
  every fundamental (set quality, dig difficulty, block kill vs touch) is a
  graded, context-dependent judgement even for trained human scouts.

**Padel / table tennis:** covered by PlaySight and Wingfield; no specialist
system found. Table tennis appears comparatively neglected — flagged as thin
coverage, not a confirmed absence.

---

## PART 5 — Direct answers

### 1. Is anyone already doing fully automatic LIVE volleyball rally-level event detection?
**No confirmed case found.**
- The only product marketed as AI-automated volleyball touch detection —
  **Hudl Assist AI (ex-Balltime)** — is **post-match**, club-tier only, and
  routes college *match* data to **human** coders instead.
- Balltime's own pre-acquisition pitch was ~1 hour turnaround.
- Every product confirmed to run **live** in volleyball (DataVolley,
  Click&Scout, SoloStats, VBStats, and presumably VolleyStation) is **100%
  human-operated**.
- The one company with a proven camera+AI+real-time-events+human-hybrid system
  at scale (Genius Sports Data Capture, 1,000+ orgs) built it **only for
  basketball**, despite owning volleyball's incumbent scouting software.
- Hawk-Eye's live volleyball presence is **officiating replay**, with a human
  making the ruling.

**Conclusion: as of September 2026, nobody has shipped live, fully automatic,
structured volleyball event detection commercially.** Genuinely open field —
but see Q2/Q3 for why.

### 2. Where does the market stop?
The frontier line is strikingly consistent across every sport: **automation
stops at the boundary between (a) tracking/framing/positions and (b) semantic
event classification with attribution.**
- **Shipped:** single-camera ball/player tracking with no venue hardware
  (SkillCorner); auto-framing cinematography with zero operator (Veo, Pixellot,
  Spiideo, Hudl Focus, PlaySight); binary/simple-geometry events (tennis lines,
  basketball make/miss).
- **Not crossed by anyone, in any sport, at production scale for a dense
  multi-actor rally sport:** automatically identifying **who** touched the ball,
  **what type** of action it was, and **how well** it was executed —
  continuously, live, through fast occlusion-heavy sequences. That is exactly
  volleyball's rally structure.
- Volleyball uniquely combines: rapid successive contacts by different players
  in a small area; **subjective quality grading built into the schema itself**
  (a "good" set vs a "bad" one is a scored category); and ball speeds/spin that
  strain consumer-grade cameras.

### 3. What does that tell us about difficulty?
The premise is **hard, not solved-elsewhere-and-unported.** The clearest
evidence: Hudl — with a dedicated AI acquisition, the world's largest volleyball
video corpus, and multi-year runway — has shipped only a post-match, club-tier
feature and still human-codes college and pro. If this were merely an
engineering-effort problem, the best-funded, most motivated player would already
have pushed further.

### 4. Market pricing structure
Clearly bimodal:
- **Manual-entry software (cheap, public, self-serve):** €49–€799/yr.
  A commodity SaaS band.
- **Consumer sensor hardware (public):** $399–$2,995 one-time (Huupe) — the only
  fully transparent hardware pricing found, and it's a simple rim sensor.
- **Everything camera/AI/installed** (Spiideo, Pixellot, Veo team tier,
  VolleyStation, Wingfield, PlaySight, Hudl Focus, KINEXON, Catapult, Genius
  Data Capture): **zero public pricing**, universally "contact sales".
- **No per-hour video-processing price is published anywhere** for any AI sports
  product, including Hudl Assist AI.
- **Implication:** there is **no public price anchor** for "AI camera system per
  court" or "AI video processing per hour" in this vertical. VolleyVerse would
  be setting the market's first transparent price — an opportunity, but with no
  comparable to benchmark against.

### 5. Realistic partners vs competitors
- **Direct competitors (must out-differentiate):** **Genius Sports / Data
  Project** (owns the scouting standard + officiating rigs + a proven basketball
  CV-hybrid playbook) and **Hudl** (Volleymetrics + Assist + the only shipping
  volleyball "AI" product). Both already hold the volleyball customer
  relationships.
- **Best realistic partners:**
  - **Spiideo / Pixellot** — cameras already installed in venues; want more
    sport-specific value on top. Pixellot in particular makes volleyball ML
    claims that are probably shallow; a genuine event-detection layer could be
    licensed onto their existing pipeline.
  - **KINEXON** — volleyball wearables, no rally-event/CV product. Sensor fusion
    (their IMU + our CV) directly attacks the contact-ambiguity problem, and they
    have federation/club relationships.
  - **Veo** — lists volleyball but has built nothing for it. Low-risk entry point.
- **BD-only / eventual acquirer, not a startup-stage partner:** **Hawk-Eye** —
  officiating credibility and an FIVB relationship, but enterprise scale and
  cost structure.

---

## Gaps to close in the next research pass
1. **VolleyStation** — capability/pricing/AI claims completely unverifiable
   (SPA). Needs a JS-capable browser.
2. **SwingVision / Wingfield** pricing — same limitation.
3. **FIVB VIS** definition and the exact scope of Hawk-Eye's current volleyball
   contract.
4. **Sideout Stats, iVolley, ChyronHego TRACAB, Stats Perform AutoStats** —
   domain/page failures; unconfirmed whether still active.
5. **Nothing here empirically tests Pixellot's or Hudl Assist AI's real output
   quality.** The highest-value next action is requesting live demos/trial output
   from both rather than reading more marketing copy.
