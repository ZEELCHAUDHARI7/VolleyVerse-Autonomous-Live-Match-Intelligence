# VolleyVerse R&D Report
## Can cameras + AI automatically produce VolleyVerse's match events?

**Engineering decision document · 12 September 2026**
**Prepared for:** Asite Tech team / VolleyVerse
**Repo inspected:** `github.com/ZEELCHAUDHARI7/volleyverse` @ `162b3f5` (31 Aug 2026), 22,403 lines TypeScript

---

## How to read this document

Every factual claim carries a marker. They are not decoration — the difference
between a vendor's marketing sentence and a number someone measured is the
difference between a plan and a wish.

| Marker | Means |
|---|---|
| ✅ | Verified this session against a primary source, with the URL or file path given |
| ⚠️ | Vendor claim — seen on the vendor's own page, not independently testable |
| 📚 | From general knowledge, **not** re-verified this session. Do not budget or quote externally |
| ❌ | Looked for, could not confirm it exists |
| 🔢 | My own arithmetic from stated inputs, not quoted from any source |

**Two constraints on this research, stated up front.** Web search was blocked
by organisation egress policy (HTTP 403) for this session and the previous one.
All verification was done by direct page fetch, a JavaScript-capable browser,
and the GitHub/Crossref APIs. Semantic Scholar and OpenAlex rate-limited every
attempt (HTTP 429), and arXiv's search endpoints are robots-disallowed. **This
review is therefore a floor, not a census** — there may be 2025–2026 work,
particularly volleyball-specific work, that simply could not be surfaced.

Second: two of the six research files from the previous session
(`04-academic-literature.md`, `06-findings-so-far.md`) were not on disk. The
academic pass was redone from scratch for this report. The synthesis in file 06
was reconstructed from the four surviving files plus the handoff note, so
where this report disagrees with a remembered conclusion from that file, **this
report is the newer evidence.** Three of the previous session's headline
conclusions did not survive contact with new evidence; they are flagged
explicitly in §1 and §6.

---

# 1. Executive summary and the decision

## 1.1 The answer

**Yes, but not the way the question is usually asked, and not all at once.**

A camera-and-AI system can plausibly produce a useful fraction of VolleyVerse's
event stream within 6–12 months. It cannot produce all 25 event types, it
cannot do so at sub-second latency, and it cannot do so accurately enough to
remove the human operator. What it *can* do — and what nothing on the market
currently does — is take the tedious 80% of the operator's workload while the
human keeps the one decision that has to be instant and has to be right.

## 1.2 What changed since the previous session

Three headline conclusions from the earlier research need correcting.

**(a) "Nobody has shipped automatic volleyball event detection commercially" is
no longer true.** ✅ **VolleyStation ships `VS Next` / `VS.AI` today, with
published prices.** The previous session could not read their site (client-
rendered SPA) and marked them unverifiable. Rendered in a real browser, the
product is explicit: *"AI platform for the next generation of volleyball"*,
with **"Automatic Player Identification"** (*"Our AI recognizes all 12 players
on both sides"*), **"Automatic Skill Recognition"**, and *"Say goodbye to the
need for dedicated scouts or coders… Just use your phone."* Pricing is public
and per-match: **$90 for 3 analyses ($30/match), $250 for 10 ($25/match)**,
first match free, results retained 12 months.
✅ [volleystation.com](https://volleystation.com) · [next.volleystation.com](https://next.volleystation.com)

This is the most important single fact in the report. It is simultaneously the
strongest threat and the cheapest research opportunity available — see §6.1 and
§21.1.

**(b) The "live" gap is real and is the *only* remaining gap.** VolleyStation's
own FAQ, which is hidden behind JavaScript accordions: *"Most uploads are
processed automatically in about the same time as the length of the video.
**45 minute match ~ 45 minute processing**"*, and *"It takes about **2 hours**
to upload and analyze your video."* ✅ Their marketing copy elsewhere claims
recognition *"in real time"* — irreconcilable with their own FAQ. Treat it as
marketing language.

Across all eight vendors re-checked: **everything automatic is post-match
batch, and everything live is human-operated.** Hudl Volleymetrics is
*"within 12 hours our professional **analysts** track every touch"* ✅; Spiideo,
despite branding the product **AutoData**, states *"every breakdown is done by
**professionals** trained in the specific context of each individual sport"* ✅;
Genius Sports' Data Capture page carries **"100% Automated"** and **"Trained
Statisticians"** on the same page ✅. Not one of the eight publishes a detection
accuracy figure.

**(c) "Our unfair advantage is that every match our scorers collect is labelled
ground truth" is only half true, and the half that is false is the expensive
half.** `stat_events.ts` is `timestamptz default now()` — the moment the
operator's *thumb landed*, not the moment the ball was touched. ✅
(`supabase/schema.sql:189`). That lag is unknown, variable, and sometimes
batched after the rally ends. The existing corpus is therefore **rally-level
sequence labels, not frame-level labels.** It is still valuable — it is exactly
what weakly-supervised sequence alignment needs — but it is not a drop-in
training set, and **the single highest-value thing to ship this month is the
plumbing that makes future data frame-alignable.** See §14.

## 1.3 The five findings that should drive the plan

**1. The architecture is already right, and this is genuinely unusual.**
VolleyVerse is event-sourced: `stat_events` is append-only and *every* number
in the product — stats, standings, charts, the showcase — is a derived view. ✅
An AI producer that can append correct rows needs to integrate with nothing
else. Writes already use client-minted UUIDs and an idempotent offline queue,
so an external service can mint ids and replay safely with no new machinery. ✅
The required schema change is a handful of nullable columns. No competitor has
this; most are retrofitting AI onto a stats database that stores aggregates.

**2. Roughly a third of the data model is derivable rather than detectable, and
the rules engine is also a constrained decoder.** `src/lib/rally.ts` is a pure
FIVB state machine ✅ (519 lines, no React, no storage, no runtime imports).
Rotation, serve order, front/back row, libero on/off, side-out vs break point
and `FAULT_ROTATION` all fall out of it for free. More importantly it runs
*backwards*: a serve can only come from the serving side's P1; a block can only
come from a front-row player; the third contact is a set by the team that
received. That turns a 12-way "who touched it" question into a 1-to-3-way
question at most contacts. **Nobody else in this market has a production FIVB
state machine wired to their event model.** See §5 and §11.3.

**3. The hardest problems are contact detection and identity, not ball
tracking — and both have a cheap, unexplored angle.** Ball tracking is close to
solved and MIT-licensed: WASB reports **F1 88.0 / accuracy 80.0** on volleyball
at a 4-pixel tolerance, with pre-trained volleyball weights ✅. Contact
detection is not: the only real-world single-camera volleyball measurement
found puts the contact frame correct in **48.9%** of cases ✅ (beach, 25 fps).
Player identity is worse: jersey numbers are legible in only **5.0–8.7%** of
player crops ✅ (hockey). But **nobody has tried audio for volleyball** ✅ — the
whistle, and the contact transient, are loud, distinct, occlusion-immune,
lighting-immune, and cost nothing. See §11.1.

**4. "Live" is the wrong latency target, and relaxing it is nearly free.**
Strictly causal inference costs roughly **20 mAP points, ~29% relative**, on
the only clean online-vs-offline comparison verified (THUMOS14: TriDet 69.3
offline → MATR 49.5 online) ✅. But volleyball hands you a natural buffer: the
dead-ball interval between rallies. Nobody needs per-contact detail in 200 ms —
they need it **before the next serve**. That is a ~8–20 second budget, not a
200 ms one, and it converts the hardest published problem in the field into one
of the easier ones. See §16.

**5. The realistic MVP keeps the human, and that is a feature.** The operator
keeps the instant 1-bit decision — *who won the point* — which is the only
input the live scoreboard needs, is impossible to get wrong, and takes one tap.
The AI takes the latency-tolerant detail work: who touched it, what the action
was, how well it was executed. This is not a compromise position; it is the
correct decomposition, because the two halves have completely different latency
and accuracy requirements and there is no engineering reason to force them
through the same pipeline.

## 1.4 The recommendation in one paragraph

Do not buy a camera rig next week. Spend **$250 on VolleyStation's 10-analysis
pack** and evaluate a competitor's shipping output against your own ground
truth — that single purchase will teach you more about achievable accuracy than
three months of building. In parallel, ship the schema migration and a
"record while you score" capture mode so that every match from now on produces
frame-alignable data. Then run exactly two technical spikes on your own gym
footage: **WASB ball tracking** (does your real-world F1 land near 88% or near
54%?) and **audio rally/contact segmentation** (can a whistle detector and an
onset detector alone recover rally boundaries and contact counts?). Build the
**serve** first, because it is the one event where the actor is free from the
rules engine, the timing is known, and the outcome is geometric. Deliberately
do not attempt 3D ball position, net touches, double contacts, jersey-number
identity, subjective grades, or sub-second latency. Full plan in §21.

---

# 2. Scope, method, and what this report is not

**The question.** VolleyVerse is a live volleyball league-management platform.
Today a human operator sits courtside and taps every contact of every rally.
Can cameras plus AI generate those same structured events automatically, in
real or near-real time, and feed them into the existing product?

**What was examined.** The VolleyVerse codebase (cloned and inspected directly
this session ✅); eight commercial vendors re-verified against their live sites;
~40 GitHub repositories with licences and activity checked via the API; and a
fresh academic pass across four sub-literatures (volleyball/group-activity CV,
ball tracking and 3D reconstruction, temporal action spotting including online
methods, and player identity/pose/calibration).

**What this report is not.** It is not a market-sizing or pricing study. It is
not a build plan with sprint-level estimates. It contains **no accuracy figure
that anyone measured on VolleyVerse's own footage**, because no such footage has
been captured and labelled yet — closing that gap is the first recommendation.

**A note on skepticism.** The dominant failure mode in this domain is
reading a benchmark number as a product capability. Three concrete examples
from this session's evidence, all verified:

- Group-activity models report **~95%** on the Volleyball Dataset — but under a
  matched backbone *without* ground-truth boxes at test time, the same methods
  fall to **83.3–87.4** ✅ (DFWSGAR, CVPR 2022). The headline protocol hands the
  model perfect player boxes it would never have in production.
- TrackNetV2's own lineage reports **F1 97.03** on badminton; WASB's independent
  re-implementation under a common protocol scores the same method **F1 83.6**
  on volleyball ✅. Single-paper numbers are systematically optimistic.
- Ball tracking papers report >90%; the one study that ran on real single-camera
  match footage got **54.2%** and wrote, in print: *"Tracking results of over
  90% from the literature could not be confirmed."* ✅ (Gomez et al., PLoS ONE
  2014).

Where a number in this report could plausibly be misread as a promise, it is
labelled with the conditions under which it was obtained.

---

# 3. VolleyVerse today: the workload being automated

## 3.1 The manual loop

Route `/console/matches/[id]/rally` → `SetupWizard` → `SetRotationGate` →
`LiveScreen` → `CourtBoard` ✅. The operator enters the toss, places the
starting six plus liberos for both teams, then for every contact either taps a
player and then ✓ / O / ✗, or **holds the player and flicks** — `src/lib/gesture.ts`
resolves a pointer delta into `LEFT | RIGHT | UP | NONE` with a 24 px threshold
and deliberately wide sectors, because *"a scorer watching the court is not
looking at the screen at all"* ✅.

## 3.2 The number that justifies the whole investigation

🔢 A 25-point set runs roughly 45–55 rallies; a tracked rally is 4–8 contacts.
That is **~200–400 deliberate inputs per set** and **~1,000–2,000 per five-set
match**, under time pressure, from one person who must also watch the court.

The codebase itself is the best evidence that this is unsustainable at full
fidelity. Three separate features exist purely to let the operator fall behind
gracefully: `skipPhase()` advances the rally state machine without logging
anything ✅; `src/lib/free-rally.ts` is an entire alternative engine that drops
the phase sequence and keeps exactly one inference (*the serving side's P1,
tapped first in a rally, is a serve; everything else is a spike*) ✅; and the
flick gesture exists because tapping small targets while watching a court does
not work. `src/lib/blocks.ts` carries an unusually honest comment about what a
block success rate can even mean given the app never sees a block that was
jumped too late ✅.

**Read that as a data-quality warning, not just a UX note.** The human ground
truth against which any model will be trained and evaluated is itself
incomplete and inconsistent in known, structured ways. §14.3 returns to this.

## 3.3 What the operator is actually good at

Worth stating plainly, because it determines the MVP split. A human courtside
is *excellent* at the instantaneous binary — the ball hit the floor, that side
won the point — and gets it essentially always right, with one tap, with zero
latency. A human is *poor*, under time pressure, at the dense detail: which of
three blockers touched it, whether that reception was "good" or "poor", whether
the dig was "super" or merely a "save". Those are precisely the judgements the
existing schema demands 1,000–2,000 times a match.

An AI has the mirror-image profile. It is bad at the instant unambiguous call
under adversarial conditions (a ball landing on a line, a light touch) and
comparatively good at consistent, repeated, latency-tolerant classification.
**The split writes itself.**

---

# 4. The integration surface

## 4.1 Event sourcing means the AI integrates with one function

From the project's own `CLAUDE.md`: *"`StatEvent` (append-only) is the single
source of truth. **No statistic, score, or standing is ever hand-stored.**"* ✅
In Postgres this is `stat_events` plus derived views `match_statistics` and
`standings`. There are no stored aggregates anywhere ✅.

The consequence is large and worth stating without hedging: **an AI event
producer does not need to integrate with statistics, analytics, standings,
charts, the showcase, or any fan-facing view.** If it appends correct rows,
every downstream number in the product updates itself.

`src/lib/repository.ts` defines `DataProvider`; the UI talks only to that
interface; two implementations satisfy it (`LocalProvider` over localStorage,
`SupabaseBackend` over Postgres + Realtime + RLS) ✅. The write method,
verified verbatim this session at `src/lib/repository.ts:122`:

```ts
addEvent: (
  matchId: string,
  teamId: string,
  playerId: string,
  setNo: number,
  type: EventType,
  vsPlayerId?: string | null,
) => StatEvent;
```

**That function is the entire AI → VolleyVerse contract.** An AI producer is a
third writer against the same append-only table. Writes are already optimistic
with client-minted `crypto.randomUUID()` ids and an idempotent offline queue
(`volleyverse:sync-queue:v1`) ✅ — so an external service can mint ids and
replay safely without any new machinery being built for it.

`match_live_state` is one JSONB row per match holding the courtside
`MatchState`; the scorer pushes on every tap (`pushLiveState`), every viewer
subscribes via Supabase Realtime (`subscribeLiveState`), and it is documented
in-repo as a rebuildable projection rather than a source of truth ✅. **A live
scoreboard fed by AI already has a delivery path.** No new realtime
infrastructure is required.

## 4.2 The event model, re-verified

25 event types in 7 groups ✅ — verified by direct count against both the
`CHECK` constraint in `supabase/schema.sql` and the union type in
`src/lib/types.ts`.

> ⚠️ **Correction to the previous session's file 01**, which stated 26. The
> in-repo schema comment also says "26 types". The actual count in both the
> constraint and the TypeScript union is **25**. Minor, but it is the kind of
> off-by-one that propagates into a test fixture.

| Group | Types | Count |
|---|---|---|
| Attack | `SPIKE_POINT` `SPIKE_TOOL` `SPIKE_IN` `SPIKE_ERR` `SPIKE_BLOCKED` | 5 |
| Reception | `RECV_PERFECT` `RECV_GOOD` `RECV_POOR` `RECV_ERR` | 4 |
| Setting | `SET_ASSIST` `SET_GOOD` `SET_ERR` | 3 |
| Blocking | `BLOCK_WIN` `BLOCK_MISS` `BLOCK_TOOLED` | 3 |
| Serving | `SERVE_ACE` `SERVE_IN` `SERVE_ERR` | 3 |
| Defence | `DIG_SUPER` `DIG_SAVE` `DIG_FAIL` | 3 |
| Faults | `FAULT_NET` `FAULT_FOUR_HITS` `FAULT_DOUBLE` `FAULT_ROTATION` | 4 |

`vs_player_id` names the opponent on the other end of a net duel — the blocker
on `SPIKE_TOOL`/`SPIKE_BLOCKED`, the spiker on `BLOCK_WIN`/`BLOCK_TOOLED` ✅.

## 4.3 What the row is missing

Verified by grep against `supabase/schema.sql`: there is **no** `source`,
`confidence`, `rally_id`, `video_ts_ms`, `review_status`, `model_version` or
any spatial column ✅.

| Missing column | Why the AI work needs it |
|---|---|
| `source` | Distinguish `HUMAN` / `AI` / `AI_CONFIRMED` / `AI_CORRECTED`. Without it you can never measure model accuracy against human ground truth, or roll back a bad model version |
| `confidence` | The entire human-in-the-loop design thresholds on a calibrated score |
| `rally_id` | Events are currently an undifferentiated time-ordered stream per set; rally grouping is *reconstructed* by walking events (`rallyOutcomes` in `analytics/volleyball.ts`) ✅. An AI produces events per rally and must write them atomically |
| `video_ts_ms` + `video_asset_id` | No link from an event to the frame it came from. Needed for the review UI, for dispute resolution, and — critically — to build a training set at all |
| `court_x/y/z` | No spatial data exists. Every zone/trajectory analytic competitors offer is impossible today |
| `review_status` | Pending / accepted / rejected, for the verification queue |
| `model_version` | Reproducibility and regression tracking |

**All of these are additive** — nullable columns plus a widened `CHECK`. There
is a clean in-repo precedent: `2026-08-12-attack-detail-and-duels.sql` widened
the type check and added nullable `vs_player_id`, with an explicit warning that
the migration must run *before* deploying code that writes the new values,
because the optimistic UI shows the tap landing while the database silently
refuses it inside the offline queue ✅. Repeat that discipline exactly.

## 4.4 The identity and access problem

The RLS history matters. `schema.sql` originally required
`auth.role() = 'authenticated'` for writes; the console sign-in was removed, so
every write was refused; `2026-08-11-open-console-writes.sql` opened everything
to `anon` with a frank in-file warning that the anon key ships in the public JS
bundle; `2026-08-13-console-admins-and-rls.sql` added a `console_admins`
allowlist; `2026-08-31-user-roles-and-admin-panel.sql` replaced it with
`user_roles` (`super_admin | admin | simple_user`) ✅.

⚠️ `user_roles` is seeded with a hard-coded personal Gmail address as
`super_admin` ✅. An AI producer needs its own service identity — either a
service-role key held server-side, or a dedicated `ai_producer` role with a
write policy scoped to `stat_events` for assigned matches only. **Do not ship a
service-role key to a gym-side device.** That box is physically accessible to
volunteers and, per §19.4, is the most attractive theft target in the rig.

## 4.5 Verdict

**The integration is not the hard part, and that is a genuinely strong
position.** Minimum viable integration, no breaking changes:

1. Migration adding the seven nullable columns above, plus an index on
   `(match_id, review_status)`.
2. An `ai_producer` role in `user_roles` with a scoped write policy.
3. A thin ingestion endpoint accepting a **proposed rally** — an ordered list of
   candidate events with confidences — which validates against `rally.ts`
   (it is pure, so it runs server-side unchanged) and writes the legal ones.
4. A verification queue: anything below threshold lands as
   `review_status = 'pending'` and is confirmed with the same hold-and-flick
   gesture already built for `CourtBoard`.
5. `pushLiveState` continues to own the scoreboard, unchanged.

The one thing needing real work regardless is spatial data: `court_x/y/z` has
no home in the current model and no consumer in the current analytics. Adding
it is a product feature, not an integration detail — and §21.4 argues for not
doing it yet.

---

# 5. Derivable versus detectable

This is the most important structural insight in the investigation and it is
worth being precise about, because it changes the size of the CV problem.

## 5.1 What the rules engine already knows

`src/lib/rally.ts` ✅ models the court as 6 positions with `FRONT_ROW = [4,3,2]`
and `BACK_ROW = [5,6,1]`; `rotate()` shifts clockwise; `resolvePoint(serving,
winner)` returns `{ nextServing, rotateWinner }` where the winner rotates **iff**
they were receiving. Phases run `SERVE → RECEIVE → SET → ATTACK → DEFEND → DIG
→ OVER`. Every contact ends `WIN` / `CONT` / `LOSE`, and *what* was won or lost
is inferred from phase plus who was tapped, never asked. `inferAction(phase,
frontRow)` resolves the one genuine ambiguity — the first touch after an
opposing attack — by court row: **front row ⇒ BLOCK, back row ⇒ DIG** ✅.

`src/lib/substitution.ts` handles court changes on one principle: a rotation
slot is the identity, not the player in it. The libero swap is automatic,
driven only by who holds serve ✅.

## 5.2 The consequence

| Fact | Obtained how | Must CV detect it? |
|---|---|---|
| Current rotation of both teams | Rules engine, from lineup + point sequence | **No** |
| Who is serving | P1 of the serving side | **No** |
| Who is front row / back row | Lineup | **No** |
| Block vs dig on first defensive touch | Court row of the player | **No — free** |
| Libero on/off court | Automatic from serve possession | **No** |
| Side-out vs break point | Derived in `rallyOutcomes` | **No** |
| `FAULT_ROTATION` | Pure rules check | **No** |
| `SET_ASSIST` vs `SET_GOOD` | Did the next event score? | **No — derivable** |
| Set/match completion, standings | Derived views | **No** |

That last one deserves emphasis because it was not in the previous analysis.
`SET_ASSIST` is defined in-repo as *"set that led directly to a point"* ✅. That
is a property of what happens **next**, not of the set itself. A CV system only
needs to detect *that a set occurred and by whom*; the assist/good distinction
falls out of the rally outcome for free. The same logic removes work from
several attack and block outcomes — see §10.

## 5.3 The engine runs backwards too

This is the part nobody else in the market can do. `rally.ts` is not only a
forward simulator; it is a **legal-move generator**, and therefore a constrained
decoder over the AI's output.

- A serve can only be produced by the serving side's P1 — **12 candidate players
  collapse to 1.**
- A block can only come from one of three front-row players — **12 collapse to 3.**
- The third contact on a side is a set by the team that just received.
- A four-hit fault is detectable *by counting*, not by classifying.
- Any proposed event sequence that violates the phase machine is rejected before
  it is written.

🔢 Across a typical rally, the actor-identification problem drops from a 12-way
choice per contact to somewhere between a 1-way and 3-way choice at most
contacts. Given that player identity is, per §9.3, the single weakest link in
the published literature, **this is the most valuable asset VolleyVerse owns
and it already exists, tested, in the repo.**

Constrained decoding against a legal-move generator is how a mediocre classifier
becomes a usable system. The architectural implication is in §12.4.
---

# 6. The commercial landscape, corrected (September 2026)

## 6.1 VolleyStation — the finding that reframes the market

✅ Verified this session by rendering the site in a JavaScript-capable browser.
The previous session's file 02 marked VolleyStation entirely unverifiable and
warned against making competitive claims. That warning is now discharged, and
the answer is not the comfortable one.

**Product line**, verbatim from the rendered homepage:

| Product | Their words |
|---|---|
| VS Score | *"Live, digital scoring solution for junior volleyball"* |
| **VS Next** | *"**Computer vision based breakdowns of matches from single video.** Generate montages, check performance and dive into physical data"* |
| VS Online | *"Our Pro stats system now in online version"* |
| League suite | Scoring, statistics, TV integrations, websites |
| **Remote scouting** | *"Let our remote team of **professional analysts** handle stats for you. We can provide **low latency live data** from any place in the world"* |

**The VS.AI claims** ✅ (from `next.volleystation.com`):
- *"Automatic Player Identification — Our AI recognizes all 12 players on both
  sides, tracking individual player performance and contributions"*
- *"Automatic Skill Recognition — From offense to defense, all the skills and
  details you already expect"*
- *"Automatic Player Workload Metrics — GPS-like capabilities without the need
  for wearable devices"*
- *"No Analyst? No Problem… **Just use your phone**"*
- ⚠️ *"10x the Data"* — no baseline defined.

**Capture requirements** ⚠️: *"A single wide-angle camera view from behind the
baseline works best — even from a phone. 1080p or higher is ideal. Taller
tripods allow us to see more of the total court, improving accuracy."*

This last line is worth dwelling on. A shipping competitor is telling you, for
free, that **a single phone behind the baseline on a tall tripod is a viable
capture rig.** That is a substantial de-risking of the hardware question (§13),
from a source with commercial incentive to claim they need something fancier.

**Timing** ✅, from FAQ accordions invisible without JavaScript:
- *"Most uploads are processed automatically in about the same time as the
  length of the video. 45 minute match ~ 45 minute processing."*
- *"It takes about 2 hours to upload and analyze your video."*

**Pricing** ✅ — the only public per-match price anywhere in this market:

| Package | Tokens | Price | Per match |
|---|---|---|---|
| VS NEXT Starter | 300 | **$90** | 3 analyses → **$30** |
| VS NEXT Plus | 1,000 | **$250** | 10 analyses → **$25** |
| Custom | — | not public | — |

First match free. Results retained 12 months, no subscription.

**Accuracy** ❌ — a full DOM string-search for `accura*` returned exactly one
hit: the tripod-height tip. No percentage anywhere.

**Strategic read.** VolleyStation is split across exactly the line this report
is about: their **live** product is humans (VS Score manual, Remote Scouting
analysts), their **automatic** product is post-match batch at ~1× real-time.
They have FIVB, Volleyball World, LOVB, PlusLiga, USA Volleyball and SV.League
logos on the page ⚠️ (logo strips are not contracts, but they are not nothing).

The actionable consequence is in §21.1 and it is the strongest single
recommendation in this report: **$250 buys ten analyses of matches you already
have ground truth for.** Run their output against your own scorers' event logs
and you obtain, in a week, a measured accuracy number for a shipping
volleyball CV product — a number that exists nowhere in public and that would
otherwise cost you six months to estimate.

## 6.2 The rest of the field

| Vendor | Live? | Automatic? | Volleyball? | Price | Evidence |
|---|---|---|---|---|---|
| **DataVolley 4** (Genius) | Live | ❌ human keyboard codes | Yes — the standard | ✅ €799/yr Pro, €299/yr Lite | ✅ |
| **Click&Scout** (Genius) | Live | ❌ human touch input | Yes | ✅ €49/yr | ✅ |
| **VolleyStation VS Next** | ❌ post-match ~1× | ✅ yes | Yes | ✅ $25–30/match | ✅ |
| **Hudl Assist AI** | ❌ post-match upload | ✅ claimed | Club: match+practice · College: **practice only** · HS: *"coming in 2026"* | not public | ✅ |
| **Hudl Volleymetrics** | ❌ 12h | ❌ *"professional **analysts**"* | Yes, college/pro | not public | ✅ |
| **Pixellot** | ❌ | ✅ highlights; ⚠️ events via VidSwap, 4–8h, method ambiguous | Yes, 19 sports | not public | ✅ |
| **Spiideo** | ❌ | ⚠️ *"AutoData"* but *"every breakdown is done by **professionals**"* | ⚠️ listed, asterisked *"Details available soon"* | not public | ✅ |
| **Veo** | ❌ | ✅ for football only | ❌ **none** — DOM search for `volley` returned zero across `/pricing` and `/other-sports` | ✅ Cam 3 **€1,299** + Starter **€549/yr**; Go from €20/mo | ✅ |
| **Genius Data Capture** | ✅ real-time | ⚠️ *"100% Automated"* **and** *"Trained Statisticians"* on the same page | ❌ basketball + soccer only | not public | ✅ |
| **Hawk-Eye** | ✅ officiating | ball tracking | ✅ **"VolleyTrack" confirmed real** — named under *Ball Tracking* on `/officiating`; no product page exists | not public | ✅ |
| **KINEXON** | live | wearable IMU/LPS | Yes — jump count, landing load, impact force. **No rally events** | not public | ✅ |
| **Catapult** | live | wearable | ⚠️ claims *"serve, set, and spike tracking"* | not public | 📚 not re-checked this session |

Three corrections to the previous session worth noting: **Veo's pricing is now
public** (it renders as `{price}` placeholders in raw HTML, which is why the
last pass missed it) and **Veo definitively has no volleyball** — a raw fetch
suggested volleyball appeared in a menu; direct DOM inspection refuted it.
**Hawk-Eye's "VolleyTrack" is real**, which the last session suspected but
could not confirm; it sits under officiating ball tracking, not event stats, so
it is not an event-detection competitor on the available evidence.

## 6.3 Where the market stops, and what that tells you

The frontier line is identical across every sport examined: **automation stops
at the boundary between tracking/framing/positions and semantic event
classification with attribution.**

Shipped everywhere: single-camera ball and player tracking with no venue
hardware; auto-framing cinematography with zero operator (Veo, Pixellot,
Spiideo, Hudl Focus); and binary/simple-geometry events (tennis line calls,
basketball make/miss).

Not crossed by anyone, in any sport, at production scale for a dense
multi-actor rally sport: automatically identifying **who** touched the ball,
**what type** of action it was, and **how well** it was executed, continuously,
live, through fast occlusion-heavy sequences.

**Keep one distinction sharp.** "AI camera" in this segment overwhelmingly
means **AI cinematography** — follow the play so nobody works a joystick. Hudl
Focus *"uses computer vision to follow the flow of the game"* and then
integrates with Sportscode *for live coding*, which presumes a human coder ✅.
That is a completely different and much easier problem than AI event detection.
Do not let competitors' marketing blur the two, and do not let your own
marketing do it either.

**The difficulty evidence is strong and points one way.** Genius Sports owns
volleyball's manual-scouting standard (DataVolley), volleyball's officiating
rigs (VideoCheck — ⚠️ up to 23 cameras per venue at up to 200 fps), *and* a
proven camera + AI + real-time-events + human-statistician hybrid at scale —
and has applied that last one to **basketball and soccer, not volleyball** ✅.
Hudl bought an AI team (Balltime) for exactly this, and two-plus years on ships
a post-match club-tier feature while still human-coding college and pro ✅. If
this were merely an effort problem, the best-funded, most motivated players
would already have pushed further.

**But the field is not empty any more.** VolleyStation shipped it, post-match,
at $25/match. The open question is no longer "is it possible" — it is "is live
possible, and is anyone's post-match output actually good?" §21.1 answers the
second question for $250.

## 6.4 Pricing structure

Bimodal and almost entirely opaque. Manual-entry software is a commodity SaaS
band at **€49–€799/yr** ✅. Everything camera/AI/installed is quote-only — with
two exceptions found this session: **Veo (€1,299 hardware + €549/yr)** and
**VolleyStation ($25–30/match)** ✅. No vendor publishes a per-hour video-
processing price. No vendor publishes an accuracy figure.

Two implications. First, **VolleyStation's $25–30/match is now the market's
price anchor** and any VolleyVerse pricing has to reason against it. Second,
the universal absence of published accuracy is itself an opportunity: a vendor
who publishes measured per-event accuracy against human ground truth would be
the only one doing so, and in a market where every buyer has been burned by
"AI" that turned out to be a highlight reel, that is a real differentiator.

---

# 7. What open source gives you for free

Full repo-by-repo detail with licences is in `03-opensource-github.md`. The
decision-relevant summary:

## 7.1 The one repo that de-risks the most

**`asigatchov/fast-volleyball-tracking-inference`** — MIT ✅, real-time
volleyball ball detection, benchmarked by its author on 886 labelled 720p/1080p
frames ⚠️ (author's own benchmark, not independent):

| Model | F1 | CPU FPS |
|---|---|---|
| VballNetV4c | **0.902** | 149.6 |
| VballNetGridV2b | 0.892 | 122.9 |
| VballNetGridV3 | 0.867 | 229.3 |
| VballNetFastV1 | 0.799 | **1140** |

A second table from the same author reports accuracy@5px: best **87.25% @
138.7 fps**, fastest **73.20% @ 271.9 fps**, on an Intel i5-10400F CPU ⚠️.

**CPU-only, sub-$1,000-hardware, real-time volleyball ball detection at ~F1 0.9
already exists under MIT licence.** That single fact removes more risk from the
roadmap than anything else in the open-source landscape. The same author
maintains `vball-net` (training side, MIT) and a PyTorch port; ⚠️ he is the
most serious solo volleyball-CV developer on GitHub and is worth contacting.

## 7.2 The week-one stack

| Repo | Licence | Why |
|---|---|---|
| `nttcom/WASB-SBDT` | **MIT** ✅ | The strongest verified volleyball ball tracker, with pre-trained volleyball weights. See §8.2 |
| `asigatchov/fast-volleyball-tracking-inference` | **MIT** ✅ | Working detector on day one, CPU real-time |
| `qaz812345/TrackNetV3` | **MIT** ✅ | Best-maintained heatmap tracker to benchmark against |
| `roboflow/supervision` | **MIT** ✅ | Saves weeks of bbox/tracking-ID/annotation plumbing |
| `roboflow/trackers` | **Apache-2.0** ✅ | ByteTrack/BoT-SORT **without touching AGPL** |
| `lyuwenyu/RT-DETR` | **Apache-2.0** ✅ | Train the player/ball/net detector here so the stack starts commercially clean |
| `open-mmlab/mmpose` (RTMPose/RTMO) | **Apache-2.0** ✅ | Serve vs set vs attack often hinges on arm/torso pose |
| `blakeblackshear/frigate` | MIT ✅ | **Study the architecture** — the most mature OSS "always-on camera → real-time local detection → event/clip generation → API" |

## 7.3 Licensing landmines

These are not hypothetical; several would poison a commercial SaaS.

- **Ultralytics YOLOv8/11/12 — AGPL-3.0** ✅. Ultralytics' own licensing page
  states that *SaaS platforms, APIs, or cloud systems that use YOLO behind the
  scenes* require a paid Enterprise Licence — **triggered by SaaS delivery
  itself, not redistribution.** Either budget for it or standardise on
  RT-DETR / YOLOX / RTMDet and keep Ultralytics to prototyping only.
- **`mikel-brostrom/boxmot` — AGPL-3.0** ✅. Same trigger. Use `roboflow/trackers`.
- **`mkoshkina/jersey-number-pipeline` — CC BY-NC 3.0** ✅, explicitly
  non-commercial. Build jersey OCR from Apache-2.0 **PARSeq** directly if you
  build it at all (§21.4 argues you shouldn't yet).
- **`shukkkur/VolleyVision`** — README says CC BY-NC-ND, GitHub metadata says
  AGPL-3.0 ✅. **Either reading blocks commercial reuse.** Study only.
- **`masouduut94/volleyball_analytics` — GPL-2.0** ✅. The most actively
  maintained volleyball CV repo found and the closest existing thing to
  VolleyVerse's goal (VideoMAE gates SERVICE/PLAY/NO-PLAY, YOLOv8 detects ball
  + serve/receive/set/spike/block). **Read it, do not vendor it.** Zero
  published accuracy numbers.
- **`SoccerNet/sn-gamestate` — GPL-3.0** ✅. Architecturally almost exactly what
  VolleyVerse wants. Needs legal review before any code is borrowed; the
  MIT-licensed `TrackingLaboratory/tracklab` it is built on is materially
  safer for the orchestration layer.
- **"Other"/blank licences** default to **full copyright reserved**, not
  permissive. Treat as AVOID until someone reads the actual LICENSE text.
- **Roboflow Universe volleyball datasets** (~300 community projects) each set
  their own licence ✅. Check per-dataset before training anything you ship.

**General pattern worth budgeting for:** much of the exciting sports-CV research
code is university thesis output under non-commercial or ambiguous terms.
Budget engineering time to **reimplement architectures under clean licences**
rather than assuming reference code is reusable.

## 7.4 The gap

❌ **No dedicated deep-learning volleyball court-keypoint repo exists.** That is
a build-it-yourself item, not a landmine — and volleyball court geometry is
markedly simpler than a soccer pitch (a rectangle, a centre line, two attack
lines, all high-contrast and known-dimension).

Also worth noting as a market signal rather than a dependency: several 0–1★
repos pushed within two weeks of the last scan are independently attempting
rally segmentation, court-calibrated stats and jersey-OCR feasibility for
volleyball ✅. **Several independent people are building this MVP right now.**

---

# 8. What the academic literature actually establishes

A full paper-by-paper review with links and verification markers is in the
rebuilt `04-academic-literature.md` saved alongside this report. What follows
is the decision-relevant extract.

## 8.1 The benchmark that everyone cites is the wrong benchmark

The Volleyball Dataset (Ibrahim et al., CVPR 2016) is the only widely used
volleyball CV benchmark ✅, and group-activity models report ~95% on it. It
should not be used to argue that per-touch event extraction works. Seven
reasons, all verified against the dataset's own repository:

1. **The labels are 4 events × 2 sides.** Strip the side and you have
   `pass / set / spike / winpoint` — and "winpoint" is a celebration, not a ball
   contact. The group task is effectively a **3-way ball-event choice plus a
   court side**, on a clip already temporally isolated for you.
2. **One label per clip, not per touch.** A rally contains 6–12 contacts; the
   dataset assigns one label to a 41-frame window. It never asks the model to
   enumerate the touches in order with timestamps — which is a scoring app's
   entire job.
3. **No serve, no reception, no free ball, no dink, no cover.** Four contact
   classes total. A DataVolley skill alphabet is not representable.
4. **Two-thirds of the individual labels are "standing"** (38,696 of ~56,000
   boxes ✅). Individual-action accuracy in the 80s is inflated by a posture
   class with no volleyball semantics.
5. **There is no ball annotation at all.** A model can reach 95% without ever
   localising the ball — which means it cannot, in principle, decide who
   touched it.
6. **There is no player identity** — no jersey number, no rotation position, no
   persistent track ID. You cannot attribute a touch to a named player.
7. **There is no quality grading.** Nothing encodes execution quality, which is
   half of VolleyVerse's schema.

And the headline number softens under scrutiny: ✅ under a **matched ResNet-18
backbone with no ground-truth boxes at test time**, ARG falls to 87.4, DIN to
86.5, Actor-Transformers to 84.3, SACRF to 83.3 (DFWSGAR, CVPR 2022, which
itself reaches 90.5). The ~95% figures are for a protocol where the model is
handed perfect player boxes it would never have in production.

> ⚠️ **A discrepancy flagged rather than reconciled.** Two independent
> verification passes returned different dataset statistics: the GitHub
> repository gives 4,830 annotated frames from 55 videos with 8 group-activity
> and 9 individual-action labels; the CVPR 2016 paper page gives 1,525 frames
> from 15 videos, 6 team and 7 player labels, and 51.1% accuracy. These are
> almost certainly an original and an extended release. **Do not quote either
> without checking which one you mean.**

## 8.2 The honest benchmark, and its number

✅ **MultiSports** (Li et al., ICCV 2021) is what our problem actually looks
like: spatio-temporally localised, per-person, fine-grained sports actions.
Its **12 volleyball classes are genuine per-touch classes** — serve, defend,
protect, block, first hit pass, spike, second hit pass, adjust, dink, no
offensive attack, save, second attack.

Verified baselines: SlowFast Det. **27.72 frame-mAP@0.5, 24.18 video-mAP@0.2,
9.65 video-mAP@0.5**; MOC (K=11) 25.22 / 12.88 / 0.62.

**That is the number that answers the question.** Ask for per-person,
temporally localised, fine-grained volleyball touches and 2021-era SOTA
delivers **single-digit-to-high-20s mAP**, not 95% accuracy. Methods have
certainly improved since; no current leaderboard figure could be verified this
session. ⚠️ Licence is **CC BY-NC 4.0 — non-commercial**, which is itself a
blocker for training a shipped product on it.

> One verification pass could not confirm volleyball is among MultiSports' four
> sports; the other read the 12-class list directly from the ICCV PDF. The
> stronger evidence wins, but flag it if this number becomes load-bearing.

## 8.3 Ball tracking — close to solved, with a caveat that matters

✅ **WASB** ("Widely Applicable Strong Baseline for Sports Ball Detection and
Tracking", BMVC 2023, NTT — MIT licence, model zoo includes **pre-trained
volleyball weights**). Its volleyball split is large and real: **143,213 train
/ 54,817 test frames**, 1280×720, 39 train games / 16 test games, built on
Ibrahim et al.'s clips with ball annotations from Perez, Liu & Kot (Pattern
Recognition 2022).

Volleyball results at τ = 4 px:

| Method | F1 | Acc | FPS |
|---|---|---|---|
| DeepBall | 60.0 | 57.1 | — |
| BallSeg | 68.4 | 75.0 | — |
| TrackNetV2 | 83.6 | 72.3 | 17.6 |
| MonoTrack | 85.1 | 75.9 | 19.7 |
| **WASB (step=1)** | **88.0** | **80.0** | 15.8 |

Cross-sport WASB F1: soccer 88.2, tennis 95.6, badminton 93.1, **volleyball
88.0**, basketball 82.6. **Volleyball is the second-hardest of the five.**

🔢 At 1280 px across ~15–24 m of court, that is ~1.2–1.9 cm/px, so the 21 cm
ball spans ~11–18 px and a 4-px tolerance is roughly 5–7.5 cm — about a third of
a ball. Good enough for trajectory fitting. But **F1 88 means roughly one frame
in eight yields no usable position, and those failures are not random — they
cluster at contacts, at net crossings and at occlusions**, which is exactly
where you need them.

**The counterweight, and it is severe.** ✅ Gomez et al. (PLoS ONE 2014), real
single-camera beach volleyball at 25 fps: ball tracking **54.2%**, and the
authors wrote in print that *"tracking results of over 90% from the literature
could not be confirmed."* The gap between 88 and 54 is the gap between curated
broadcast clips and a camera someone put on a tripod. §21.2 makes measuring
where *your* gym lands the first experiment.

## 8.4 3D ball position — solved with four cameras, not with one

✅ The Waseda group ran **four cameras at the four corners of a real
gymnasium** with a particle filter and reports **99.23% 3D tracking success at
3.05 ms/frame on GPU** (75.1 ms on CPU — a 24.6× speedup), on 2014 Japanese
tournament footage. ⚠️ "Success rate" is their own criterion; **no metric error
threshold was retrievable**, so do not read it as "99% of frames within X cm".

Monocular 3D is not there:
- ✅ **Apparent-size depth** (Van Zandycke & De Vleeschouwer, CVPRW 2022):
  1.6 px diameter error → **1.8–2.3 m 3D position MAE, ~10% relative distance
  error**. Unusable for deciding who touched the ball.
- ✅ **Physics-fit over a full flight** (MonoTrack, CVSports 2022): ballistic
  model with drag, DLT from court corners and net-post tips, plus priors on hit
  position. **8.0 cm mean error on synthetic data, 14.9 cm without priors.** On
  *real* video they report only reprojection error (23.3–37.1 px). **There is no
  verified metric 3D error on real monocular video anywhere in this review.**

And volleyball is worse-behaved than badminton here: MonoTrack's prior assumes
contact happens near the player's feet plus ~2 m. Volleyball contact heights run
from a floor dig to a 3.2 m spike. **That prior does not transfer.**

## 8.5 Contact detection — the weakest link, with a clear pattern

The verified spread tracks frame rate and camera count almost perfectly:

| Setting | Contact / event accuracy |
|---|---|
| Table tennis, 120 fps, controlled (TTNet) | ✅ **97.0%** |
| Badminton, monocular, curated (MonoTrack HitNet) | ✅ **89.7% acc, F1 0.946** |
| **Volleyball, 4-camera HDTV, 606 real events** (Cheng et al., IEICE 2017) | ✅ **88.61%** |
| **Beach volleyball, 1 camera, 25 fps, real matches** | ✅ **48.9%**, and only within a ±10-frame (±0.4 s) window |

Mechanically every approach detects the contact as a **discontinuity in the
fitted trajectory**. MonoTrack's baseline is literally a derivative test —
53.8% — beaten by a learned HitNet at 89.7% ✅. 🔢 At 30 fps a volleyball
contact lasting a few milliseconds is never observed directly; it is
*interpolated* between incoming and outgoing segments, so temporal resolution is
inherently ±1 frame (±33 ms) and the 3D contact point is an extrapolation, not a
measurement.

❌ **No dedicated volleyball paper on player–ball contact detection could be
found.** The nearest analogue in any sport is **audio-based hit detection in
table tennis** (Zhang, Dou & Chen, ICPR 2006) ✅. §11.1 argues this is the
opening.

## 8.6 Player identity — the number that should change your architecture

✅ **Koshkina & Elder (CVPRW 2024)**, the reference jersey-number pipeline,
reports end-to-end **91.4% on hockey images, 87.45% on SoccerNet test, 79.31%
on the challenge split**. Good numbers. But the load-bearing figure is buried in
their data description:

> **Legible player crops: hockey train 4,706 / 94,036 (5.0%); val 923 / 14,138
> (6.5%); test 2,158 / 24,809 (8.7%).** ✅

**For 91–95% of frames, a vision system cannot read the number at all.**
SoccerNet's own organisers say the same in their own words: *"the jersey
numbers might be visible on a very small subset of the whole tracklet"* ✅.

This is why every published system identifies at **tracklet** level — detect
text where legible, majority-vote across the track — and why the headline
numbers are not the numbers that apply to us:

1. ✅ The SoccerNet metric is accuracy over tracklets **including correct
   predictions of the "−1 / no number visible" class**. A model earns credit for
   correctly saying "I can't see it". For event attribution, that is a miss.
2. ✅ SoccerNet-GSR annotates a tracklet if the number is visible **in at least
   one frame of a 30-second clip**. An event log needs identity at a specific
   40 ms contact instant.
3. Even the best end-to-end soccer figure, 79.31% on the harder split, means
   **one tracklet in five is wrong.**

Additional verified failure modes: *"when the jersey number of the occluding
player is visible it can affect both legibility and number predictions for the
tracklet"* (at a volleyball block, attacker and two blockers overlap at the
exact instant of the event); and *"only one digit may be visible even when the
jersey number consists of two digits"* — #7 vs #17 vs #27 is a live confusion.

## 8.7 The cascade problem, quantified

✅ **SoccerNet Game State Reconstruction** is the closest published task to what
VolleyVerse needs: video → each person's court position + role + team + jersey
number. Its baseline scores **22.26 GS-HOTA**, even though individual modules
score 50–92 under oracle conditions — because the metric's identity term is
binary: *"failing to correctly predict at least one attribute turns the
corresponding detection into a False Positive."* ✅

The 2024 challenge winner tripled that to **63.81** ✅ — but did it with a
closed-set **TeamID model trained on 111 specific uniforms**, and **GS-HOTA
allows a 5-metre localisation tolerance.** 🔢 On a 9×18 m volleyball court, 5 m
is over half the playing area. A GS-HOTA of 63.81 is therefore a *generous*
upper bound on what this stack delivers at volleyball's spatial precision.

**This cascade is the central architectural risk**, and §12 is designed around
avoiding it.

## 8.8 Two pieces of good news on attribution

✅ **SportsMOT** (ICCV 2023) includes volleyball, and volleyball is its *best*
split: MixSort-Byte scores **72.5 HOTA / 87.0 IDF1 / 66.8 AssA / 78.7 DetA** on
volleyball, against 60.8 HOTA for basketball and 66.4 for football. Players are
well separated and the court is bounded. ⚠️ But AssA 66.8 means about a third of
the association work is still wrong, and the errors cluster at the net during a
block and on the floor after a dig.

✅ **Calibration is easier for us than the literature suggests.** SoccerNet's
best calibration result (ACC@5 73.22% at 75.59% completeness) is for a *moving
broadcast PTZ camera*. A fixed gym camera on a rigid 18×9 m court is a one-time
setup task, not a per-frame estimation problem. **Do not import that risk.**

⚠️ One caveat that bites volleyball specifically: the GSR work assumes the
bottom-centre of a player's box lies on the ground plane and notes this
*"limits the precision of the estimated locations in the case of jumps."*
Volleyball is a jumping sport, and spikes and blocks are the worst case.

## 8.9 Action spotting — precision is achievable, live is expensive

✅ Frame-precise spotting is a solved-enough problem *in principle*: **E2E-Spot
(ECCV 2022) reaches 96.1–96.9 mAP at a 1-frame tolerance on tennis**; **T-DEED
(CVPRW 2024)** reaches 85.15 (figure-skating comp) at δ=1 frame rising to 91.70
at δ=2, and won the 2024 SoccerNet Ball Action Spotting challenge. Relaxing the
tolerance by a single frame is worth roughly **+7 to +16 mAP** — precision is
expensive at the margin.

✅ **SoccerNet Ball Action Spotting** is the closest football analogue: 12 dense
ball-contact classes at a 1-second tolerance, best result **86.47 mAP@1** —
trained on only **7 annotated games**.

✅ The honest counterweight: **P2ANet**, 14-class dense table tennis on **25 fps
broadcast**, gets **48% AUC localisation and 82% top-1 recognition**, and its
authors call the benchmark still-challenging. Frame rate is the binding
constraint on fast net sports.

**The cost of going causal**, from the only clean same-metric comparison
verified (MATR, ECCV 2024, Table 1, THUMOS14 average mAP):

| Setting | Method | Avg mAP |
|---|---|---|
| Offline | TriDet | **69.3** |
| Offline | ActionFormer | 66.8 |
| Online | MATR | **49.5** |
| Online | OAT-OSN | 44.6 |

✅ **~20 mAP points, ~29% relative.** Compute is *not* the blocker — MiniROAD
(ICCV 2023) hits 71.8 THUMOS per-frame mAP at **0.0158 GFLOPs and 15.8M
params**, roughly 0.4% of prior SOTA compute ✅. Accuracy-without-lookahead is.

⚠️ **An engineering hypothesis, not a published result:** E2E-Spot and T-DEED
are *not* full-video methods — they run bidirectional temporal modules over
short clips. A live system could likely buy back most of the causal penalty by
accepting a fixed 1–2 second lookahead buffer. No paper measuring this
trade-off could be found. §16 argues volleyball gives you far more than 2
seconds for free.

## 8.10 Quality grading — two papers, both 2025, both reception only

✅ **Nako et al. (ICPRAM 2025)** is the single most relevant paper found: it
trains a TimeSformer on single-view match video **against manually recorded
DataVolley labels**, with a hierarchical front/back → A/B vs C → A vs B pass
grading. ✅ **Mori & Sawano (IEEJ 2025)** does automatic reception evaluation
from video. ⚠️ **No accuracy figure from either could be verified** (SciTePress
metadata confirmed; J-STAGE returned HTTP 500).

Two things follow. First, **grading is a live research direction and reception
is where it starts** — which happens to match §10's independent conclusion about
which grades are tractable. Second, and more useful: **both groups built their
training set by aligning video to manually scouted DataVolley files.** That is
precisely the asset VolleyVerse can generate at zero marginal cost, and it is
the strongest external validation of the data strategy in §14.

## 8.11 Where our problem is harder than every published benchmark

Stated plainly, because optimism here is expensive:

1. **Event density.** ✅ SoccerNet v1 has ~1 event per 6.9 minutes. A volleyball
   rally has 4–8 contacts in 5–10 seconds — 🔢 roughly **0.5–1.5 events/second,
   or 200–500× denser.** Almost all SoccerNet-derived tuning assumes sparsity.
2. **Attribution is a separate pipeline.** Spotting outputs *(time, class)*.
   SoccerNet treats re-ID, tracking and jersey numbers as **three separate
   tasks** ✅. Binding a contact to a player is a second system.
3. **Quality grading has no benchmark analogue** outside the two 2025 reception
   papers. That is action *quality assessment*, a different literature again.
4. **Data volume.** The closest task, SoccerNet-BAS, has **7 annotated games** ✅.
5. **Simultaneity and occlusion.** Blocks and digs involve near-simultaneous
   contacts by multiple players — a failure mode absent from tennis, diving and
   figure skating, where the best spotting numbers come from.

---

# 9. The five hard problems

Everything above collapses into five. Ranked by how much of the risk each one
carries.

## 9.1 Contact detection — HARDEST

Deciding the instant and location at which a player touched the ball. ❌ No
dedicated volleyball paper exists. Best real-world single-camera volleyball
measurement: **48.9%** ✅. Best multi-camera: **88.61%** ✅. Everything
downstream — action type, actor, grade, contact count, four-hit faults — is
built on this primitive. **If contact detection does not work, nothing works.**

## 9.2 Live latency — HARD, but mostly self-inflicted

✅ ~29% relative accuracy cost for strictly causal inference. Mitigated almost
entirely by redefining the latency target (§16). This is the problem most likely
to be solved by changing the requirement rather than the model.

## 9.3 Player attribution — HARD, and the literature's answer is wrong for us

✅ Jersey numbers legible in 5–8.7% of crops. ✅ GSR cascade collapses to 22.26
baseline / 63.81 best at a 5 m tolerance. **But volleyball hands you priors no
other sport has** — strict rotational order, six on court, a libero in
contrasting kit, and the rally phase structure. §11.3 argues identity should be
a *constrained assignment over a rally*, not per-frame OCR. ⚠️ No published work
does this; it is the obvious domain-specific lever and it is unexploited.

## 9.4 Quality grading — MEDIUM-HARD, and partly a label problem

Half of VolleyVerse's schema is graded (`RECV_PERFECT/GOOD/POOR`,
`DIG_SUPER/SAVE`, `SET_GOOD`). ✅ Only reception has been attempted in the
literature. The deeper issue is that **the ground truth is subjective and
inconsistent**: "super" vs "save" is one operator's judgement at speed. §14.3
argues some of these grades should be redefined geometrically before any model
is trained on them.

## 9.5 Ball tracking — MEDIUM, and the one you can measure next week

✅ WASB F1 88.0 with MIT-licensed pre-trained volleyball weights; ✅ real-world
single-camera beach measurement 54.2%. The variance between those two numbers
is the whole question, and it is answerable in a week with your own footage
(§21.2). 🔢 The physics is unforgiving: at a 36.1 m/s serve the ball travels
**1.20 m between frames at 30 fps** and 0.60 m at 60 fps; at 1/1000 s shutter it
smears 3.6 cm (~17% of ball diameter), and gym lighting often will not permit
1/1000 s.
---

# 10. Event-by-event feasibility: all 25 types

This is the operational core of the report. For each event the question is not
"can a model classify this in a curated clip" but "can a system decide this,
with an actor attached, from a fixed camera in a school gym, well enough that a
human confirming it is faster than a human entering it."

**Rating key**

| | Meaning |
|---|---|
| ⚫ **FREE** | Derivable from `rally.ts` or from the surrounding event sequence. No CV required |
| 🟢 **HIGH** | Build first. Strong priors, geometric outcome, or a constrained actor |
| 🟡 **MEDIUM** | Phase 2. Plausible but needs real data and will carry meaningful error |
| 🔴 **LOW** | Do not attempt yet. Either physically marginal at our capture spec, or the human ground truth is too noisy to train against |

**Two dimensions are scored separately**, because they fail independently:
**Actor** (which named player) and **Label** (which event type / grade).

## 10.1 Serving — build this first

| Event | Actor | Label | Overall | Why |
|---|---|---|---|---|
| `SERVE_IN` | ⚫ FREE | 🟢 HIGH | 🟢 **HIGH** | Actor is P1 of the serving side — the rules engine already knows it, no detection at all. Timing is known (rally start). Label = ball crossed the net and stayed in play |
| `SERVE_ERR` | ⚫ FREE | 🟢 HIGH | 🟢 **HIGH** | Ball into the net or landing outside the court. Pure geometry against a calibrated court. The most mechanically decidable event in the entire schema |
| `SERVE_ACE` | ⚫ FREE | 🟡 MEDIUM | 🟡 **MEDIUM** | Defined in-repo as *"untouched serve, instant point"* ✅. Requires proving a **negative** — that nobody touched it. A light fingertip touch is exactly the contact class §8.5 shows is hardest. Confusable with `SERVE_IN` + `RECV_ERR` |

**The serve is the right first target and it is not close.** The actor problem —
the single weakest link in the published literature (§8.6, §9.3) — **disappears
entirely**, because FIVB rotation rules and VolleyVerse's own state machine
already determine who is serving. The timing problem disappears because the
serve is the first contact of a rally and rally boundaries are the easiest thing
in the whole pipeline to detect (§11.1). What remains is a ball-trajectory and
landing-point question against a calibrated static court, which is the part of
the problem the literature is strongest on.

🔢 A five-set match contains roughly 180–250 serves. Automating serve events
alone would remove ~10–15% of operator taps and produce the serve-efficiency
statistics coaches actually ask for, with a measurable, publishable accuracy
number. That is a real product increment, not a demo.

**The one honest caveat:** `SERVE_ACE` vs (`SERVE_IN` + `RECV_ERR`) is the same
touch/no-touch judgement that human scorers themselves get wrong. Route
low-confidence aces to the human queue and let the operator resolve them — the
distinction matters to players and it is one tap.

## 10.2 Reception

| Event | Actor | Label | Overall | Why |
|---|---|---|---|---|
| `RECV_ERR` | 🟡 | 🟢 HIGH | 🟡 **MEDIUM** | Outcome is geometric — ball hits the floor or goes out of play off the reception. Actor is constrained to the receiving formation |
| `RECV_PERFECT` | 🟡 | 🟡 | 🟡 **MEDIUM** | See the geometric reframing below |
| `RECV_GOOD` | 🟡 | 🟡 | 🟡 **MEDIUM** | As above |
| `RECV_POOR` | 🟡 | 🟡 | 🟡 **MEDIUM** | As above |

**The grading is more tractable than it looks, if you reframe it.** The in-repo
definitions are ✅ `RECV_PERFECT` = *"ideal ball to the setter"*, `RECV_GOOD` =
*"setter can run the offence"*, `RECV_POOR` = *"in play but limits attack
options"*. Every one of those is a statement about **where the ball ended up
relative to the setter**, not about the receiver's technique.

That means the grade can be computed as a geometric function — distance from the
ball's arrival point to the setter's position, plus arrival height — rather than
learned as a subjective class. You already know where the setter is: the rules
engine knows the lineup, and `S` is a position in it. 🔢 A three-band threshold
on (distance, height) reproduces most of what a human scorer is doing when they
tap ✓ / O / ✗ under time pressure.

This matters because it converts the problem from *"learn a human's subjective
grade"* into *"track the ball's arrival point"*, which is the thing WASB already
does at F1 88. ⚠️ It will not match the human labels exactly — and §14.3 argues
that where they diverge, the geometric definition is probably the better one.

✅ External support: **Nako et al. (ICPRAM 2025)** is a hierarchical reception
grader trained against DataVolley labels, and **Mori & Sawano (IEEJ 2025)** does
automatic reception evaluation. Reception is where the published grading work
starts, and this analysis arrived at the same place independently.

**Actor** is the weak side here. The receiver is one of the 5–6 players in the
receiving formation — better than 12, worse than 1. Serve-receive formations are
highly structured and the ball's landing zone narrows it further. 🟡.

## 10.3 Setting

| Event | Actor | Label | Overall | Why |
|---|---|---|---|---|
| `SET_GOOD` | 🟢 HIGH | 🟡 | 🟡 **MEDIUM** | Actor is almost always the designated setter, whose position the rules engine knows. Detecting *that* a set occurred is a strong-prior temporal problem (second contact of the possession) |
| `SET_ASSIST` | 🟢 HIGH | ⚫ **FREE** | 🟡 **MEDIUM** | ✅ Defined as *"set that led directly to a point"* — a property of the **next** event, not this one. The CV system never classifies it. If the following event is `SPIKE_POINT` or `SPIKE_TOOL` by the same team, the preceding `SET_GOOD` becomes `SET_ASSIST`. **This is free** |
| `SET_ERR` | 🟢 HIGH | 🟡 | 🟡 **MEDIUM** | Ball unplayable, or crosses without an attack, or a double called |

**`SET_ASSIST` is the clearest example in the schema of work the rules engine
does for you.** A naive reading of the event list suggests three set classes to
distinguish; in reality there are two, and one of them is determined by
something that happens afterwards. Any per-frame classifier trying to learn
"assist" as a visual class is learning noise.

**Setter identity is the strongest actor prior in the game after the server.**
Teams run one setter; the rules engine holds the lineup; the setter takes the
second contact on the large majority of possessions. 🟢.

## 10.4 Attack

| Event | Actor | Label | Overall | Why |
|---|---|---|---|---|
| `SPIKE_IN` | 🟡 | 🟢 HIGH | 🟡 **MEDIUM** | The spike itself is the most visually distinct action in volleyball — a jump, a full arm swing, an abrupt ball-speed change. Outcome "kept in play" is derivable from the rally continuing |
| `SPIKE_ERR` | 🟡 | 🟢 HIGH | 🟡 **MEDIUM** | Ball lands out or into the net off the attacker. Pure geometry |
| `SPIKE_POINT` | 🟡 | 🟡 | 🟡 **MEDIUM** | Ball lands in the opponent's court untouched-or-undug. Mostly geometric, but requires distinguishing from `SPIKE_TOOL` |
| `SPIKE_TOOL` | 🟡 | 🔴 LOW | 🔴 **LOW** | Requires detecting that the ball **touched the block** before going out. A deflection off fingertips at the moment of maximum occlusion |
| `SPIKE_BLOCKED` | 🟡 | 🔴 LOW | 🔴 **LOW** | Same physical problem, opposite outcome. Also emits a paired `BLOCK_WIN` on the other side |
| `vs_player_id` (which blocker) | 🔴 LOW | — | 🔴 **LOW** | Identify one of 2–3 jumping, overlapping players at the net at the exact frame of peak occlusion. ✅ *"When the jersey number of the occluding player is visible it can affect both legibility and number predictions"* |

**Detecting a spike is easy; classifying its outcome splits cleanly into
"geometric" and "impossible".** `SPIKE_IN` / `SPIKE_ERR` / `SPIKE_POINT` are
decided by where the ball goes and whether the rally continues — all downstream
of ball tracking. `SPIKE_TOOL` and `SPIKE_BLOCKED` are decided by whether a
fingertip touched a 21 cm ball moving at speed, in front of a wall of arms.

🔢 That is the same physical measurement Genius Sports' VideoCheck addresses
with ⚠️ **up to 23 cameras at up to 200 fps** ✅. A single 60 fps camera is three
orders of magnitude away from that instrumentation. **Do not attempt block
touches in the MVP** — route the whole class to the human.

⚠️ Note a derivation hazard: `analytics/volleyball.ts` deliberately **excludes**
`SPIKE_BLOCKED` and `BLOCK_TOOLED` from its win/error tallies, because each of
those rallies logs two events (one per side) and counting both would double-
score the rally ✅. An AI producer emitting these must preserve the pairing, or
the derived statistics silently break.

## 10.5 Blocking

| Event | Actor | Label | Overall | Why |
|---|---|---|---|---|
| `BLOCK_WIN` | 🟡 | 🟡 | 🟡 **MEDIUM** | Ball deflects sharply downward into the attacker's court and lands. The trajectory reversal is large and detectable — the easiest of the three |
| `BLOCK_MISS` | 🟡 | 🔴 LOW | 🔴 **LOW** | Requires detecting that a block was *attempted and beaten*. Pose-based (jump at net, hands up) — but see the label-noise warning below |
| `BLOCK_TOOLED` | 🟡 | 🔴 LOW | 🔴 **LOW** | Light deflection out of bounds. Same problem as `SPIKE_TOOL` |

⚠️ **`BLOCK_MISS` carries a ground-truth problem, not just a detection
problem.** `src/lib/blocks.ts` contains an unusually candid comment about what a
block success rate can mean *given the app never sees a block that was jumped
too late* ✅. In other words, the human data for this event is already known by
the team to be structurally incomplete. Training a model against it would teach
the model to reproduce the operator's omissions. **Leave it alone until the
definition is tightened.**

⚫ One free win: `inferAction(phase, frontRow)` already resolves block-vs-dig on
the first defensive touch by court row ✅. The AI never has to make that call.

## 10.6 Defence

| Event | Actor | Label | Overall | Why |
|---|---|---|---|---|
| `DIG_FAIL` | 🔴 | 🟢 HIGH | 🟡 **MEDIUM** | The *label* is trivially decidable — the ball hit the floor. The *actor* is genuinely arbitrary: who "failed"? Nearest defender is a convention, not a fact |
| `DIG_SAVE` | 🟡 | 🟡 | 🟡 **MEDIUM** | Ball kept alive off a defensive contact. Detectable as a low trajectory reversal |
| `DIG_SUPER` | 🟡 | 🔴 LOW | 🔴 **LOW** | ✅ *"extraordinary save"* — the most subjective label in the entire schema |

**`DIG_SUPER` is where the label-noise problem is at its worst.** "Extraordinary"
is one person's judgement, made in under a second, while watching a court. Two
operators will not agree; one operator will not agree with themselves across a
tournament. 🔢 A model trained on this will have an irreducible error floor set
by inter-annotator disagreement, not by the model.

There is a way out, and it is the same move as §10.2: **define it
geometrically** — ball speed at contact, distance the defender covered, whether
the contact was below knee height, whether the player left their feet. That is
computable and consistent. ⚠️ But it will not reproduce the historical human
labels, which means you cannot use those labels to validate it. That is a
product decision (§14.3), not an engineering one, and it should be made
deliberately rather than discovered during training.

## 10.7 Faults — and the idea that changes this whole group

| Event | Actor | Label | Overall | Why |
|---|---|---|---|---|
| `FAULT_ROTATION` | ⚫ FREE | ⚫ **FREE** | ⚫ **FREE** | A pure rules check against the lineup at service. `rally.ts` already computes it. Zero CV |
| `FAULT_FOUR_HITS` | 🟡 | 🟡 | 🟡 **MEDIUM** | Derivable *by counting contacts* rather than by classifying a fault — entirely downstream of contact detection (§9.1). If contact counting works, this is free; if it doesn't, nothing else works either |
| `FAULT_NET` | 🔴 | 🔴 LOW | 🔴 **LOW** | Player–net contact. ⚠️ This is what VideoCheck's up-to-23-camera, up-to-200 fps rig exists to adjudicate ✅ |
| `FAULT_DOUBLE` | 🔴 | 🔴 LOW | 🔴 **LOW** | Double contact / lift — among the most contested calls in the sport, disputed by referees in slow motion |

**The reframe: these are referee decisions, not physical events.**

A net touch or a double contact enters VolleyVerse's data because *a referee
blew a whistle and made a signal*, not because a camera measured a contact. So
the detection target is wrong. The right target is the referee.

FIVB's official hand signals are a **standardised gesture vocabulary,
deliberately designed to be unambiguous from a distance** — large, slow,
held for a beat, performed by a person in distinctive clothing standing in a
fixed, known, unoccluded location on a stand at the net post. 📚 (the FIVB
Rules of the Game include an official hand-signal chart; the exact current
signal set was not re-verified this session and should be checked against the
current rulebook before implementation.)

Compare the two problems honestly:

| Target | Actor | Occlusion | Duration | Instrumentation needed |
|---|---|---|---|---|
| Detect a fingertip touching the net | Any of 6 players, mid-jump | Maximum | ~20 ms | ⚠️ up to 23 cameras @ 200 fps |
| Detect the referee's "net touch" signal | One person, known location, distinctive kit | Minimal | ~1–2 s | One camera pointed at the stand |

🔢 The second problem is roughly two orders of magnitude easier, and it produces
exactly the label VolleyVerse stores. Combined with whistle detection from audio
(§11.1) to localise *when* a decision was made, and the rules engine to
constrain *which* faults are legal in the current phase, this is a genuinely
tractable path to the entire fault group — and to `DIG_FAIL`, `SPIKE_ERR` and
every other rally-ending decision.

⚠️ **Flagged as an untested hypothesis.** No published work on referee-signal
recognition in volleyball was found ❌. It requires the referee to be in a
camera's field of view, which is a placement constraint, and it says nothing
about *which player* committed the fault. But it is cheap to test — a few
hundred labelled signal clips — and the upside is large.

## 10.8 Summary

| Rating | Count | Events |
|---|---|---|
| ⚫ **FREE** | 1 (+2 partial) | `FAULT_ROTATION`; plus `SET_ASSIST`'s label and block-vs-dig inference |
| 🟢 **HIGH** | 2 | `SERVE_IN`, `SERVE_ERR` |
| 🟡 **MEDIUM** | 14 | `SERVE_ACE`, all 4 reception, all 3 setting, `SPIKE_IN`/`ERR`/`POINT`, `BLOCK_WIN`, `DIG_FAIL`, `DIG_SAVE`, `FAULT_FOUR_HITS` |
| 🔴 **LOW** | 7 | `SPIKE_TOOL`, `SPIKE_BLOCKED`, `BLOCK_MISS`, `BLOCK_TOOLED`, `DIG_SUPER`, `FAULT_NET`, `FAULT_DOUBLE`, plus `vs_player_id` |

🔢 **The tractable set covers roughly 70% of event volume in a typical match**,
because the 🔴 events are individually rare — tools, block-outs and net faults
are a small fraction of contacts, while serves, receptions, sets and attacks
are almost all of them. A system that handled only the ⚫ and 🟢 and the
geometric half of 🟡 would still remove most of the operator's work.

**That is the MVP thesis stated numerically**, and it is why "AI takes the
detail, human keeps the point" is the right decomposition rather than a
consolation prize.

---

# 11. The underused signals

Three sources of information that the published literature has largely ignored
and that are cheap, available today, and unusually well-suited to volleyball.

## 11.1 Audio — the biggest unexploited opportunity in this report

❌ No volleyball work using audio could be found. ✅ The only close analogue in
any sport is audio-based hit detection in table tennis (ICPR 2006). Meanwhile
every camera-based approach in §8.5 is fighting occlusion, motion blur, frame
rate and lighting to infer a contact that **makes a loud, distinctive noise**.

Three separate signals live in a single microphone:

**(a) The referee's whistle.** Loud, narrow-band, tonal, ~2–4 kHz, utterly
unlike anything else in a gym. It marks every rally start and every rally end,
which means it gives you **rally segmentation essentially for free** — and rally
segmentation is the frame on which the rules engine hangs (§12.2). 🔢 A
classical onset + band-energy detector would likely get this to high accuracy in
a day's work; no deep learning required for a first pass.

**(b) The contact transient.** A spike contact is a sharp, high-energy impulse;
a set is soft; a dig is intermediate; a block is a duller slap. These differ in
attack time and spectral content. 🔢 If even a coarse contact/no-contact
detector works, it directly attacks §9.1 — the hardest problem — with a modality
that is **immune to occlusion, immune to lighting, immune to motion blur, and
independent of frame rate.**

**(c) Serve/ace disambiguation.** §10.1's one caveat — did anyone touch that
serve — is a touch/no-touch question. A microphone answers touch/no-touch
questions better than a 60 fps camera does.

**The cost is essentially zero.** Every phone and every action camera already
records audio; a $20 USB microphone beats all of them. Storage is negligible.
Compute is negligible. And critically, **audio can be tested on footage you have
not captured yet** — any existing match recording with sound is a usable
experiment today.

⚠️ **Honest risks.** Gyms are reverberant and loud; crowd noise, music between
rallies, squeaking shoes and adjacent courts all intrude. Multi-court venues are
the worst case — a whistle from court 3 is indistinguishable from yours without
directional audio. Ball-contact transients from an adjacent court will be
present. None of this is fatal for whistle detection (the temporal pattern of
*your* rally correlates with *your* video) but it may well be fatal for contact
transients in a busy hall. **Test it early precisely because it could fail
early and cheaply.** §21.2 makes it experiment two.

## 11.2 The referee

Covered in §10.7. Restated as a principle: **for every event in the schema that
exists because an official made a ruling, the official is a better detection
target than the physical event.** That covers the entire fault group and
contributes to every rally-ending event.

## 11.3 The rules engine as a constrained decoder

§5.3 established that `rally.ts` collapses actor identification from 12-way to
1-to-3-way at most contacts. The architectural consequence deserves stating
separately because it inverts the standard pipeline.

**The standard sports-CV pipeline** (SoccerNet GSR, and every commercial system
examined) is a cascade: detect → track → re-identify → OCR the jersey → assign
team → project to pitch → classify action. ✅ Its measured behaviour is that
errors multiply: baseline 22.26 GS-HOTA from modules individually scoring 50–92,
because *"failing to correctly predict at least one attribute turns the
corresponding detection into a False Positive."*

**The volleyball-specific alternative** is to treat a rally as a constrained
assignment problem:

1. Detect and track bodies for the duration of one rally (✅ volleyball is
   SportsMOT's best split, 72.5 HOTA / 87.0 IDF1 — track identity within a
   ~10-second rally is comparatively reliable).
2. **Anchor tracks to roster slots at the start of the rally**, when players are
   in known serve-receive positions and the rules engine states exactly who
   stands where. This is the key move: identity is assigned *once per rally*
   from a strong prior, not recovered per-frame from pixels.
3. Read jersey numbers opportunistically to *confirm* the anchoring, not to
   establish it. At 5–9% legibility ✅ you will get several confirmations per
   rally across 12 players — plenty to validate an assignment, nowhere near
   enough to build one.
4. Propagate through the rally under the tracker, and re-anchor at the next
   rally boundary. **Errors do not accumulate across rallies**, because every
   rally resets against the rules engine.
5. Reject any proposed event sequence that violates the phase machine.

🔢 The error-compounding structure is completely different: a cascade multiplies
failure probabilities along a chain, while this design re-grounds against a
known-correct prior every 10 seconds. ⚠️ **No published work does this** — it
requires a production rules engine wired to the event model, which the academic
datasets do not have and the commercial vendors have not built. It is, as far as
this investigation can establish, **VolleyVerse's only genuine technical moat.**

## 11.4 One signal deliberately not recommended

Wearables. ✅ KINEXON's volleyball offering is IMU/LPS load monitoring (jump
count, landing load, impact force) with no rally-event product; ⚠️ Catapult
claims serve/set/spike identification from wearables. ⚠️ **Whether players may
legally wear tracking sensors in official FIVB or NCAA competition could not be
established** — four attempts to fetch the relevant rulebooks failed, and no
citable rule either permitting or forbidding it was found. Football's IFAB
amended Law 4 in 2015 to permit EPTS ✅, so rulebooks *do* get rewritten for
this, but volleyball's status is genuinely unknown.

Independently of the rules question: wearables say nothing about the ball,
require every player to wear and charge a device, and impose a second parallel
RF infrastructure. For a school-and-club market this is the wrong shape. **Not
in scope.**

---

# 12. Proposed pipeline architecture

## 12.1 Design principles

1. **Never write directly to `stat_events`.** Propose rallies; let the server
   validate and write. The AI is an *event proposer*, not an event author.
2. **The rally is the atomic unit**, not the frame and not the match.
3. **Re-ground against the rules engine at every rally boundary** so errors
   cannot accumulate (§11.3).
4. **Fuse audio and video from the start.** Retrofitting a second modality is
   much harder than designing for two.
5. **Emit calibrated confidence, or emit nothing.** The human-in-the-loop design
   (§15) is worthless without it.
6. **Every proposal carries a video timestamp**, always, even when nobody is
   watching. That is how the training set gets built (§14).

## 12.2 The stages

```
 ┌─ audio ──────────────────────────────────────────────────┐
 │  whistle detector ──────► rally boundaries               │
 │  onset/transient detector ─► contact candidates (t, type) │
 └───────────────────────────────┬──────────────────────────┘
                                 │
 ┌─ video ───────────────────────▼──────────────────────────┐
 │  1. court calibration (one-time, from painted lines)      │
 │  2. ball detection+tracking  (WASB / VballNet, MIT)       │
 │  3. player detection+tracking (RT-DETR + roboflow/trackers)│
 │  4. pose (RTMPose/RTMO) — for action type, not identity    │
 └───────────────────────────────┬──────────────────────────┘
                                 │
 ┌─ fusion ──────────────────────▼──────────────────────────┐
 │  5. contact detection: trajectory discontinuity  ⨁ audio  │
 │     transient ⨁ pose event  →  (t, court_xy, track_id)    │
 │  6. actor assignment: track_id → roster slot, anchored at  │
 │     rally start from rally.ts lineup, confirmed by OCR      │
 │  7. action typing: phase prior ⨁ pose ⨁ ball kinematics    │
 │  8. grading: geometric features (§10.2, §10.6)             │
 └───────────────────────────────┬──────────────────────────┘
                                 │
 ┌─ constrained decode ──────────▼──────────────────────────┐
 │  9. beam search over candidate event sequences, scored by  │
 │     model confidence, REJECTING any sequence illegal under │
 │     rally.ts. Output = the best legal rally.               │
 └───────────────────────────────┬──────────────────────────┘
                                 │
 ┌─ ingestion ───────────────────▼──────────────────────────┐
 │ 10. POST proposed rally → edge function → validate against │
 │     rally.ts (pure, runs server-side unchanged) → write    │
 │     stat_events with source/confidence/rally_id/video_ts   │
 │ 11. below threshold ⇒ review_status='pending' ⇒ human queue│
 └──────────────────────────────────────────────────────────┘
```

## 12.3 Why stage 9 is the interesting one

Stages 1–8 are conventional and largely available off the shelf. **Stage 9 is
where VolleyVerse's advantage is actually realised**, and it is the part no
competitor can copy quickly.

A per-contact classifier produces, for each contact, a distribution over event
types and actors. Naively you take the argmax and you inherit the cascade
problem of §8.7. Instead, treat the rally as a sequence and search for the
**highest-scoring legal sequence** under `rally.ts`. Illegal hypotheses — a
serve from a non-P1 player, a block by a back-row player, a fourth contact
without a fault, a set by the team that did not just receive — are pruned before
they are scored.

🔢 The effect is the same as constrained decoding in speech recognition: a
mediocre acoustic model plus a strong language model outperforms a good acoustic
model alone. `rally.ts` is the language model, it is already written, it is
already tested, and it is pure — meaning it runs unchanged in the ingestion
endpoint, in the edge box, and in the browser.

## 12.4 Where compute lives

**Do not put the model in the gym on day one.** For an MVP whose output is
consumed within the inter-rally window (§16), a wired or 4G uplink streaming
compressed video to a server is adequate, and it removes an entire category of
operational problems (§19.4). 🔢 At H.265 1080p60, one camera is ~4–6 Mbps ✅ —
within reach of a reasonable venue uplink, and recordable locally as a fallback.

Move to edge compute when, and only when, latency measurement says you need to.
§13.3 covers the hardware when that day comes.

---

# 13. Hardware: what to test, what to skip

Full BOM tables for seven configurations are in `05-hardware-and-edge-compute.md`.
The decision-relevant extract follows.

## 13.1 The physics that constrains everything

🔢 At an elite serve of ~130 km/h (36.1 m/s):

| FPS | Ball travel between frames |
|---|---|
| 30 | **1.20 m** |
| 50 | 0.72 m |
| 60 | 0.60 m |
| 120 | 0.30 m |

| Shutter | Blur distance | As % of ball diameter |
|---|---|---|
| 1/250 s | 14.4 cm | ~69% — a smear, not a circle |
| 1/500 s | 7.2 cm | ~34% |
| 1/1000 s | 3.6 cm | ~17% — workable for centroid tracking |
| 1/2000 s | 1.8 cm | ~9% |

**These two requirements fight each other under gym lighting.** A 1/1000 s
shutter admits ~1/16 the light of a 1/60 s exposure, forcing high sensor gain,
which produces exactly the grain that destroys a detector's ability to find ball
edges. ✅ Typical unrenovated school/club gyms run ~300–500 lux, well-renovated
LED gyms 750–1000+, FIVB broadcast venues 1000–1500+ — and the difference
between those is a venue capex item you do not control.

⚠️ **India-specific trap worth flagging explicitly.** India runs 50 Hz mains, so
flicker-safe shutter speeds are multiples of **1/100 s**, not the 1/60-family
multiples that US camera defaults and US-authored tutorials assume. Set
anti-banding to 50 Hz explicitly, and note that 1/1000 and 1/2000 are clean
multiples of neither — banding can persist under bad fixtures regardless.

✅ **Consumer security cameras are out.** Three independent datasheets across
three resolution classes all cap at 20–30 fps: Reolink RLC-1224A 20 fps, RLC-810WA
20 fps fixed, Axis M3216-LVE 25/30 fps, Axis P3737-PLE 20 fps per channel. This
is not coincidence — it is the ISP/encoder pipeline these vendors build for
surveillance. ⚠️ Also, for an India-based company selling into
university/government-linked facilities, Hikvision and Dahua carry real
compliance risk: ✅ India banned Chinese surveillance equipment including
Hikvision from government buildings, critical infrastructure and public-sector
projects in April 2026 — verify against the actual notification text before
specifying either brand for any public-sector venue.

## 13.2 The one hardware experiment worth running

**Do not buy a four-camera rig.** Buy two cameras and answer one question.

| Path | Kit | Cost | What it tests |
|---|---|---|---|
| **A — zero-cost** | A phone or an action camera on a tall tripod behind the baseline | **$0** (BYO) | ⚠️ Exactly the rig VolleyStation tells its own customers to use ✅. If this works, the hardware question is over |
| **B — the real-camera hypothesis** | 1× FLIR/Teledyne BFS-PGE-19S4C-C, 2 MP global shutter @ 60 fps, **$655** ✅ + fast lens ~$150–300 + PoE injector or 12 V supply + a mini-PC | ~**$1,000–1,400** | Does true global shutter at 60 fps measurably improve ball-tracking F1 over a rolling-shutter consumer sensor, *in your gym, at your lux*? |

**Run both on the same rallies and measure the delta in ball-tracking F1.** That
single A/B answers the hardware question empirically for ~$1,000, instead of
being argued about for six months. 🔢 If the delta is small, the entire product
can ship on phones and the BOM collapses; if it is large, you have a
quantified justification for a real camera.

⚠️ Two integration gotchas on path B that are routinely forgotten: most GigE
Vision machine-vision cameras are **not PoE-native** ✅ (budget an injector or a
12 V supply), and they ship as a bare C/CS-mount body — **the lens is a separate
purchase** and its aperture matters more than the sensor under gym lighting.

**Action cameras are not a production sensor.** ✅ The DJI Osmo Action 5 Pro's
*best case* is 240 minutes at 1080p/24 fps with Wi-Fi off — less at the modes
you actually want — against an 8-hour tournament day; none are PoE; none speak
RTSP/NDI/SRT natively; and all throttle thermally on sustained high-fps
recording because the failure mode is heat, not battery. Fine for a volunteer's
supplementary angle. Not a primary sensor.

## 13.3 Compute, when you eventually need it

✅ Verified prices: Jetson Orin Nano Super devkit **$249**; Orin Nano 8 GB module
$299; Seeed reComputer J3011 (cased, industrial) **$813.75**; Hailo-8 M.2 26 TOPS
**$169**; Hailo-8L **$85**. Cloud: AWS g4dn.xlarge (T4) **$0.526/hr**,
g6.xlarge (L4) **$0.805/hr** ✅.

⚠️ **The honest throughput planning number is 1–2 camera streams of real-time
1080p60 detection+tracking per Orin NX-class module**, not the 4–8 that a naive
reading of a 200–400 fps benchmark suggests. Those benchmarks exclude
pre/post-processing, use the smallest model variant, and do not model
multi-stream contention on shared GPU/DLA engines.

🔢 Cloud batch is cheap for a post-match MVP: at ~3× real-time on a T4, one hour
of footage costs roughly **$0.18**; at 1× on a heavier model, **$0.53–$1.01 per
camera-hour**. Against VolleyStation's $25–30/match price point ✅, compute is
clearly not the cost driver in this business — which is worth knowing before
anyone optimises it.

## 13.4 Storage and network

✅ 1080p60 H.265 is ~4–6 Mbps, ~1.8–2.7 GB per camera-hour. 🔢 A 4-camera
8-hour tournament day is ~80 GB — trivially one consumer NVMe. Raw NDI for the
same day would be ~2.16 TB ⚠️ (~25× more), so **decide up front whether raw
footage is ever retained for retraining or dispute review.** Given §14 argues
the footage *is* the asset, the answer is probably "keep H.265 proxies forever,
keep raw never".

Use RTSP camera→box on the LAN and SRT for anything leaving the building ✅.
PoE switch budgets are a real constraint (✅ Netgear GS308LP $52.99 / 8-port;
GS116EP $279.99 / 16-port, 180 W) — check the total power budget against camera
class before ordering.

## 13.5 Placement, calibration and the venue

✅ Court is 9 × 18 m, free zone minimum 3 m all sides, ceiling minimum 7 m
(recommended 8 m). 🔢 Framing court + free zone (24 m) from a few metres behind
the baseline demands a fisheye-class FOV whose edge distortion degrades
resolution *exactly where the far-court ball needs it most*. Mounting higher and
further back needs ~90–100° — which is the geometry the single-unit commercial
products are built around.

**Mounting hierarchy:** ceiling truss (best, needs a lift and facility
permission) > elevated side-wall bracket beyond the free zone > tripod at the end
of the free zone (fastest, occupies the free zone, vulnerable to a stray ball).

**Calibration** ✅: intrinsic once per camera/lens; extrinsic every time the
camera physically moves. A single-camera homography from the court's own painted
lines solves the **ground plane only** — it cannot recover ball height. That is
a geometric fact, not a software gap, and it is the reason §21.4 defers 3D.

⚠️ **"Who maintains it" has an honest answer: not the venue.** A school gym with
no IT staff will not notice a dead camera, a full disk or an overheating box.
Remote health monitoring and a labelled physical runbook are an ongoing
*operational* cost line, not a one-time hardware line.
---

# 14. The data asset — and the flaw in it

## 14.1 The claim, restated honestly

The previous session's headline was: *every match our scorers already collect is
labelled ground truth; start recording video alongside manual scoring
immediately and build a training set for free.*

The strategic instinct is right, and it is externally validated: ✅ **both 2025
volleyball grading papers built their training sets by aligning video to
manually scouted DataVolley files.** That is the only route anyone has found to
a volleyball corpus with per-touch grades, and VolleyVerse can generate it as a
by-product of a product it already runs.

**But the execution detail is wrong today, and it is the expensive kind of
wrong.**

## 14.2 Why today's events are not frame labels

✅ `stat_events.ts` is `timestamptz not null default now()` — the moment the row
was created, which is the moment the **operator's thumb landed**. In
`src/lib/types.ts` the in-memory `StatEvent.ts` is documented as *"epoch ms —
preserves entry order for undo"* ✅. Entry order. Not event time.

Between the ball being touched and the tap landing there is:
- human reaction and decision time;
- the ✓/O/✗ refinement step, and for duels a second prompt for the opposing
  player;
- rallies where the operator falls behind and catches up afterwards — which the
  codebase explicitly supports via `skipPhase()` and `free-rally.ts` ✅;
- undo and re-entry, which rewrites `ts` entirely.

🔢 The offset is plausibly 0.5–3 s, variable within a rally, and occasionally
several seconds when taps are batched. **Frame-level supervision needs ±1–2
frames (±33–66 ms).** Today's data is two orders of magnitude off.

## 14.3 What the existing corpus *is* good for

It is not worthless — it is just a different kind of label, and knowing which
kind determines which algorithms can consume it.

| Use | Viable today? |
|---|---|
| Frame-level supervised training | ❌ No — timestamps are entry time |
| **Rally-level ordered sequence labels** (what happened, in what order) | ✅ **Yes, and this is the valuable one** |
| **Weakly-supervised sequence alignment** (align an unordered-in-time but correctly-ordered event sequence to a video, CTC/DTW-style) | ✅ **Yes** |
| Per-player, per-match, per-season distributions for sanity-checking model output | ✅ Yes |
| Validating a rally-segmentation model's rally count | ✅ Yes |
| Training subjective grades (`DIG_SUPER`, `BLOCK_MISS`) | ⚠️ Poor — see below |

**The right technical framing is weak supervision with alignment**, not
supervised frame labelling. You know the *sequence* of contacts in a rally is
correct even when you do not know *when* each happened. That is exactly the
setting CTC-style alignment losses were invented for, and it means the existing
archive has real training value **provided you also have the video**.

⚠️ **The label-noise warning deserves its own line.** §10.5 and §10.6 identified
two events where the human ground truth is structurally unreliable:
`BLOCK_MISS`, where the app *never sees a block jumped too late* (the team's own
in-code comment ✅), and `DIG_SUPER`, where "extraordinary" is a subjective call
made in under a second. Training on these teaches a model to reproduce an
operator's omissions and inconsistencies. **Either redefine them geometrically
before modelling, or exclude them.** Do not quietly train on them and then be
surprised by a low ceiling.

## 14.4 The fix, and it is small

Three changes, all shippable this month, that make every future match a usable
training example:

1. **`video_ts_ms` and `video_asset_id` on `stat_events`** (§4.3). Nullable.
2. **A clock-sync mechanism.** The simplest robust version: when the scorer
   starts a match, the app displays a full-screen high-contrast frame carrying a
   monotonic timestamp — a digital clapperboard — which the operator points the
   camera at for two seconds. One frame of that in the recording pins the video
   clock to the app clock for the whole match. No NTP, no network, no hardware,
   works on any phone. 🔢 Accuracy is one frame.
3. **A "record while you score" affordance.** Whatever is easiest for the
   operator: a second phone on a tripod, a note in the pre-match checklist, a
   rule that the club uploads the stream they already produce.

🔢 **20–50 matches of video with aligned, ordered event sequences is a corpus
nobody else in volleyball has**, and it accrues at zero marginal cost from
matches that were going to be scored anyway. ✅ For comparison, SoccerNet's ball
action spotting task — the closest published analogue — trains on **7 annotated
games.**

## 14.5 Privacy and consent — flag for counsel now, not later

⚠️ **This is a genuine blocker that is easy to discover too late.** Recording
school and club volleyball means recording **minors**, systematically, with
identity attached (name, jersey number, per-player performance data), and
retaining it to train models.

India's Digital Personal Data Protection Act 2023 📚 imposes specific
obligations around processing children's personal data, including verifiable
parental consent and restrictions on certain kinds of processing. 📚 **The exact
obligations were not verified this session and this report is not legal
advice** — but the shape of the problem is clear enough that it should go to
counsel before the first recorded match, not after the first 50.

Practical items to settle before capture starts: who owns the footage (club,
league, or VolleyVerse); what consent the club obtains from parents and whether
it covers model training as distinct from match broadcast; retention period;
whether identity can be stripped for the training corpus while preserved in the
product; and whether a parent can require deletion of a child's data after a
model has been trained on it. None of these are hard to answer. All of them are
expensive to answer retroactively.

---

# 15. Human-in-the-loop design

## 15.1 The split

| Decision | Owner | Latency requirement | Why |
|---|---|---|---|
| **Who won the point** | **Human** | Instant, one tap | 1 bit. Never ambiguous to a person courtside. Drives the live scoreboard, the rules engine, rotation, and the entire match state |
| Rotation, serve order, front/back row, libero, side-out | **`rally.ts`** | Instant | Derived, free, already built |
| Who touched it, what action, how well | **AI** | Seconds | Latency-tolerant, repetitive, consistent — the 90% of taps that exhausts the operator |
| Low-confidence AI proposals | **Human, asynchronously** | Between rallies, or post-match | The verification queue |

This keeps the operator's input at roughly **one tap per rally** instead of
🔢 4–8, a reduction of ~80–90% in courtside workload, while preserving a
perfect live score. It is also the only configuration in which a first-generation
model is shippable at all: a wrong `RECV_GOOD` is a stat error someone can fix
later, while a wrong point is a broken match.

## 15.2 The verification queue

Everything below the confidence threshold lands as `review_status = 'pending'`.
The review UI should reuse `CourtBoard` and the existing hold-and-flick gesture
(`src/lib/gesture.ts` ✅) — the operator already knows it, and it was designed
for exactly this: fast confirmation without looking at the screen.

Three review contexts, in increasing order of value:
- **Between rallies** — the 8–20 s dead ball (§16). Confirm or correct the rally
  that just ended. Highest value, because corrections are still in the
  operator's working memory.
- **Between sets** — batch review of the set.
- **Post-match** — `/console/matches/[id]/review` already exists ✅.

## 15.3 Confidence must be calibrated, not just produced

⚠️ A raw softmax score is not a probability. If the threshold is set on
uncalibrated scores, the queue will be simultaneously too long and too leaky.
Budget for temperature scaling or isotonic regression against held-out human
labels, and **measure calibration explicitly** (reliability diagram, expected
calibration error) rather than assuming it.

The operational target is a **precision-first** operating point: it is far
better to send 40% of events to review and be right about the other 60% than to
auto-accept 90% at 85% precision. 🔢 A scorer who finds one error per set stops
trusting the system; a scorer who confirms a short queue and finds it always
right starts leaning on it.

## 15.4 The flywheel

Every human correction writes a row with `source = 'AI_CORRECTED'` and a
`video_ts_ms` ✅ — which is a **frame-accurate label on a real failure case**,
the most valuable training data that exists. The review queue is not overhead;
it is the labelling pipeline, and it is why `source` and `video_ts_ms` are
non-negotiable parts of the first migration.

---

# 16. Latency: redefining "live"

## 16.1 The published problem

✅ Strictly causal inference costs ~20 mAP points / ~29% relative (TriDet 69.3
offline → MATR 49.5 online, THUMOS14). ✅ Compute is not the constraint —
MiniROAD reaches 71.8 per-frame mAP at 0.0158 GFLOPs. The penalty is purely
about not being able to look ahead.

## 16.2 Volleyball's free lunch

**Volleyball is not a continuous-flow sport.** Between the end of one rally and
the serve of the next there is a mandatory dead-ball interval: the referee
whistles, the ball is retrieved, players rotate, the server takes position, the
referee whistles for service. 🔢 That interval runs roughly **8–20 seconds** in
normal play, and longer at timeouts, substitutions and set breaks.

Nobody watching a volleyball match needs to know who dug the ball 200 ms after
they dug it. They need the **score** instantly — which the human provides (§15.1)
— and they need the detail **before the next rally starts**, so the timeline
reads correctly and the operator can confirm it while it is fresh.

**So the real latency budget is not 200 ms. It is ~5 seconds**, with a hard
ceiling at the next serve.

## 16.3 What that buys

🔢 A 5-second budget for a 5–10 second rally means the system can:
- use a **bidirectional** temporal model over the complete rally rather than a
  causal one — recovering most or all of the ~29% penalty;
- run the full constrained-decode beam search over the rally (§12.3), which
  requires the whole sequence by definition;
- fuse audio and video with a proper alignment pass rather than streaming
  heuristics;
- batch inference at the rally level, which is far more GPU-efficient than
  per-frame streaming.

⚠️ **This is an engineering hypothesis, not a published result** — §8.9 notes
that no paper measuring the accuracy-vs-lookahead curve for precise spotting
could be found. But the direction is well supported: E2E-Spot and T-DEED, the
two strongest precise-spotting methods, are *already* bidirectional over short
windows rather than full-video, which is structurally the same trade.

## 16.4 The latency budget in practice

| Stage | Realistic | Note |
|---|---|---|
| Rally end detected (whistle) | ~0.1–0.3 s | Audio, cheap |
| Video buffered and encoded | 0.1–0.5 s | ✅ |
| Transport to compute | 0.05–2 s | LAN edge vs cloud round trip |
| Inference over the whole rally | 1–3 s | Bidirectional, batched |
| Constrained decode | <0.1 s | `rally.ts` is pure and fast |
| Write + realtime push to clients | 0.1–0.5 s | Existing `pushLiveState` path ✅ |
| **Total** | **~2–6 s after rally end** | Comfortably inside the dead ball |

This reframing is, in engineering terms, the cheapest large win available in the
whole project. **It does not require a better model. It requires not
over-specifying the requirement.**

---

# 17. Schema and API changes

## 17.1 Migration 1 — provenance (ship this month, before any CV work)

```sql
alter table stat_events
  add column source        text,      -- HUMAN | AI | AI_CONFIRMED | AI_CORRECTED
  add column confidence    real,      -- calibrated [0,1]
  add column rally_id      uuid,
  add column video_ts_ms   bigint,
  add column video_asset_id uuid,
  add column review_status text,      -- pending | accepted | rejected
  add column model_version text;

create index stat_events_review_idx on stat_events (match_id, review_status);
create index stat_events_rally_idx  on stat_events (match_id, rally_id);
```

All nullable. Existing rows and existing code are unaffected — exactly the
pattern `2026-08-12-attack-detail-and-duels.sql` established ✅.

⚠️ **Run the migration before deploying code that writes the new values.** The
in-repo warning is specific and worth repeating: the optimistic UI shows the tap
landing while the database silently refuses it inside the offline queue ✅. A
silent write failure in an event-sourced system is the worst possible failure
mode, because the derived views will simply be wrong and nothing will alarm.

## 17.2 Migration 2 — the AI identity

An `ai_producer` role in `user_roles`, with a write policy scoped to
`stat_events` for matches it is explicitly assigned. ⚠️ Not the service-role
key, and not `anon` (which the 2026-08-11 migration opened and the 2026-08-31
migration closed again ✅).

## 17.3 The ingestion contract

One endpoint. It accepts a **proposed rally**, never a loose event:

```
POST /api/ai/rally
{
  matchId, setNo, rallyId,
  videoAssetId, modelVersion,
  events: [
    { clientId, teamId, playerId, type, vsPlayerId?,
      confidence, videoTsMs, courtXy? }
  ]
}
```

Server-side it must:
1. Replay the rally against `rally.ts` from the current match state. **Reject
   the whole rally if the sequence is illegal** — do not write a partial rally.
2. Write legal events with `source = 'AI'`, setting
   `review_status = 'pending'` for anything below the confidence threshold.
3. Be idempotent on `clientId` so a retry after a network failure is safe. The
   existing offline-queue design already assumes client-minted ids ✅.

**Atomicity at the rally level is the important property.** A half-written rally
corrupts the rotation state, and because everything downstream is a derived
view, that corruption propagates silently into standings.

## 17.4 What deliberately does not change

`pushLiveState` and the Realtime subscription keep owning the scoreboard ✅. The
human scorer's path is untouched. `DataProvider` gains no new methods — the AI
writes through the ingestion endpoint, not through the client interface. **If a
change is needed to `repository.ts`, something has gone wrong with the design.**

---

# 18. Cost model

🔢 All figures are order-of-magnitude planning numbers built from the verified
component prices in §13, not quotes. ₹ conversions at an assumed ~₹88/USD —
confirm the live rate and budget separately for Indian customs duty and GST on
imported hardware.

## 18.1 What the first six months actually cost

| Item | Cost | Note |
|---|---|---|
| **VolleyStation 10-analysis pack** | **$250** ✅ | The single highest-information purchase available (§21.1) |
| Hudl Assist club-tier trial | 🚫 not public | Demo-gated; worth requesting |
| Path A capture (phone + tall tripod) | ~$50 | Tripod only; phone is BYO |
| Path B capture (global-shutter A/B) | ~$1,000–1,400 | ✅ BFS-PGE-19S4C-C $655 + lens + PoE/12V + mini-PC |
| USB microphone | ~$20–50 | For the audio experiment |
| Cloud GPU for experiments | ~$100–300 | ✅ g4dn.xlarge $0.526/hr |
| Labelling ~2,000 contact instants | 🔢 ~40–80 person-hours | Can be done by the same scorers, in the review UI |
| **Total hardware + external spend** | 🔢 **~$1,500–2,000** | Everything else is engineering time |

**That is the entire capital requirement to answer every open question in this
report.** It is worth stating plainly, because the instinct in camera projects
is to buy the rig first.

## 18.2 Per-match running cost at scale

🔢 Against ✅ AWS g4dn.xlarge at $0.526/hr: a 90-minute match processed at ~3×
real-time on one camera is ~30 minutes of GPU time ≈ **$0.26**. At 1× on a
heavier model, ~**$0.79**. Even at four cameras and 1×, ~**$3.20/match**.

✅ VolleyStation charges **$25–30/match**. 🔢 **Compute is ~1–10% of a
competitor's price point.** The cost of this business is engineering, human
review, and sales — not inference. Do not spend early effort optimising GPU
cost; spend it on accuracy and on the review loop.

## 18.3 Per-venue hardware, if and when it is needed

From `05-hardware-and-edge-compute.md`, 🔢 approximate BOM:

| Config | Cost | When |
|---|---|---|
| A — 1 fixed camera + small edge box | **$1,000–2,100** | Where the MVP lives if path B wins the A/B |
| B — 2 cameras | $2,300–3,960 | First point at which triangulated ball height is possible at all |
| C — 4+ cameras | $4,900–8,000 | Full-court 3D. ⚠️ *"this is where a volunteer setup genuinely struggles"* |
| D — multi-camera + dedicated net camera | $5,800–10,800 | Officiating-adjacent. Out of scope |

⚠️ Add for every deployment, and these are the lines that get forgotten:
mounting and facility permission, recalibration on every move, remote health
monitoring, and a support path when a volunteer unplugs something. §13.5: the
venue will not maintain it.

---

# 19. Risks

## 19.1 Technical

| Risk | Severity | Evidence | Mitigation |
|---|---|---|---|
| **Ball tracking collapses in a real gym** | **High** | ✅ 88.0 F1 on benchmark clips vs 54.2% on real single-camera match footage | Measure it in week 1 (§21.2). This is the kill-criterion experiment |
| **Contact detection does not reach usable accuracy** | **High** | ✅ 48.9% single-camera, 25 fps; ❌ no volleyball paper exists | Audio fusion (§11.1); ≥60 fps capture; accept human confirmation for contacts |
| Player attribution fails through net occlusion | Medium-High | ✅ 5–8.7% jersey legibility; ✅ AssA 66.8 on volleyball MOT | Rally-anchored constrained assignment (§11.3), not per-frame OCR |
| Grades don't match human labels | Medium | Only reception is attempted in the literature | Redefine geometrically (§10.2, §10.6); accept divergence deliberately |
| Live latency unachievable | Low | ✅ ~29% causal penalty | Redefine "live" as the inter-rally window (§16). Largely dissolved |
| Cascade error compounding | Medium | ✅ GSR baseline 22.26 from modules scoring 50–92 | Re-ground at every rally boundary; constrained decode (§12.3) |

## 19.2 Competitive

| Risk | Severity | Note |
|---|---|---|
| **VolleyStation's VS.AI is already good** | **High** | ✅ Shipping, $25–30/match, claims automatic player ID + skill recognition. **Unknown quality.** §21.1 resolves this for $250 |
| Genius Sports ports its basketball playbook to volleyball | Medium-High | ✅ They own DataVolley, VideoCheck rigs on pro courts, *and* a proven camera+AI+real-time+human-hybrid system at 1,000+ orgs — applied to basketball and soccer, not volleyball. They have the blueprint and could enter any time |
| Hudl extends Assist AI to live | Medium | ✅ Two-plus years post-acquisition they remain post-match and club-tier, and still human-code college. Slower than feared |
| Someone open-sources it first | Low-Medium | ✅ Several 0–1★ volleyball CV repos pushed within two weeks of the last scan. Signal, not threat |

## 19.3 Legal and licensing

| Risk | Severity | Action |
|---|---|---|
| **Recording minors** (DPDP Act 2023 and equivalents) | **High** | 📚 §14.5. Counsel **before** the first recorded match |
| Ultralytics AGPL triggers on SaaS delivery | Medium-High | ✅ Standardise on RT-DETR / YOLOX / RTMDet (Apache-2.0) from day one; keep Ultralytics to prototyping |
| GPL contamination from reference repos | Medium | ✅ `volleyball_analytics` GPL-2.0, `sn-gamestate` GPL-3.0 — read, do not vendor |
| Training data licences | Medium | ✅ MultiSports is CC BY-NC 4.0; the Volleyball Dataset's *data* licence is unstated and derives from Olympic broadcast footage; ~300 Roboflow volleyball datasets each set their own terms |
| Hikvision/Dahua procurement in public-sector venues | Medium | ✅ India banned Chinese surveillance equipment in government/public-sector projects, April 2026. Verify the notification text before specifying |

## 19.4 Operational

⚠️ **The venue will not maintain the system.** A school gym with no IT staff
will not notice a dead camera, a full disk or an overheating box. Remote health
monitoring (camera heartbeat, disk, temperature) plus a labelled physical
runbook a volunteer can follow is an ongoing cost line from day one, not a
phase-2 nicety.

⚠️ Gyms are frequently not climate-controlled — especially in India, where
equipment rooms can run hot for months. A fanless embedded box tolerates this;
a gaming-GPU mini-PC in an unventilated cupboard will throttle. ⚠️ The compute
box is the most attractive theft target in the rig and belongs in a lockable
enclosure. ⚠️ Portable rigs need extrinsic recalibration at every setup, and a
volunteer setting the angle wrong silently degrades accuracy with no alarm.

## 19.5 The risk of succeeding at the wrong thing

Worth naming. The most likely failure mode for this project is **not** that the
CV doesn't work. It is building a technically impressive system that produces
events nobody trusts, because the accuracy is 85% and a coach who finds one
wrong attribution stops believing the other 99. 🔢 Precision-first thresholds
and a visible review queue (§15.3) are not UX polish — they are what determines
whether the output gets used at all.

---

# 20. Success metrics and kill criteria

**Define these before the experiments run, not after.** The value of a
kill criterion is entirely in having agreed it in advance.

## 20.1 Experiment-level gates

| Experiment | Success | Investigate further | **Kill / rethink** |
|---|---|---|---|
| **E1 — Ball tracking on your gym footage** (500 hand-labelled frames, τ=4 px) | F1 ≥ 0.80 | 0.60–0.80 | **F1 < 0.60** → capture spec is wrong. Fix lighting/fps/placement before any model work |
| **E2 — Audio rally segmentation** (whistle → rally boundaries) | ≥95% of rallies bounded within ±1 s | 80–95% | < 80% → audio may still work for contacts; drop it as the rally clock |
| **E2b — Audio contact transients** | Detects ≥70% of contacts, <30% false positives | 50–70% | < 50% in a single-court hall → drop audio contacts, rely on trajectory |
| **E3 — Rally segmentation (fused)** | ≥98% of rallies correctly bounded | 90–98% | < 90% → nothing downstream can work; rally is the atomic unit |
| **E4 — Serve events end-to-end** | ≥90% agreement with human labels on `SERVE_IN`/`SERVE_ERR` | 75–90% | **< 75%** → if the *easiest* event with a **free actor** can't reach 75%, the whole event-detection thesis is in doubt |
| **E5 — Competitor benchmark** (VolleyStation output vs your ground truth) | — | — | Not a gate — an input to strategy. See §21.1 |

**E4 is the real go/no-go.** The serve has a free actor, a known time, a
geometric outcome and no occlusion. If it cannot clear 75% agreement, no other
event will, and the honest conclusion is to stop building a detector and
reconsider — possibly as a licensee or partner rather than a builder.

## 20.2 Product-level metrics, once something ships

| Metric | Target | Why |
|---|---|---|
| **Operator taps per rally** | ≤ 1.5 (from 🔢 4–8) | The actual value proposition |
| **Precision on auto-accepted events** | ≥ 97% | §19.5 — trust is binary and fragile |
| Recall at the auto-accept threshold | ≥ 50% initially | Low recall is fine; low precision is fatal |
| Review queue length per set | ≤ 15 items | Must be clearable in a set break |
| Calibration error (ECE) | ≤ 0.05 | The threshold is meaningless otherwise |
| End-to-end latency, rally end → events visible | ≤ 6 s | §16 — inside the dead ball |
| Matches with aligned video + events | 🔢 50 by month 6 | The corpus is the moat |

## 20.3 Strategic kill criteria

Stop or fundamentally rethink if any of these become true:

- **E4 fails** (< 75% on serve events).
- **VolleyStation's output turns out to be genuinely good** *and* they move to
  live. Then the correct move is partnership, licensing or differentiation on
  league management — not a head-on CV race against a funded incumbent with FIVB
  relationships.
- **Legal counsel says recording minors for model training is impractical**
  under DPDP without consent machinery the clubs will not adopt. Without the
  corpus there is no moat, and the project becomes an integration play.
- **Twelve months pass without a single match scored primarily by AI.** Not a
  technical criterion — an honesty criterion.

---

# 21. The plan — what I would actually build, starting Monday

*Written as if I were the R&D engineer responsible for this, with one engineer,
a small budget, and an obligation to have something real in six months.*

## 21.1 Week 1 — spend $250 before writing a line of code

**Buy VolleyStation's 1,000-token pack ($250, 10 analyses, first match free).**
✅ Upload ten matches for which VolleyVerse already holds a complete human event
log. Export their output. Align it to your own `stat_events` and compute, per
event type, the agreement rate.

This is the highest-information action available in the entire project, and the
reasoning is worth spelling out:

- It produces **the only real accuracy number for a shipping volleyball CV
  product that exists anywhere** — none of the eight vendors publish one ✅.
- If their output is good, you have learned that the post-match problem is
  solved by a competitor at $25/match, and your differentiation must be *live*,
  *league management*, or *partnership* — a strategy-defining fact.
- If their output is poor, you have learned that "Automatic Player
  Identification" and "Automatic Skill Recognition" ⚠️ are marketing over a
  weak system, that the bar is lower than feared, and roughly which event types
  break — which tells you where to aim.
- Either way it costs **$250 and one week**, versus roughly six months to reach
  the same conclusion by building.

In parallel, request a Hudl Assist club-tier demo and, if a trial is available,
run the same comparison. ✅ Their volleyball AI is club-tier and post-match.

**Also in week 1, and this one is not optional:** send the §14.5 privacy
question to counsel. Recording minors systematically, with identity attached,
for model training, is a decision with a long lead time and it gates everything
in §21.3.

## 21.2 Weeks 1–3 — two experiments and one migration, in parallel

### Ship the provenance migration (§17.1)

Seven nullable columns, two indexes, an `ai_producer` role. It touches nothing
existing, it takes a day, and **every match scored after it lands is a
potentially usable training example** while every match before it is not. Ship
it first for that reason alone.

Alongside it, the **digital clapperboard** (§14.4): a full-screen high-contrast
timestamp the operator points a camera at for two seconds at match start. One
frame of it in the recording pins the video clock to the app clock. No NTP, no
hardware, works on any phone, accurate to one frame.

And a **"record while you score"** prompt in the pre-match checklist. 🔢 The
target is 50 matches of video with aligned ordered event sequences by month 6;
✅ SoccerNet's closest analogous task trains on 7 games.

### E1 — Ball tracking on your own footage

Clone ✅ `nttcom/WASB-SBDT` (MIT, **pre-trained volleyball weights**) and ✅
`asigatchov/fast-volleyball-tracking-inference` (MIT, ⚠️ author-reported F1
0.902 at 149.6 CPU FPS). Hand-label 500 frames from one of your own gym
recordings. Run both. Measure F1 at τ = 4 px.

**This is the single most informative technical experiment available**, because
the answer is already bracketed by two verified numbers that are 34 points
apart: ✅ **88.0** on curated benchmark clips, ✅ **54.2%** on real single-camera
match footage. Where your gym lands between those tells you whether the capture
spec needs fixing before any modelling begins. Gate: §20.1.

### E2 — Audio

A USB microphone and a day's work. Two detectors:
- **Whistle:** band-limited energy + onset detection, ~2–4 kHz. Validate
  against rally boundaries reconstructed from the event log.
- **Contact transients:** onset detection with attack-time and spectral
  features. Validate against contact counts per rally from the event log.

🔢 Low cost, no labelling required beyond what the event log already gives you,
and it attacks §9.1 — the hardest problem — with a modality that is immune to
occlusion, lighting and motion blur. ❌ Nobody has published this for
volleyball. ⚠️ It may fail outright in a multi-court hall; that is exactly why it
should be tested in week 2 rather than month 5.

## 21.3 Weeks 4–12 — build the serve, and only the serve

### The hardware A/B (§13.2)

| Path | Kit | Cost |
|---|---|---|
| **A** | Phone or action cam on a tall tripod **behind the baseline** | **$0** + tripod |
| **B** | ✅ FLIR BFS-PGE-19S4C-C (2 MP global shutter @ 60 fps, **$655**) + fast lens + PoE injector/12 V + mini-PC | ~$1,000–1,400 |

Record the same rallies on both. Measure the ball-tracking F1 delta. ⚠️ Note
that path A is *precisely the rig VolleyStation instructs its own customers to
use* ✅ — a competitor with every incentive to demand better hardware says a
phone behind the baseline works. If the delta is small, the BOM collapses to
nothing and the product ships on phones.

### E3 — Rally segmentation

Fuse whistle detection with ball-motion energy to emit rally start/end. Validate
against the event log's reconstructed rally boundaries.

**This is also the first shippable feature**, independent of any event
detection: automatic per-rally clipping. Coaches want rally clips. It is useful
on its own, it proves the capture and sync plumbing end to end, and it produces
the segmentation every later stage depends on.

### E4 — Serve events, end to end

Build the narrowest possible vertical slice:

1. Rally start from E3.
2. **Actor: free.** ⚫ P1 of the serving side, straight from `rally.ts`.
3. Ball track from E1 through the serve flight.
4. Court homography from the painted lines (one-time; the court is a rectangle
   with known dimensions and high-contrast markings).
5. Landing point → `SERVE_IN` / `SERVE_ERR` geometrically.
6. `SERVE_ACE` only when the receiving side's tracks show no plausible contact
   **and** audio shows no transient — otherwise route to human review.
7. Propose the rally to the ingestion endpoint (§17.3); `rally.ts` validates;
   write with `source='AI'`, `confidence`, `video_ts_ms`.
8. Low confidence → `review_status='pending'` → the hold-and-flick review queue.

**Why the serve and nothing else:** it is the one event where the hardest
published problem — actor identity — is *free*, the timing is known, the outcome
is geometric, and there is no occlusion. 🔢 It is also 10–15% of all events. If
this does not clear 75% agreement (§20.1), the thesis is in trouble and you
have learned that in three months for under $2,000.

### The stack

| Layer | Choice | Licence |
|---|---|---|
| Ball tracking | WASB, then VballNet | ✅ MIT |
| Detector | RT-DETR (**not** Ultralytics) | ✅ Apache-2.0 |
| Tracking | `roboflow/trackers` (**not** boxmot) | ✅ Apache-2.0 |
| Pose (later) | RTMPose / RTMO via mmpose | ✅ Apache-2.0 |
| Glue | `roboflow/supervision` | ✅ MIT |
| Runtime | ONNX Runtime; OpenVINO on CPU; TensorRT on GPU | ✅ MIT / Apache-2.0 |
| Compute | Cloud batch (g4dn/g6) — **no gym-side box yet** | — |
| Rules / decode | `src/lib/rally.ts`, unchanged | in-house |

⚠️ **Start commercially clean.** ✅ Ultralytics' AGPL-3.0 triggers on SaaS
delivery itself, not redistribution. Choosing RT-DETR in week 4 costs nothing;
retrofitting it in month 18 costs a rewrite.

## 21.4 What I would deliberately NOT try to solve yet

This list is as important as the build plan, and each item has a reason rather
than a shrug.

| Not now | Why not |
|---|---|
| **3D ball position / `court_x,y,z`** | ✅ Needs 4 calibrated cameras (the one verified volleyball system that works used 4 corner cameras). ✅ Monocular alternatives give 1.8–2.3 m error, or 8 cm only in simulation with priors that don't transfer to volleyball's floor-to-3.2 m contact range. And there is **no consumer for spatial data in the current analytics** — it is a product feature, not an integration detail |
| **`FAULT_NET`, `FAULT_DOUBLE`** | ⚠️ This is what VideoCheck's up-to-23-camera, up-to-200 fps rig exists for ✅. Three orders of magnitude from one 60 fps camera. Revisit via referee-signal recognition (§10.7), not contact detection |
| **`SPIKE_TOOL`, `SPIKE_BLOCKED`, `BLOCK_TOOLED`, `vs_player_id`** | Fingertip deflections at peak occlusion. ✅ Occluding players corrupt each other's identification. Lowest feasibility, modest event volume |
| **`DIG_SUPER`, `BLOCK_MISS`** | Not a detection problem — a **label** problem. ✅ The team's own code notes the app never sees a block jumped too late; "extraordinary" is a sub-second subjective call. Redefine before modelling, or exclude |
| **Jersey-number OCR as the identity mechanism** | ✅ 5–8.7% legibility. Use rally-anchored constrained assignment (§11.3) and treat OCR as confirmation only |
| **Officiating / line calls** | Different accuracy bar (adversarial, disputed), different liability, and ✅ Hawk-Eye already owns it with an FIVB relationship and a confirmed "VolleyTrack" |
| **Sub-second live latency** | ✅ ~29% relative accuracy cost for causal inference, in exchange for a requirement nobody has. §16 |
| **Gym-side edge compute** | Adds theft, heat, maintenance, remote-support and calibration-drift problems ✅ before you know whether the model works. Cloud batch first |
| **Multi-camera and portable calibration** | ⚠️ 4-camera extrinsic calibration is *"the most failure-prone step in the whole system"* and *"where a volunteer setup genuinely struggles"*. One gym, one camera, fixed |
| **Wearables** | ⚠️ Official-competition legality genuinely unverified; says nothing about the ball; charging logistics a no-IT-staff venue cannot absorb |
| **Beach volleyball** | ✅ The one substantial paper reports 54.2% ball tracking. Different sport, different problem |
| **Removing the human scorer** | The human keeps a 1-bit, zero-latency, always-correct decision. Nothing about that is worth automating |

## 21.5 Months 4–6

Assuming E1–E4 pass their gates:

1. **Extend to reception**, using the geometric grade reframing (§10.2) — ball
   arrival point relative to the known setter position. ✅ Externally validated
   as the right starting point by both 2025 grading papers.
2. **Extend to attack outcomes** (`SPIKE_IN`/`ERR`/`POINT`) — geometric, and
   ✅ the spike is the most visually distinct action in the sport.
3. **Build the constrained decoder** (§12.3): beam search over candidate rally
   sequences, pruned by `rally.ts`. This is the piece that turns three mediocre
   classifiers into one usable system and it is the part nobody can copy quickly.
4. **Instrument the review queue as a labelling pipeline** — `AI_CORRECTED`
   rows with `video_ts_ms` are frame-accurate labels on real failure cases.
5. **Hit 50 aligned matches.** ✅ The closest published benchmark task has 7.

## 21.6 The one-paragraph answer

**Build the serve, on a phone, with audio, validated by the rules engine.**
Spend $250 on a competitor's output before spending anything on cameras, and
ship the provenance migration and video-clock sync this month so that every
match from now on becomes training data. Test exactly two hardware paths — a
phone behind the baseline versus one global-shutter camera at 60 fps — and let a
measured ball-tracking F1 delta settle the question instead of an argument.
Detect rally boundaries first (whistle plus ball motion), then serves, because
the serve is the only event where the actor is free, the time is known and the
outcome is pure geometry. Use MIT/Apache components only, keep compute in the
cloud, keep the human on the point, and give the AI the detail. And do not touch
3D ball position, net touches, double contacts, block deflections, subjective
grades, jersey OCR, officiating, or edge boxes until something simpler has
demonstrably worked — because ✅ the best-resourced companies in this market have
been trying for years and still ship post-match, club-tier, human-assisted
products, and the reason is that every one of those problems is harder than it
looks.

---

# Appendix A — Research methodology

**Sessions.** This report consolidates a previous research session (4 surviving
files: codebase analysis, commercial landscape, open-source scan, hardware) and
a new session (12 Sept 2026) that rebuilt the academic literature review from
scratch, re-verified the commercial landscape with a JavaScript-capable browser,
and re-inspected the VolleyVerse repository directly.

**Verification methods used.** Direct page fetch; a JavaScript-rendering browser
for SPA sites that defeated plain fetching; the GitHub REST API for repository
licences, stars and push dates; the Crossref REST API for bibliographic records;
CVF Open Access, arXiv `/abs/`, BMVC proceedings, J-STAGE, MDPI, PLOS and
SciTePress for papers; and `git clone` plus direct file inspection for the
VolleyVerse codebase.

**What did not work.** ❌ WebSearch returned HTTP 403 on every query in both
sessions — an organisation egress policy, not a retryable error. ❌ Semantic
Scholar and OpenAlex APIs returned HTTP 429 on every attempt. ❌ arXiv's search
endpoints, `paperswithcode.com` and `dblp` are robots-disallowed. ❌ CVF Open
Access began returning 403 partway through one pass.

**Consequence, stated plainly.** The literature review is **recall-limited**. It
was assembled by named-paper lookup, Crossref bibliographic queries and
proceedings indices rather than by keyword search. There may be 2024–2026 work,
particularly volleyball-specific work, that simply could not be surfaced. Treat
the academic sections as a floor, not a census.

---

# Appendix B — What this report could not verify

Carried forward deliberately, so that nothing here hardens into assumed fact.

**Commercial**
- ⚠️ No vendor in this market publishes a detection accuracy figure. Every
  capability claim in §6 is vendor self-reported; there are no third-party
  benchmarks, reviews or independent comparisons in this report.
- ❌ VolleyStation's "Automatic Skill Recognition" is never enumerated — which
  skills, which outcome codes, whether attack outcomes are classified. §21.1
  resolves this empirically.
- ❌ Whether Pixellot's VidSwap 4–8 hour breakdown is fully automatic or
  human-assisted. The page genuinely supports both readings.
- ❌ Hawk-Eye's "VolleyTrack" exists as a name only — no product page, no spec,
  no FIVB deployment detail, no statement of scope.
- ⚠️ Veo Go's exact price renders ambiguously (20 / 29 / 39 EUR by term). Use
  "from €20/month".
- 📚 Catapult's volleyball claims were not re-checked this session.

**Academic**
- ⚠️ Volleyball Dataset statistics conflict between the GitHub repository
  (4,830 frames / 55 videos / 8+9 labels) and the CVPR 2016 paper page (1,525
  frames / 15 videos / 6+7 labels / 51.1%). Almost certainly two releases.
- ❌ No current (2024–2026) SOTA figure on either the Volleyball Dataset or
  MultiSports could be verified. The newest verified figures are 2021–2022.
- ⚠️ One verification pass could not confirm volleyball is among MultiSports'
  four sports; another read the 12-class volleyball list from the ICCV PDF.
- ❌ No accuracy figure from either 2025 volleyball grading paper (Nako et al.;
  Mori & Sawano) could be read — metadata confirmed, results tables not.
- ⚠️ The Waseda "99.23% 3D tracking success" figure has no retrievable metric
  error threshold. Do not read it as "99% of frames within X cm".
- ⚠️ TrackNetV2's own reported metrics were never read directly; all V2 numbers
  quoted here come from third parties who disagree with each other
  substantially. TrackNetV4 is a non-peer-reviewed "research report" with no
  numbers in its abstract.
- ❌ No published measurement of the accuracy-vs-lookahead curve for precise
  event spotting — §16's central hypothesis is therefore an engineering
  argument, not a cited result.
- ❌ Frame-level jersey legibility rates for volleyball (or soccer). Only the
  hockey figures (5.0 / 6.5 / 8.7%) are verified.
- ❌ The proportion of SoccerNet jersey tracklets labelled "−1" — without it the
  92.85% headline cannot be decomposed into "read the number" versus "correctly
  said not visible".
- ⚠️ A numeric conflict between MAT's and MiniROAD's THUMOS tables was flagged
  rather than silently reconciled; the two-source-agreement figures were used.
- ⚠️ The Volleyball Dataset's *data* licence is unstated (the repo's BSD-2-Clause
  covers code); the footage derives from Olympic broadcasts. **A live legal
  question for commercial use.**

**Hardware**
- 📚 GoPro, Insta360 and DJI prices could not be scraped from JS storefronts.
- ⚠️ Veo/Hudl/Pixellot/Spiideo install pricing ranges are market colour, not
  vendor-confirmed. Do not quote externally.
- ⚠️ Coral TPU availability status is uncertain — `coral.ai/products` now
  redirects with no clear availability statement.
- ❌ Scoreboard data-feed integration (Daktronics et al.) has no self-serve
  public API for small venues. OCR on a camera pointed at the scoreboard is the
  realistic path.
- ⚠️ Whether players may legally wear tracking sensors in official FIVB/NCAA
  competition — four rulebook fetch attempts failed. Genuinely open.
- ⚠️ ₹88/USD is a ballpark, not a live FX rate; Indian customs duty and GST on
  imported hardware are not modelled.

**Legal**
- 📚 India's DPDP Act 2023 obligations around children's data were not verified
  in detail. §14.5 is a flag for counsel, **not legal advice.**

---

# Appendix C — Anchor files if the repo moves on

All integration analysis is anchored to these files at commit `162b3f5`
(31 Aug 2026). Re-read them before acting on §4, §5 or §17.

| File | Why |
|---|---|
| `supabase/schema.sql` | The relational model, the 25-type `CHECK`, and RLS |
| `src/lib/types.ts` | `EventType`, `StatEvent`, `Match`, `Player` |
| `src/lib/rally.ts` | The FIVB state machine — the constraint engine |
| `src/lib/repository.ts` | `DataProvider.addEvent` — the injection seam |
| `src/lib/substitution.ts` | Libero and rotation rules |
| `src/lib/providers/live-state.ts` | The live broadcast channel |
| `src/lib/analytics/volleyball.ts` | How stats are derived; the double-scoring exclusions |
| `src/app/console/matches/[id]/rally/page.tsx` | The manual workflow being replaced |
| `CLAUDE.md`, `REALTIME_SYNC.md` | The team's own architecture docs |

**Verified current this session:** 22,403 lines TypeScript; 25 event types (not
26 — §4.2); 5 migrations through `2026-08-31-user-roles-and-admin-panel.sql`; no
provenance, confidence, rally, video or spatial columns present.

---

# Appendix D — Companion research files

| File | Status |
|---|---|
| `01-volleyverse-codebase-analysis.md` | ✅ Re-verified current. One correction: 25 event types, not 26 |
| `02-commercial-landscape.md` | ⚠️ Superseded in part by §6 — VolleyStation, Veo pricing and Hawk-Eye VolleyTrack are all materially corrected |
| `03-opensource-github.md` | ✅ Current. Licences and landmines unchanged |
| `04-academic-literature.md` | **Rebuilt this session** — the original was missing from disk |
| `05-hardware-and-edge-compute.md` | ✅ Current |
| `06-findings-so-far.md` | ❌ Missing from disk. Its synthesis is reconstructed in §5, §9, §10, §12 and §15 of this report |

---

*Prepared 12 September 2026. Every accuracy figure in this document was measured
by someone else, on someone else's footage, under conditions stated where known.
None of them were measured on VolleyVerse's own data, because that data does not
exist yet. §21.2 is the fastest route to changing that, and until it is done,
every number here is a prior and not a prediction.*
