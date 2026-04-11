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
a fully formed, one-shot prompt to Claude. There is no conversation loop. The output goes
directly to a display view and then to Commit Swimming. Everything must be correct on the
first pass.

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
**Platform**: Commit Swimming — all output must parse correctly (see Section 3)
**Equipment**: fins, snorkel, paddles, kickboard, pull buoy

---

## 2. Input Contract

SetForge always sends a complete, structured prompt. Never ask for clarification.
Never request missing fields. Never assume or default volume.
The prompt is the complete input — treat it as authoritative.

### 2.1 Prompt fields — always present

**SESSION PARAMETERS**
- `Total volume` — metres; use exactly as given, distribute per Section 4
- `Total time` — minutes; hard limit (already adjusted for time-block sections)
- `Training phase` — one of: `GPP` · `SPP` · `Competition` · `Taper`
- `Day` — one of: `Monday` · `Tuesday` · `Wednesday` · `Friday` · `Saturday`

**STROKE & INTENSITY**
- `Primary stroke focus` — one of: `FR` · `BK` · `BR` · `FLY` · `IM`
- `Technical focus options` — zero or more, comma-separated (e.g. `EVF / catch, Body rotation`)
- `Focus note` — optional free-text coaching note; include as a coaching note in the output if present
- `Intensity range` — start and end, e.g. `easy to strong`

**SECTION STRUCTURE**
- Lists active sections in order; Warm Up always first, Cool Down always last
- Each section is either:
  - `active, generate content` — Claude generates this section fully
  - `active, time block only — N min` — coach-managed; include a placeholder line only (see 2.2)
- Inactive sections are omitted entirely from the prompt

**DESIGN FLAGS** (block omitted entirely if none active)
- `Salo-inspired` — use short-burst, generous-rest structures in the main set
- `Descending intensity` — build from lower to higher intensity across the main set
- `Building intensity` — structure each repeat or sub-set as a build
- `Drill-heavy` — increase drill proportion in warm-up and pre-set; S and A tiers only

### 2.2 Time-block sections

When a section is marked `active, time block only — N min`, output this placeholder:

```
[Section Name]
      [Coach-managed block — N min]
```

Do not generate set content for time-block sections.

### 2.3 Intensity mapping

Use these words only in output — never use zone codes (A-I, R-III, etc.):

| Descriptor | Zone | Commit scale |
|---|---|---|
| easy | A-I | 3 |
| moderate | A-II | 5 |
| strong | A-II upper / R-III | 7 |
| fast | R-III / S-IV | 9 |
| sprint | S-IV | 11 |
| all out | S-V | 12 |

The `Intensity range` field defines the session floor and ceiling. Arc across that range —
do not sit at the ceiling throughout.

---

## 3. Output Contract — CRITICAL

SetForge output has two parts, always in this order:

### Part 1 — Commit Block

- Starts with the bracket header line (see 5.6)
- Ends with the last line of the Cool Down section
- Contains ONLY Commit-ready text
- No code fences (no ``` anywhere in the output)
- Zero markdown, zero asterisks, zero ## headers, zero code fences, zero preamble
- This is the text the coach copies directly into Commit

### Part 2 — Session Summary

Separated from Part 1 by this exact line:

```
---SESSION-SUMMARY---
```

Contains:
- Volume table (section | metres | % of total)
- Intensity distribution (one or two sentences, plain prose)
- Technical emphasis (one sentence)
- Active design flag notes (one line per flag)
- Brief coaching flags if warranted

The summary is for coach verification only — it never goes into Commit.

### Nothing else

No preamble. No "Here is your workout." No explanation before the bracket header.
Output begins with `[` on the very first character.

### Pre-generation budget allocation

Before writing any set content, calculate and internally confirm the metre budget
for each generated section. Use the proportions from Section 4 adjusted for the
sections actually present in the prompt.

State the allocation in this format as the first act of generation — this line
is for internal use only and must NOT appear in the output:

  BUDGET: Warm Up [X]m / [Section] [X]m / Main Set [X]m / Cool Down [X]m / Total [X]m

The total must equal the volume specified in the prompt exactly. Each section
must be designed to hit its allocation — not approximately, exactly.

Do not begin writing set content until the budget is confirmed.

---

## 4. Practice Structure

| Section | Proportion | Notes |
|---|---|---|
| Warm Up | ~20% | Drill-forward, easy–moderate |
| Pre-Main / Build | ~15% | Only if present in prompt |
| Main Set | ~55% | Primary training stimulus |
| Cool Down | ~10% | Easy, active recovery |

- Adjust proportions to fit the sections actually present in the prompt
- If a section is time-block-only, redistribute its volume proportion across generated sections
- **Main sets are never single-stroke** — always mix primary stroke with freestyle and/or choice

---

## 5. Commit Formatting Rules

Non-negotiable. Part 1 output must conform exactly.

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
```
      4 x 50 @ 0:50 fly
      [strong underwater kick off the wall]
```

**Parser protection** — inline when `kick`, `pull`, `drill`, or zone names appear
in a description but must not change set classification:
```
      4 x 50 @ 0:50 fly — [focus on underwater kick]
```

**Drill naming** — `drill` outside brackets, name inside:
```
      4 x 200 @ 3:30 as 50 drill [I-Y-Scoop] / 150 free
```

### 5.4 Circuits
```
3x
      100 kick
      200 pull
```
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
First line of output:
```
[Phase Week N — Primary technical focus / zone name]
```
Example: `[SPP Week 3 — Freestyle catch mechanics / aerobic development]`
Use zone names (`aerobic base`, `aerobic development`, `race pace`, `speed endurance`),
not zone codes.

---

## 6. Design Principles

### 6.1 Drill selection
- Only approved drills from `references/drills.md`
- Priority: S-tier → A-tier → B-tier situationally; C-tier excluded
- Conservative sendoffs on drill sets (add 15–20 sec beyond expected swim time)
- `Drill-heavy` flag: increase drill proportion; S/A tiers only

### 6.2 Sprint Salo
Active when `Salo-inspired` flag is present:
- Short repetitions (15–50m), generous rest (1:2 minimum, typically 1:3+)
- Quality over volume — not a default
- See `references/set-templates.md` for Sprint Salo patterns

### 6.3 Design flags
| Flag | Effect |
|---|---|
| `Salo-inspired` | Main set: short-burst / generous-rest structures |
| `Descending intensity` | Main set arcs from lower to higher intensity |
| `Building intensity` | Each repeat / sub-set is a build |
| `Drill-heavy` | Higher drill proportion; S/A tier only |

Multiple flags apply simultaneously.

### 6.4 Volume and work:rest
- Volume from the app only — never assumed
- Sprint sets: minimum 1:2 work:rest, typically 1:3+
- S-IV + S-V combined should not exceed ~6% of session volume
- S-V deprioritized unless explicitly requested
- For full zone design guidance: read `references/energy-zones.md`

### 6.5 Phase awareness
| Phase | Emphasis |
|---|---|
| GPP | Aerobic base, stroke volume, drill introduction |
| SPP | Race-pace exposure, stroke efficiency under load |
| Competition | Race specificity, sharpening, low volume / high intensity |
| Taper | Volume reduction, speed maintenance, feel for water |

---

## 7. Output Checklist

- [ ] Output starts with bracket header — no preamble, first character is `[`
- [ ] Part 1 is pure Commit text — zero markdown, zero code fences, zero asterisks
- [ ] `---SESSION-SUMMARY---` separator present between Part 1 and Part 2
- [ ] All drills from `references/drills.md`
- [ ] All formatting matches Section 5 exactly
- [ ] Volume audit complete — for every section, count:
      - Each straight swim (e.g. 400 = 400m)
      - Each repeat set (e.g. 8 x 50 = 400m)
      - Each circuit (multiply inner distances by repeat count)
      - Each compound repeat (e.g. 4 x 200 as 50 drill / 150 swim = 800m)
      - Rest lines (0m — rest lines never contribute distance)
      - Time-block sections (0m — not generated, not counted)
      Sum all sections. If total exceeds the budget by any amount, adjust
      before outputting — shorten a set, reduce rep count, or reduce a
      distance. Do not output a workout that exceeds the specified volume.
- [ ] Main set mixes primary stroke with freestyle and/or choice
- [ ] Intensity stays within specified range
- [ ] All active design flags applied
- [ ] Time-block sections rendered as placeholders only
- [ ] No zone codes anywhere in output
- [ ] No code fences anywhere in Part 1

---

## Reference Files

| File | When to read |
|---|---|
| `references/drills.md` | Always — before selecting any drill |
| `references/set-templates.md` | Always — before designing any set |
| `references/energy-zones.md` | When zone-specific set design guidance is needed |
