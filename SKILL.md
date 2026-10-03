---
name: swim-workout
description: >
  Use this skill whenever building, drafting, formatting, or editing a competitive swim
  practice or workout for SetForge. Triggers include any request to generate a practice,
  build a set, format a workout for Commit Swimming, or produce session output from
  SetForge app parameters. Always read this SKILL.md fully before generating any output —
  this skill governs the output contract, Commit formatting, set design, drill selection,
  and energy zone logic. Do not generate any workout content without consulting this skill.
---

# SetForge Swim Workout Skill

## Overview

This skill governs all workout generation for SetForge — a native iPhone app that sends
a fully formed prompt to Claude. Most requests are one-shot. The coach may follow up in the
same conversation with an adjustment request (see 3.8). The output goes directly to a display
view and then to Commit Swimming. Every response must be correct on its own — never
rely on a follow-up to fix it.

Read this file fully. Then read the reference files as directed below.

**Always read before generating:**
- `references/drills.md` — approved drill library (S/A/B/C tiers); no other drills permitted
- `references/set-templates.md` — zone-organized set patterns; draw from these when designing sets
- `references/energy-zones.md` — full zone design guidelines; read when zone guidance is needed

---

## 1. Context

**Group**: Velocity (competitive youth, Swimming Canada T2T framework)
**Pool**: 25-metre
**Session length**: defined by the app prompt (`Total time` field)
**Schedule**: Monday, Tuesday, Wednesday, Friday, Saturday
**Platform**: Commit Swimming — all output must parse correctly (see Section 5)
**Equipment**: fins, snorkel, paddles, kickboard, pull buoy

---

## 2. Input Contract

SetForge always sends a complete, structured prompt. Never ask for clarification.
Never request missing fields. Never assume or default volume.
The prompt is the complete input — treat it as authoritative.

The prompt opens with `Generate a swim practice with the following parameters:`, then
the blocks below, always in this fixed order, separated by blank lines:

1. `SESSION PARAMETERS` — always present
2. `STROKE & INTENSITY` — always present
3. `SECTION STRUCTURE` — always present
4. `DESIGN FLAGS` — optional
5. `DRILL NOTES` — optional
6. `RECENT SESSIONS` — optional; always last, because it is context, not instruction

Fields and blocks marked optional may be absent. When one is absent, skip every rule
that depends on it — never invent a value for it.

### 2.1 Prompt fields

**SESSION PARAMETERS**
- `Total volume` — metres for the generated sections only; distribute per Section 4.
  It may already be reduced by the app to fit around time-block sections — use it as given,
  never scale it up or down yourself. Time-block sections contribute 0m.
  Hit it within ±100m in the final workout.
  **Exception — `Salo-inspired` active:** volume is a ceiling, not a target. Coming in under is
  fine (see `references/set-templates.md`, Sprint Salo Patterns); state the shortfall in
  Coaching flags (§3.3), e.g. `Salo session: 2,900m of 3,500m — quality over volume`.
  If the volume cannot fit inside `Total time`, the time cap wins (§3.6); state the shortfall
  in Coaching flags.
- `Total time` — minutes; hard limit (already adjusted for time-block sections)
- `Training phase` — one of: `GPP` · `SPP` · `Competition` · `Taper`
- `Phase week` — optional; a whole number (e.g. `- Phase week: 3`), the week within the
  current phase. Used only in the workout header (5.6). If absent, the header has no week —
  do not work one out from RECENT SESSIONS or anything else.
- `Day` — one of: `Monday` · `Tuesday` · `Wednesday` · `Friday` · `Saturday`

**STROKE & INTENSITY**
- `Primary stroke focus` — one of: `FR` · `BK` · `BR` · `FLY` · `IM`
  - Technical focus options (optional) follow the stroke after an em dash, comma-separated:
    `- Primary stroke focus: FR — EVF / catch, Body rotation`. With none selected the line
    is just `- Primary stroke focus: FR`.
  - Focus note (optional) — free-text coaching note on its own indented line directly below
    the stroke line, with no label. Include it as a coaching note in the output if present.
- `Intensity range` — start and end, e.g. `easy to strong`

Example:

    STROKE & INTENSITY
    - Primary stroke focus: FR — EVF / catch, Body rotation
      long and strong off every wall
    - Intensity range: easy to strong

**SECTION STRUCTURE**
- Lists active sections in order; Warm Up always first, Cool Down always last
- Each section is either:
  - `active, generate content` — Claude generates this section fully
  - `active, time block only — N min` — coach-managed; include a placeholder line only (see 2.2)
- Inactive sections are omitted entirely from the prompt
- A section line may be followed by an optional indented `[note: …]` line. This is the coach's
  hint for that section only — follow it when designing that section (e.g. `[note: broken 200s]`
  under Main Set). Section hints never override volume, time or intensity range.

Example:

    SECTION STRUCTURE
    - Warm Up: active, generate content
    - Pre-Set: active, generate content
    - Starts & Dives: active, time block only — 12 min
    - Main Set: active, generate content
      [note: broken 200s]
    - Cool Down: active, generate content

**DESIGN FLAGS** (block omitted entirely if none active)
- `Salo-inspired` — use short-burst, generous-rest structures in the main set
- `Descending intensity` — build from lower to higher intensity across the main set
- `Building intensity` — structure each repeat or sub-set as a build
- `Drill-heavy` — increase drill proportion in warm-up and pre-set; S and A tiers only

**DRILL NOTES** (block omitted entirely if the coach picked no drills; either line may be omitted)
- `Include` — comma-separated drill names from `references/drills.md`. Use each listed drill
  at least once, in a section where it fits (Warm Up, Pre-Set, Drill Work). Coach inclusion
  overrides tier rules: a listed C-tier drill is allowed, and `Drill-heavy`'s S/A restriction
  does not apply to listed drills.
- `Exclude` — comma-separated drill names. Do not use these drills anywhere in the session.
- If a name does not match a drill in `references/drills.md`, ignore it and mention it under
  Coaching flags.
- When the block is absent, select drills normally per 6.1.

Example:

    DRILL NOTES
    - Include: Fist Drill, Heel Tag
    - Exclude: Zipper Drill

**RECENT SESSIONS** (block omitted entirely if there is no recent history)
- Practices swum in the past 7 days, most recent first — context only; see 2.4
- When the block is absent, design the session from today's parameters alone.

### 2.2 Time-block sections

When a section is marked `active, time block only — N min`, output this placeholder:

[Section Name]
      [Coach-managed block — N min]

Do not generate set content for time-block sections.
Time-block sections contribute no metres to the volume audit.

### 2.3 Intensity mapping

Use these words only in output — never use zone codes (A-I, R-III, etc.):

| Descriptor | Zone           | Commit scale |
|------------|----------------|--------------|
| easy       | A-I            | 3            |
| moderate   | A-II           | 5            |
| strong     | A-II upper / R-III | 7        |
| fast       | R-III / S-IV   | 9            |
| sprint     | S-IV           | 11           |
| all out    | S-V            | 12           |

The `Intensity range` field defines the session floor and ceiling. Arc across that range —
do not sit at the ceiling throughout.

### 2.4 Recent sessions

Applies only when the RECENT SESSIONS block is present. Example:

    RECENT SESSIONS
    - Friday 26 Sep (3 days ago) — SPP, 3800m, FR — EVF / catch, easy to fast
      Theme: SPP Week 3 — Freestyle catch mechanics / aerobic development
      Main set: 3x (4 x 100 @ 1:25 strong, 200 pull @ 3:00 moderate); 8 x 50 @ 1:00 odds fast evens easy
      Feedback: too hard — "half the group missed the 1:25s"
    - Wednesday 24 Sep (5 days ago) — SPP, 3500m, BK, easy to strong
      Main set: 6 x 200 @ 3:10 as 50 drill [Single Arm] / 150 swim moderate
      Feedback: none

Each entry gives the day and date, how many days before today's session it was, phase,
volume, stroke focus (with technical focus after an em dash, if any) and intensity range,
then optional `Theme:` and `Main set:` lines, then `Feedback:` from the coach after the
practice — a label, optionally followed by ` — "coach's note"`.

How to use it — a light touch:
- **Today's parameters always win.** SESSION PARAMETERS, STROKE & INTENSITY, SECTION STRUCTURE,
  DESIGN FLAGS and DRILL NOTES are never changed by recent sessions: not volume, time, phase,
  stroke focus, intensity range, sections, flags or drill picks.
- **Don't repeat recent main sets.** Do not repeat a main set listed in RECENT SESSIONS: change
  at least one of repeat distance, set format (straight / broken / circuit / descending /
  ladder) or equipment. If today's stroke focus matches a recent session, also use different
  drills from that session where DRILL NOTES allow.
- **Day after a hard session.** If the most recent session was 1 day ago and its intensity range
  ended at `fast`, `sprint` or `all out`, keep today's highest-intensity work short and late in
  the main set (still within today's intensity range).
- **Feedback labels:**
  - `worked` — this kind of set suited the group; similar structures are fine on a different
    day with a different stroke or distance.
  - `too hard` — for similar sets today, use more generous sendoffs (about +5 sec per 100)
    or fewer reps.
  - `too easy` — for similar sets today, tighten sendoffs (about −5 sec per 100) or add reps.
  - `changed` — the coach edited that workout before swimming it; avoid that main-set format today.
  - `didn't work` (older entries) — avoid that main-set format today.
  - `none` — no signal; use only for variety.
  - A quoted note is the coach's own words. Treat it as a preference for today's design
    where relevant.
- Never mention recent sessions in Part 1 or Part 3. Part 2 may refer to them only in
  **Week context** (3.3).

---

## 3. Output Contract — CRITICAL

Output has three parts, always in this order. Do not deviate from this structure.

### 3.1 Generation sequence — follow exactly

1. Calculate the section budget silently — see Section 3.4
2. Write Part 1 — the first draft workout
3. Write Part 2 — the session summary, including full volume audit
4. If the audit finds any section over budget, apply corrections and rewrite
   those sections explicitly in the summary
5. Write Part 3 — the final corrected workout, reflecting all corrections from step 4

Part 3 is what the app displays and the coach copies. It must match the
final audited figures in Part 2 exactly.

### 3.2 Part 1 — Draft Workout

- Starts with the bracket header line (see 5.6)
- Ends with the last line of the Cool Down section
- Contains only Commit-ready text — no markdown, no code fences, no preamble
- This is a working draft — the app discards it and displays Part 3 instead

### 3.3 Part 2 — Session Summary

Separated from Part 1 by this exact line:

---SESSION-SUMMARY---

Contains in this order:

**Volume audit** — for each generated section, list every set with its metre
value, the section total, and whether it matches the budget:

    Warm Up
      400 straight swim = 400m
      6 x 75 = 450m
      Section total: 850m — budget: 700m — OVER by 150m
      CORRECTION: reduce to 4 x 75 = 300m → section total 700m — MATCH

If any section is over budget, document the correction clearly.
The corrected figures must be carried into Part 3.

**Intensity distribution** — one or two sentences, plain prose

**Technical emphasis** — one sentence

**Week context** — one sentence on how today differs from the recent sessions; only when the
RECENT SESSIONS block is present — omit this line entirely otherwise

**Active design flag notes** — one line per active flag; omit if none

**Coaching flags** — brief, if warranted

### 3.4 Pre-generation budget allocation

Before writing Part 1, calculate the metre budget for each section silently.
Use the proportions from Section 4 adjusted for the sections present in the prompt.

This calculation is silent — do not write it, do not label it, do not output
it in any form. It must not appear anywhere in the response.

The format to use internally:

  Warm Up [X]m / [Section] [X]m / Main Set [X]m / Cool Down [X]m / Total [X]m

The total must equal the `Total volume` specified in the prompt (with `Salo-inspired`,
it may be lower — see 2.1).

### 3.5 Part 3 — Final Corrected Workout

Separated from Part 2 by this exact line:

---FINAL-WORKOUT---

- Identical in format to Part 1 — pure Commit-ready text
- Reflects all corrections documented in the Part 2 audit
- This is the authoritative output — it must match the Part 2 final figures exactly
- No code fences, no markdown, no preamble
- Starts with the bracket header line
- Ends with the last line of the Cool Down section

### 3.6 Time estimation

The `Total time` field is a hard limit. When building sets, estimate session
duration continuously and ensure the workout fits within the available time.

Use these rules to calculate time for every set:

**Sets with a sendoff:**
  time = sendoff × rep count
  Example: 8 x 50 @ 1:10 = 8 × 1:10 = 9:20

**Sets without a sendoff:**
  Use 30 seconds per 25m as the default pace estimate.
  Example: 400 straight swim = 16 × 0:30 = 8:00
  Example: 6 x 75 = 6 × (3 × 0:30) = 6 × 1:30 = 9:00

**Rest lines:**
  Use face value.
  Example: 2:00 rest = 2:00

**Time-block sections:**
  Use the coach-specified minutes directly — do not generate sets for these.

**Circuits:**
  Sum the time of all lines inside the circuit, then multiply by the round count.
  Example: 3x (4 x 100 @ 1:25 + 200 @ 3:00 + 1:00 rest)
           = 3 × (4 × 1:25 + 3:00 + 1:00)
           = 3 × (5:40 + 3:00 + 1:00)
           = 3 × 9:40 = 29:00

Sum estimated time across all sections. If the total exceeds `Total time`,
reduce volume or remove sets before finalising the workout. The time hard
cap is non-negotiable — do not output a workout that exceeds it.

### 3.7 Output rules

- No preamble before Part 1 — first character of the entire response is `[`
- No horizontal rules before the bracket header
- No budget calculation visible anywhere in the response
- No zone codes anywhere in any part
- No code fences in Part 1 or Part 3

### 3.8 Adjustment turns

After a workout has been generated, the coach may send a follow-up message in the same
conversation asking for a change (e.g. "main set's too long, cut 300").

- Apply only the requested change. Keep everything else from the previous final workout
  (Part 3) unless it must change to stay valid (Commit formatting, intensity range, drill
  rules, time cap).
- The coach's request wins over the original `Total volume` and `Total time` when they
  conflict (e.g. "add 400" even if it runs past the time). Otherwise both still apply.
  The Part 2 volume audit and summary report the new total volume and estimated time.
- Always return the full three-part output again, exactly per Section 3: Part 1,
  `---SESSION-SUMMARY---`, Part 2, `---FINAL-WORKOUT---`, Part 3. No preamble,
  no reply to the coach outside the three parts. Note the change made under Coaching flags.

---

## 4. Practice Structure

| Section          | Proportion | Notes                          |
|------------------|------------|--------------------------------|
| Warm Up          | ~20%       | Drill-forward, easy–moderate   |
| Pre-Main / Build | ~15%       | Only if present in prompt      |
| Main Set         | ~55%       | Primary training stimulus      |
| Cool Down        | ~10%       | Easy, active recovery          |

- Adjust proportions to fit the sections actually present in the prompt
- If a section is time-block-only, redistribute its proportion across generated sections
- **Main sets are never single-stroke** — always mix primary stroke with freestyle and/or choice

---

## 5. Commit Formatting Rules

Non-negotiable. Applies to Part 1 and Part 3.

### 5.1 Spacing and syntax
- Spaces around `x`: `4 x 100` — never `4x100`
- Sendoffs: `@ 1:25`, `@ 0:50` — leading zero always for sub-minute times
- `@` strictly for sendoffs only — never for rest
- Rest written inline or on its own line: `30 sec rest` or `2:00 rest`
- Intensity words only: `easy` `moderate` `strong` `fast` `sprint` `all out`

### 5.2 Set formatting
- Numbered groupings: `12 x 50 1-4 kick @ 1:00, 5-8 drill [Fist Drill] @ 1:00, 9-12 free @ 0:50`
- Odds/evens: `6 x 100 @ 1:25 odds sprint evens easy`
- Distance breakdowns: `300 choice (50 drill 25 swim)`
- Timed sets: `6:00 easy swimming, focus on technique`
- Rest lines: `2:00 rest` on its own line

### 5.3 Bracket rules

**Coaching notes** — own line, 6-space indent:

      4 x 50 @ 0:50 fly
      [strong underwater kick off the wall]

**Parser protection** — inline when `kick`, `pull`, `drill`, or zone names appear
in a description but must not change set classification:

      4 x 50 @ 0:50 fly — [focus on underwater kick]

**Drill naming** — `drill` outside brackets, name inside:

      4 x 200 @ 3:30 as 50 drill [I-Y-Scoop] / 150 free

### 5.4 Circuits

      3x
            100 kick
            200 pull

**Nested circuits do not work** — write as explicit separate sets instead.
Sendoffs go on individual lines inside the circuit, not on the header line.

### 5.5 Section headings and indentation
- Section headings flush left — no markdown, no indentation
- Sets indented exactly 6 spaces
- Coaching notes indented 6 spaces, on their own line
- No extra blank lines between sets within a section
- One blank line between sections
- Headings must not contain `kick`, `pull`, or energy system names unbracketed

### 5.6 Workout header

First line of Part 1 and Part 3:

      [Phase Week N — Primary technical focus / zone name]

- `Phase` is the `Training phase` from the prompt.
- `Week N` uses the `Phase week` value from the prompt. If `Phase week` is absent, omit
  `Week N` entirely: `[Phase — Primary technical focus / zone name]`. Never invent a week number.

Example with `Phase week: 3`: `[SPP Week 3 — Freestyle catch mechanics / aerobic development]`
Example without `Phase week`: `[SPP — Freestyle catch mechanics / aerobic development]`
Use zone names (`aerobic base`, `aerobic development`, `race pace`, `speed endurance`),
not zone codes.

---

## 6. Design Principles

### 6.1 Drill selection
- Only approved drills from `references/drills.md`
- Priority: S-tier → A-tier → B-tier situationally; C-tier excluded
- DRILL NOTES (2.1), when present, override the tier priority above: included drills are
  always used, excluded drills never are
- Conservative sendoffs on drill sets (add 15–20 sec beyond expected swim time)
- `Drill-heavy` flag: increase drill proportion; S/A tiers only

### 6.2 Sprint Salo
Active when `Salo-inspired` flag is present:
- Short repetitions (15–50m), generous rest (1:2 minimum, typically 1:3+)
- Quality over volume — not a default
- See `references/set-templates.md` for Sprint Salo patterns

### 6.3 Design flags

| Flag                  | Effect                                              |
|-----------------------|-----------------------------------------------------|
| `Salo-inspired`       | Main set: short-burst / generous-rest structures    |
| `Descending intensity`| Main set arcs from lower to higher intensity        |
| `Building intensity`  | Each repeat / sub-set is a build                    |
| `Drill-heavy`         | Higher drill proportion; S/A tier only              |

Multiple flags apply simultaneously.

### 6.4 Volume and work:rest
- Volume from the app only — never assumed
- Sprint sets: minimum 1:2 work:rest, typically 1:3+
- S-IV + S-V combined should not exceed ~6% of session volume
- S-V deprioritized unless explicitly requested
- For full zone design guidance: read `references/energy-zones.md`

### 6.5 Phase awareness

| Phase       | Emphasis                                               |
|-------------|--------------------------------------------------------|
| GPP         | Aerobic base, stroke volume, drill introduction        |
| SPP         | Race-pace exposure, stroke efficiency under load       |
| Competition | Race specificity, sharpening, low volume / high intensity |
| Taper       | Volume reduction, speed maintenance, feel for water    |

---

## 7. Output Checklist

- [ ] First character of response is `[` — no preamble of any kind
- [ ] Part 1 present — Commit-ready draft, bracket header to Cool Down
- [ ] `---SESSION-SUMMARY---` separator on its own line after Part 1
- [ ] Part 2 present — volume audit, corrections documented, intensity and technical summary
- [ ] `---FINAL-WORKOUT---` separator on its own line after Part 2
- [ ] Part 3 present — final corrected workout, bracket header to Cool Down
- [ ] Part 3 figures match Part 2 final audited totals exactly
- [ ] Final volume within ±100m of `Total volume` (with `Salo-inspired`: at or under, shortfall stated; after an adjustment, per 3.8)
- [ ] Estimated session time does not exceed the `Total time` hard cap
- [ ] Main set mixes primary stroke with freestyle and/or choice
- [ ] Intensity stays within specified range
- [ ] All active design flags applied
- [ ] Time-block sections rendered as placeholders only
- [ ] If RECENT SESSIONS is present: main set does not repeat a recent main set; feedback applied
- [ ] All drills from `references/drills.md`
- [ ] If DRILL NOTES is present: included drills present; excluded drills absent
- [ ] All formatting matches Section 5 in both Part 1 and Part 3
- [ ] No zone codes anywhere in the response
- [ ] No code fences in Part 1 or Part 3
- [ ] No budget calculation visible anywhere in the response

---

## Reference Files

| File                          | When to read                                      |
|-------------------------------|---------------------------------------------------|
| `references/drills.md`        | Always — before selecting any drill               |
| `references/set-templates.md` | Always — before designing any set                 |
| `references/energy-zones.md`  | When zone-specific set design guidance is needed  |
