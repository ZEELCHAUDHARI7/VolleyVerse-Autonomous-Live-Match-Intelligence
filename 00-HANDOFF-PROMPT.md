# HANDOFF PROMPT — paste this into the new account/session

> Copy everything between the `---` lines below and paste it as your first message
> in the new session. Make sure `C:\R&D` is connected as a folder first
> (Claude desktop app → Add folder), and attach the same Project if you can.

---

I am continuing an R&D investigation for **VolleyVerse** (repo:
https://github.com/ZEELCHAUDHARI7/volleyverse). A previous session already
completed the research phase and saved everything to my `C:\R&D` folder, in
`C:\R&D\volleyverse-rnd\`.

**Read these files first, in this order, before doing anything else:**

1. `01-volleyverse-codebase-analysis.md` — full teardown of our existing
   product: architecture, Supabase schema, the `StatEvent` model, the FIVB
   rules engine, live-state sync, and the exact integration seams where an
   AI/CV system could inject events. **This is the most important file.**
2. `02-commercial-landscape.md` — competitors and existing products
   (DataVolley, VolleyStation, Hudl/Balltime, Volleymetrics, Spiideo,
   Pixellot, Veo, Genius Sports, Hawk-Eye, Catapult, KINEXON + tennis/basketball
   proxies).
3. `03-opensource-github.md` — GitHub/open-source projects, with licences and
   maturity verdicts, plus licensing landmines.
4. `04-academic-literature.md` — research papers, with what is solved /
   partially solved / open.
5. `05-hardware-and-edge-compute.md` — cameras, edge compute, sensors, venue
   reality, BOM cost tables for setups A–G.
6. `06-findings-so-far.md` — the synthesis that was in progress when the
   previous session ended: event-by-event feasibility ratings, the proposed
   pipeline, the human-in-the-loop design, and the MVP direction.

**The context you need to know:**

Our current VolleyVerse product is a live volleyball league-management
platform where a human operator sits courtside and taps buttons to record
every contact of every rally. We want to know whether cameras + AI installed
at a court could automatically generate those same structured events in real
or near-real time, and feed them into VolleyVerse.

The previous session's headline conclusions (verify these, don't just accept
them):

- VolleyVerse's event-sourced architecture (`stat_events` append-only + all
  statistics as derived views) means an AI event producer can plug in
  **without redesigning the product** — it is an additive change at the
  `DataProvider.addEvent` seam plus a few new columns.
- A large part of our data model is **derivable, not detectable** — rotation,
  libero swaps, serve order, front/back row, side-out vs break point all fall
  out of the existing FIVB rules engine. The CV job is smaller than it looks,
  and the rules engine can act as a constraint/prior on the AI's output.
- **Nobody has shipped live, fully-automatic, structured volleyball event
  detection commercially.** Hudl (which acquired Balltime) ships only a
  post-match, club-tier "breakdown". That is evidence the problem is genuinely
  hard, not merely unattempted.
- The recommended MVP split is: **keep the human on the instant 1-bit decision
  (who won the point) and give the AI the latency-tolerant detail work (who did
  what, and how)** — not full autonomy.
- Our real unfair advantage is that **every match our scorers already collect
  is labelled ground truth**. We should start recording video alongside manual
  scoring immediately, on any phone, to build a training set for free.

**What I want you to do now:**

Pick up from `06-findings-so-far.md` and produce the final, complete R&D report
covering all 21 sections listed at the end of that file, ending with a very
practical answer to: *if you were the R&D engineer responsible for VolleyVerse
and had to start experimenting next week, exactly what would you build first,
what hardware would you test, what data would you try to detect, what
technology would you use, and what would you deliberately NOT try to solve
yet?*

Be skeptical, technical and honest. Do not hallucinate products, papers,
repos, pricing or accuracy numbers. Where the saved research marks something
"unverified" or "vendor claim", keep that caveat in the final report. Save the
finished report into `C:\R&D\volleyverse-rnd\` as
`VolleyVerse-RnD-Report.md`, and also publish it as an artifact I can share
with the team.

Note: web search was blocked in the previous session (HTTP 403) — all research
was done via direct page fetches. If web search works for you, use it to
re-verify the pricing figures and the items flagged as unverified.

---

## Notes for whoever reads this

- Research date: **12 September 2026**.
- The repo was inspected at whatever `main` was on that date. If it has moved
  on, re-check `supabase/schema.sql`, `src/lib/types.ts`, `src/lib/rally.ts`
  and `src/lib/repository.ts` — those four files are where all the integration
  analysis is anchored.
- Every research file carries its own methodology note about what was and
  wasn't verifiable. Trust the ✅/⚠️/📚 markers.
