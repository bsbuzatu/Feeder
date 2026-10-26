# Feeder — Training Log

Single-file offline PWA for strength training and Zone 2 cardio. No build step, no dependencies, no backend. All data lives in the browser (IndexedDB) and never leaves the device.

**Live:** https://bsbuzatu.github.io/Feeder/
**Build:** `2026-10-26-v11`

---

## What it does

| Tab | Purpose |
|---|---|
| **Gym** | Week plan, next session (A→B→C rotation), run a Full-Body session, Zone 2 timer or manual walk entry |
| **Fast** | Start / end a fast (16 h target), edit start and end afterwards, 7-day average, history |
| **Food** | **Log** meals by preset code + comment · **Cook** batch recipes for 2 portions · **Shop** weekly list from the planned batches · **List** approved foods |
| **Stats** | Body composition, waist, Health card (Fibroscan / CAP / labs vs targets), fasting and intake averages, volume |
| **Settings** | Targets (kcal, protein, fast hours, goal weight), export / import JSON v4, deload, Zone 2 calibration, wipe |

## v11 (26 Oct 2026) — three goals in one app

Goals: lean muscle, lose visceral fat, clear the fatty liver (CAP 371 dB/m, 4.4 kPa, 19.10.2026).

- **Rotation, not weekdays.** The next session is the letter after the last one lifted. A missed day shifts the rotation instead of skipping a session.
- **Zone 2 only on non-lifting days** (Tue / Thu / Sat / Sun), 150 min/week target. Walks without the timer are logged manually.
- **Fasting store** (`fasts`): `{start, end, target}`. Logging a meal for today ends a running fast.
- **Meal presets** with codes: `SHK` / `SHKN` morning shake, `OML` weekend omelette, `M1`–`M6` batch meals (one cook = lunch + early dinner), `OTH` free entry with kcal / protein estimate. Macros come from the recipe; the comment field records deviations.
- **Targets** default to 2,150 kcal, 185 g protein, 16 h fast, goal weight 86 kg (−8 %). Editable in Settings.
- **Labs store** (`labs`): one record per date; the Health card shows the latest value per marker against targets.
- Export is JSON **v4** (adds `fasts`, `meals`, `labs`, `mealPresets`); import accepts v2–v4 with de-duplication.

---

## Training program

Full-Body ×3 per week (Mon / Wed / Fri). Replaced a 4-day Upper/Lower split because actual attendance was 2.6–2.9 sessions per week, which meant lower-body days were skipped disproportionately — quads were getting 7 sets/week and hamstrings 5, both below the hypertrophy threshold.

| Session | Anchor lifts |
|---|---|
| **Full-Body A** | Back Squat · Bench Press · One-Arm Machine Row · Lying Leg Curl · DB Lateral Raise · Ab Crunch |
| **Full-Body B** | Leg Press · RDL · Seated DB OHP · Close-Grip Pulldown · Chest Fly · Hammer Curl + Overhead Triceps (superset) · Seated Calf |
| **Full-Body C** | Hip Thrust · Incline DB Press · Pull-Up · Leg Extension · Seated Leg Curl · Face Pull · One-Leg Calf · Incline DB Curl |

Weekly volume at 3 sessions: quads 8 · hamstrings 8 · chest 8 · back 9 · delts 9 · calves 4 · arms 4–6. At 2 sessions every muscle still gets 2 stimuli — the old split gave it 1.

Targets are tracked live in **Stats → Sets per muscle group (7 days)**, colour-coded against retention minimums.

---

## Progression engine

Boxes are pre-filled with **today's prescription**, computed from the last session. Ramp sets repeat; the decision is made on the **top set**:

| Last top set | Today |
|---|---|
| reps ≥ top of range | weight **+ increment**, reps back to bottom of range |
| reps within range | same weight, **+1 rep** |
| reps < bottom of range | same weight, hold and aim for bottom of range |

Increments are per-exercise (`inc` in the `SESSIONS` array): 5 kg on squat/leg press/hip thrust/RDL, 2.5 kg on presses and rows, 1 kg on lateral raises and curls.

Confirming a set saves exactly what is in the boxes. **Edit first if you did something different** — the checkmark is a commitment, not an autofill.

Bodyweight work (pull-ups) is stored as 0 kg and progressed on reps.

### Why boxes are not just "last session's numbers"

The v3 export revealed identical set-for-set entries repeated across four separate sessions (Lying Leg Curl `35×12×3 = 1129 kg` on 10.07, 21.07, 03.08, 10.08). The previous build pre-filled the last session's values and the checkmark saved them unedited, so the tonnage series was partly fictional and progression could not be assessed. Prescribing forward instead of backward removes the failure mode: the reflexive tap now moves the load up.

Empty boxes are rejected — a set cannot be confirmed without both kg and reps.

---

## Deload mode

Settings → toggle. Runs sessions at −40% sets, same weights, RIR 3–4, progression paused. Triggers documented in-app:

- bench or squat regression two sessions in a row
- resting HR more than 5 bpm above baseline
- sleep under 6 h for 3 nights

---

## Zone 2

Calibrated 22 Aug 2026 for a 44-year-old, RHR ≈ 67, HRmax estimated at 177 (Tanaka; not directly measured).

| Method | Range |
|---|---|
| 60–70% HRmax | 106–124 |
| Karvonen 50–60% HRR | 122–133 |
| MAF (180 − age) | cap 136 |

**Working target: 118–133 bpm, hard cap 135.**

Treadmill at 6–6.5 km/h, 6–10% incline, adjusting incline to hold 125–132. Talk test: full sentences. 3 × 40 min on non-lifting days, progressing toward 150 min/week. Never before a lifting session.

Logged as `type: 'cardio'` workouts with duration and avg/max HR entered manually. The weekly total shows on the Gym tab and as a Stats KPI.

> Recalculate these numbers if resting HR shifts materially or HRmax is ever measured directly. A previous build shipped 130–142, derived from RHR 60 and percentages set too high — that band is Zone 3 for this profile and compromises recovery between lifting days.

---

## Data model

IndexedDB `bogdan_nutrition_tracker`, `DB_VERSION = 3`.

| Store | Key | Contents |
|---|---|---|
| `workouts` | `id` (auto) | strength sessions and cardio blocks |
| `metrics` | `date` | weight, muscle, fat, waist, sleep, steps, flags |
| `settings` | `key` | deload flag |
| `entries` | `id` (auto) | legacy food entries, preserved but not displayed |

```js
// strength
{ date, type:'strength', sessionId, sessionName, startTs, endTs, deload?,
  exercises:[{ exId, name, targetSets, targetReps, rir, rest, unit, inc, ss,
               sets:[{ weight, reps, done }] }] }

// cardio
{ date, type:'cardio', sessionId:'zone2', startTs, endTs, z2min, avgHR, maxHR,
  targetLo, targetHi }
```

Only confirmed sets are persisted — unchecked sets are stripped on finish.

### Exercise identity

Every exercise has a stable `exId` plus an alias list in the `EX` map. Renaming the display name does not orphan history: `Chest-Supported DB Row` → `One-Arm Machine Row` still resolves to `csRow`, so progression continues across the rename.

When an exercise moves to different equipment (cable → machine, barbell → dumbbell), **the first session's prescription will be on the old scale.** Edit the weights once; from the second session the engine is calibrated again.

---

## Export / import

**Settings → Export JSON** produces `training-log-YYYY-MM-DD.json` containing workouts, metrics, settings, the exercise catalogue and the current program.

Import merges rather than overwrites: workouts are de-duplicated on `date|sessionId` (plus `startTs` for cardio, since multiple blocks per day are valid), and existing metric fields win over imported ones.

**Export before every deploy.** Data survives a file replacement because IndexedDB is keyed to the origin, not the file — but verify that, don't assume it.

---

## Deploy

Replace `index.html` on `main`. GitHub Pages publishes in 1–2 minutes.

The page sends `no-cache` headers, but iOS sometimes holds the old version in memory: close the PWA from the app switcher and reopen. Confirm the build string in **Settings → Database contents**.

Access code: `bbb` — a friction gate, not security. Anything sensitive should not be in here.

---

## Constraints and conventions

- Single file. No bundler, no npm, no external requests.
- Vanilla JS in strict mode. No framework.
- Dark theme, neon green (`#39FF14`), blue (`#4aa3ff`) reserved for cardio.
- UI text is English throughout, sized for reading mid-set without glasses.
- Nutrition data reflects an atorvastatin + ezetimibe protocol: grapefruit and pomelo are listed as contraindicated (CYP3A4).

---

## License

None. Personal project.
