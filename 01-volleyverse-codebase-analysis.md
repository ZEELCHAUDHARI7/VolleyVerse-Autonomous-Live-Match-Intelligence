# VolleyVerse — Codebase & Data-Model Analysis (STEP 1)

Source: https://github.com/ZEELCHAUDHARI7/volleyverse — inspected 12 Sept 2026.
~22,400 lines of TypeScript across `src/`.

---

## 1. Tech stack

| Layer | Choice |
|---|---|
| Framework | Next.js 15 (App Router) + React 19 + TypeScript 5 |
| Styling | Tailwind v4 (`@tailwindcss/postcss`) |
| Charts | Recharts 2.15 |
| Motion | GSAP 3.15, Lenis (smooth scroll) |
| Backend | Supabase (`@supabase/supabase-js` 2.110, `@supabase/ssr` 0.12) — **optional** |
| Tests | Plain Node scripts with `node:assert/strict`, run via `node --experimental-strip-types *.test.mjs`. No Jest/Vitest, no lint script. |
| Build gate | `npm run build` is the closest thing to a typecheck gate |

Key operational fact: **with no `NEXT_PUBLIC_SUPABASE_URL` / `ANON_KEY`, the
app silently runs fully local**, backed by `localStorage`, with cross-tab sync
via the browser `storage` event. That is the default dev mode.

---

## 2. Architecture — the three things that matter for AI integration

### 2.1 Event sourcing is the core invariant

From `CLAUDE.md`:

> `StatEvent` (append-only) is the single source of truth. **No statistic,
> score, or standing is ever hand-stored.** Every number shown anywhere — match
> stats, attack/serve/reception percentages, standings, season records — is
> derived from `StatEvent`s (+ set scores).

In Postgres this is `stat_events` plus two derived **views**: `match_statistics`
and `standings`. There are no stored aggregates anywhere.

**Why this matters enormously:** an AI/CV system does not need to integrate
with statistics, analytics, standings, charts, the showcase, or the fan views
*at all*. If it can append correct rows to `stat_events`, **every downstream
number in the product updates itself**. This is the single most favourable
architectural fact in the whole investigation.

### 2.2 The repository boundary is a clean injection seam

`src/lib/repository.ts` defines `DataProvider`. The UI talks *only* to this
interface. Two implementations satisfy it:

- `LocalProvider` (`src/lib/store.tsx`) — localStorage, offline-first
- `SupabaseBackend` (`src/lib/providers/supabase-store.ts`) — Postgres +
  Realtime + RLS

The event-writing method is:

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

**This one function is the entire AI → VolleyVerse contract.** An AI producer
is just a third writer against the same append-only table. Writes are already
optimistic with **client-minted UUIDs** (`crypto.randomUUID()`) and an
idempotent offline queue (`volleyverse:sync-queue:v1`) — which means an
external service can mint ids and replay safely without any new machinery.

### 2.3 The live projection is already a broadcast channel

`match_live_state` is one JSONB row per match holding the rich courtside
`MatchState` (lineups, running score, current rally). The scorer pushes it on
every tap (`pushLiveState`); every viewer subscribes via Supabase Realtime
(`subscribeLiveState`). It is explicitly documented as **a rebuildable
projection, not a source of truth**.

So a live scoreboard fed by AI already has a delivery path. No new realtime
infrastructure is needed.

---

## 3. The data model

### 3.1 Relational schema (`supabase/schema.sql`, mirrored 1:1 by `src/lib/types.ts`)

```
leagues → seasons → divisions
                 → tournaments → tournament_groups
                              → matches
venues → courts
teams → team_honours, staff, team_players → players
                                          → roster_view (flattened, what the UI consumes)
matches → match_officials, match_sets, match_rosters, stat_events, match_live_state
VIEWS: roster_view, match_statistics, standings   (all security_invoker = true)
```

Notable details:

- `team_players` carries `jersey_no` (nullable), `position`
  (`OH|OPP|MB|S|L|DS`, plus `U` = "Universal" in the TS type), `is_captain`,
  `is_reserve`. Unique on `(team_id, season_id, jersey_no)`.
- `matches.total_sets` is 3 or 5; `published` is the visibility gate.
- `match_rosters` marks `is_starter` and `is_libero` per match.
- `REPLICA IDENTITY FULL` on every shared table, all added to the
  `supabase_realtime` publication, so deletes propagate too.

### 3.2 `stat_events` — the event model

```sql
create table stat_events (
  id uuid primary key default gen_random_uuid(),
  match_id uuid not null references matches (id) on delete cascade,
  team_id  uuid not null references teams (id),
  player_id uuid not null references team_players (id),
  set_no integer not null,
  type text not null check (type in ( ...26 types... )),
  vs_player_id uuid references team_players (id),  -- the duel opponent
  ts timestamptz not null default now()
);
```

**26 event types, in 7 groups:**

| Group | Types |
|---|---|
| Attack | `SPIKE_POINT` (kill), `SPIKE_TOOL` (block-out), `SPIKE_IN`, `SPIKE_ERR`, `SPIKE_BLOCKED` |
| Reception | `RECV_PERFECT`, `RECV_GOOD`, `RECV_POOR`, `RECV_ERR` |
| Setting | `SET_ASSIST`, `SET_GOOD`, `SET_ERR` |
| Blocking | `BLOCK_WIN`, `BLOCK_MISS`, `BLOCK_TOOLED` |
| Serving | `SERVE_ACE`, `SERVE_IN`, `SERVE_ERR` |
| Defence | `DIG_SUPER`, `DIG_SAVE`, `DIG_FAIL` |
| Faults | `FAULT_NET`, `FAULT_FOUR_HITS`, `FAULT_DOUBLE`, `FAULT_ROTATION` |

`vs_player_id` names the opponent on the other end of a net duel: the blocker
on `SPIKE_TOOL`/`SPIKE_BLOCKED`, the spiker on `BLOCK_WIN`/`BLOCK_TOOLED`.
Every spiker-vs-blocker matchup in the product derives from this column.

### 3.3 ⚠️ What `stat_events` does NOT have — the gap for AI

This is the crux of STEP 1. The current event row has **no**:

| Missing | Why AI needs it |
|---|---|
| `source` / provenance | Distinguish `HUMAN` / `AI` / `AI_CONFIRMED` / `AI_CORRECTED`. Without it you can never measure model accuracy against human ground truth, or roll back a bad model version. |
| `confidence` | The whole human-in-the-loop design depends on a calibrated score to threshold on. |
| `rally_id` | Events are currently an undifferentiated time-ordered stream per set. Rally grouping is *reconstructed* by walking events (`rallyOutcomes` in `analytics/volleyball.ts`). An AI produces events *per rally* and needs to group them atomically. |
| `video_ts_ms` + `video_asset_id` | No link from an event back to the frame it came from. Needed for review UI, for dispute resolution, and to build a training set. |
| `court_x, court_y, court_z` | No spatial data at all. All the zone/trajectory analytics competitors offer are impossible today. |
| `review_status` | Pending / accepted / rejected, for the verification queue. |
| `model_version` | Reproducibility and regression tracking. |

**All of these are additive.** New nullable columns + a widened `CHECK`
constraint. The project already has a clean precedent for exactly this kind of
migration — see `2026-08-12-attack-detail-and-duels.sql`, which widened the
type `CHECK` and added the nullable `vs_player_id` column, with an explicit
warning that the migration must be run *before* deploying code that writes the
new values (because the optimistic UI shows the tap landing while the database
silently refuses it inside the queue).

---

## 4. The rules engine — VolleyVerse's hidden asset

`src/lib/rally.ts` is a **100% pure FIVB state machine** (no React, no storage,
no DOM, no runtime imports at all). It models:

- **Court**: 6 positions; P1 = back-right = the serving slot; `FRONT_ROW = [4,3,2]`,
  `BACK_ROW = [5,6,1]`. `Lineup = Record<Position, playerId>` is the single
  truth of "who is where".
- **Rotation**: `rotate()` shifts clockwise; `resolvePoint(serving, winner)`
  returns `{ nextServing, rotateWinner }` — the winner rotates **iff** they were
  receiving (a side-out).
- **Toss**: `servingFromToss`, `servingForSet` (alternation convention for
  sets 2..n-1, explicitly flagged in-code as an assumption not an FIVB rule),
  `firstServerForSet` (returns `null` for the deciding set until a fresh toss
  is entered — scoring is blocked, the engine never silently alternates).
- **Phases**: `SERVE → RECEIVE → SET → ATTACK → DEFEND → DIG → OVER`.
- **The trio**: every contact ends one of three ways — `WIN` (✓), `CONT` (O),
  `LOSE` (✗). *What* was won/lost/continued is **inferred from phase + who was
  tapped, never asked.*
- **`inferAction(phase, frontRow)`**: the only genuine ambiguity in the flow is
  the first touch after an opposing attack — **front row ⇒ BLOCK, back row ⇒
  DIG**. Position resolves it.
- **`resolveTrio(action, side, trio)`** is the core table mapping
  action × trio → `{ event, pointTo, nextPhase, nextSide }`.
- **`skipPhase()`**: courtside reality — scorers miss contacts, so the flow can
  advance without logging anything.

`src/lib/substitution.ts` handles court changes on one principle: **a rotation
slot is the identity, not the player in it.** The libero swap is **automatic**,
driven only by who holds serve (serving ⇒ MB on court; receiving ⇒ libero in
the back-row MB's slot, FIVB 19.3.2.1). Ordering contract: on a side-out,
**rotate first, then sync liberos**.

`src/lib/free-rally.ts` is a looser alternative engine that drops the phase
sequence entirely and keeps exactly one inference: *the serving side's P1,
tapped before anything else in the rally, is a serve; everything else is a
spike.* This exists because real rallies don't follow the canonical sequence.

### Why this is the most important finding in STEP 1

A huge fraction of VolleyVerse's data model is **derivable, not detectable**:

| Fact | How it's obtained today | Does CV need to detect it? |
|---|---|---|
| Current rotation of both teams | Rules engine, from lineup + point sequence | **No** |
| Who is serving | P1 of the serving side | **No** |
| Who is front row / back row | Lineup | **No** |
| Block vs dig on first defensive touch | Court row of the player | **No — free** |
| Libero on/off court | Automatic from serve possession | **No** |
| Side-out vs break point | Derived in `rallyOutcomes` | **No** |
| `FAULT_ROTATION` | Pure rules check | **No** |
| Set/match completion, standings | Derived views | **No** |

This cuts the computer-vision problem down substantially, and it works in the
other direction too: **the rules engine is a constraint on what the AI is
allowed to propose.** A serve can only come from the serving side's P1. A
"block" can only come from a front-row player. The third contact is a set from
the team that just received. Constrained decoding against a legal-move
generator is how you turn a mediocre classifier into a usable system — and
nobody else in this market has a production FIVB state machine already wired
to their event model.

---

## 5. How a live match works today (the manual workflow to be replaced)

Route: `/console/matches/[id]/rally` (`page.tsx`, ~1,175 lines) →
`SetupWizard` → `SetRotationGate` → `LiveScreen` → `CourtBoard`.

1. **Toss** — winner + serve/receive choice.
2. **Line-ups** — tap-to-place the starting six for both teams, plus liberos.
   `lineupComplete()` requires six slots and six *different* people.
3. **Live**: the screen shows both courts. The operator:
   - taps a player, then taps ✓ / O / ✗; **or**
   - **holds the player and flicks** — `src/lib/gesture.ts` resolves a pointer
     delta into `LEFT | RIGHT | UP | NONE` with a 24px threshold and very wide
     sectors, explicitly because "a scorer watching the court is not looking at
     the screen at all". Down = cancel (narrow sector, deliberately).
   - ✓/✗ can be refined into `KILL | TOOL | BLOCKED | ERROR`, and the two duel
     kinds prompt for the opposing player (→ `vs_player_id`).
   - faults are tapped separately (`NET | FOUR_HITS | DOUBLE | ROTATION`).
4. Undo exists at two levels — **action-undo** (within the current rally, via
   `RallyState.current`) and **rally-undo** (via `RallySnapshot` history, which
   restores score, lineups, libero state and sub counters, and deletes the
   emitted event ids).
5. Every tap calls `persist` → local write + `pushLiveState` → Supabase.
6. Post-match corrections at `/console/matches/[id]/review`.

**Order-of-magnitude workload:** a 25-point set is roughly 45–55 rallies; a
tracked rally is 4–8 contacts. That is **~200–400 deliberate inputs per set**,
**~1,000–2,000 per five-set match**, under time pressure, from one person who
must also watch the court. That is the problem worth automating — and the
existence of `skipPhase()`, `free-rally.ts` and the flick gesture is direct
evidence the team already knows the operator cannot keep up with full fidelity.

---

## 6. Statistics & analytics

- `src/lib/metrics.ts` + `src/lib/analytics/` derive everything.
- `analytics/framework.ts` is **sport-agnostic**: `SportModule` →
  `MatchAnalytics` (KPIs, progression, timeline, comparison metrics,
  performers, stat tables, team stat lines, distributions). New sports plug in
  via `registerSport()`.
- `analytics/volleyball.ts` (~730 lines) is the first concrete module. It
  reconstructs the point sequence from events:
  - `WIN_EVENTS = {SPIKE_POINT, SPIKE_TOOL, SERVE_ACE, BLOCK_WIN}`
  - `ERR_EVENTS = {SPIKE_ERR, SERVE_ERR, RECV_ERR, SET_ERR, BLOCK_MISS, DIG_FAIL}`
  - `SPIKE_BLOCKED` and `BLOCK_TOOLED` are deliberately **excluded** — each of
    those rallies logs two events (one per side) and counting both would
    double-score the rally.
  - `rallyOutcomes()` classifies each rally as break point / side-out /
    first-ball side-out. **It assumes the first server of a set alternates
    (odd sets → home)** because events alone don't record the toss.
- `src/lib/spikes.ts` and `src/lib/blocks.ts` are separate, deliberately
  narrower tally modules (documented reason: `PlayerLine.spikeSuccesses` counts
  `SPIKE_IN` as a success, which the spiker screen must not do).
  `blocks.ts` carries an unusually honest comment about what a block success
  rate *can* mean given the app never sees a block that was jumped too late.

---

## 7. Security / access control (relevant to letting a machine write)

The history here matters:

1. `schema.sql` originally required `auth.role() = 'authenticated'` for writes.
2. The console sign-in was removed → every write was refused → 
   `2026-08-11-open-console-writes.sql` opened **everything to `anon`**, with a
   frank in-file warning that the anon key ships in the public JS bundle and
   `published` became a display filter rather than a boundary.
3. `2026-08-13-console-admins-and-rls.sql` introduced a `console_admins`
   allowlist + `is_console_admin()`.
4. `2026-08-31-user-roles-and-admin-panel.sql` replaced it with `user_roles`
   (`super_admin | admin | simple_user`), `is_super_admin()`, and re-pointed
   every write policy at `is_console_admin()`.

⚠️ **Note for the AI work:** `user_roles` is seeded with a hard-coded personal
Gmail address as `super_admin`. An AI event producer will need its own
service identity — either a Supabase service-role key held server-side (never
in the edge box), or a dedicated `ai_producer` role added to `user_roles` with
a scoped write policy limited to `stat_events` for matches it is assigned to.
Do **not** ship a service-role key to a gym-side device.

---

## 8. Integration verdict

> **Can an AI/CV system feed events into the existing system without
> redesigning VolleyVerse?**
>
> **Yes.** The event-sourced model, the `DataProvider` boundary, the
> client-minted UUIDs, the idempotent offline queue and the realtime live-state
> projection are collectively close to an ideal ingestion surface. The required
> changes are additive:

**Minimum viable integration (no breaking changes):**

1. Migration: add `source`, `confidence`, `rally_id`, `video_ts_ms`,
   `review_status`, `model_version` as **nullable** columns on `stat_events`
   (+ an index on `(match_id, review_status)`). Existing rows and existing code
   are unaffected — nullable columns change no existing row, exactly as the
   `vs_player_id` migration did.
2. An `ai_producer` role in `user_roles` + a write policy scoped to
   `stat_events`.
3. A thin ingestion endpoint (Supabase Edge Function or a Next.js route
   handler) that accepts a *proposed rally* — an ordered list of candidate
   events with confidences — validates it against `rally.ts` (the rules engine
   runs server-side unchanged, it's pure), and writes the legal ones.
4. A verification queue screen: everything below the confidence threshold
   lands as `review_status = 'pending'` and is confirmed with the same
   hold-and-flick gesture already built for `CourtBoard`.
5. `pushLiveState` continues to own the scoreboard — unchanged.

**What would need real work regardless:** spatial data (`court_x/y/z`) has no
home in the current model and no consumer in the current analytics; adding it
is a genuine feature, not an integration detail.

---

## 9. Files to re-read if the repo has moved on

| File | Why |
|---|---|
| `supabase/schema.sql` | The whole relational model + RLS |
| `src/lib/types.ts` | `EventType`, `StatEvent`, `Match`, `Player` |
| `src/lib/rally.ts` | The FIVB state machine — the constraint engine |
| `src/lib/repository.ts` | `DataProvider` — the injection seam |
| `src/lib/substitution.ts` | Libero/rotation rules |
| `src/lib/providers/live-state.ts` | The live broadcast channel |
| `src/lib/analytics/volleyball.ts` | How stats are derived from events |
| `src/app/console/matches/[id]/rally/page.tsx` | The manual workflow being replaced |
| `CLAUDE.md`, `REALTIME_SYNC.md` | The team's own architecture docs |
