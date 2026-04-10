---
name: swim-workout
description: >
  Use this skill whenever building, drafting, formatting, or editing a competitive swim
  practice or workout. Triggers include any request to write a practice, build a set,
  format a workout for Commit Swimming, plan a training week, or design a session for
  any group or phase. Always read this skill before generating any workout output —
  even partial sets, warm-ups, or single main sets. This skill governs formatting,
  drill selection, set structure, volume targets, and Commit parsing conventions.
---

# Swim Workout Skill

## Overview

This skill governs all competitive swim practice design and formatting. It ensures
consistent, predictable output across sessions — correct Commit formatting, approved
drill vocabulary, and structurally sound set design aligned with Swimming Canada's
AAD/LTAD framework.

Always read this SKILL.md fully before writing any workout content.
For drill reference, read `references/drills.md`.
For set templates, read `references/set-templates.md`.

---

## 1. Context

**Primary group**: Velocity (Ben's group)  
**Pool**: 25-metre  
**Session length**: 90 minutes, water only (dryland handled separately)  
**Schedule**: Monday, Tuesday, Wednesday, Friday, Saturday  
**Framework**: Swimming Canada AAD/LTAD — Train to Train (T2T)  
**Platform**: Commit Swimming (all output must parse correctly — see Section 3)  
**Equipment available**: fins, snorkel, paddles, kickboard, pull buoy  

---

## 2. Practice Structure

Every session follows this arc:

**Total session volume must be explicitly provided or confirmed before building a workout.
If not given, ask before proceeding. Do not assume or default to any target.**

Once volume is established, distribute it approximately as:

| Section | Proportion | Notes |
|---|---|---|
| Warm Up | ~20% | Drill-forward, low intensity, A-I zone |
| Pre-Main / Build | ~15% | Optional; transitions to main set focus |
| Main Set | ~55% | Primary training stimulus |
| Cool Down | ~10% | Easy, active recovery |

**Zone reference** (Swimming Canada):
- A-I: aerobic base / easy
- A-II: aerobic development / moderate
- R-III: race pace / threshold
- S-IV: speed / above threshold
- S-V: sprint / maximal

---

## 3. Commit Formatting Rules

These rules are non-negotiable. All output must conform to the Commit parsing engine.

### 3.1 Spacing and syntax
- Spaces around "x": `4 x 100` ✓ — never `4x100`
- Sendoffs: `@ 0:50`, `@ 1:25` — strictly for sendoffs only, never for rest. Always use a leading zero for sub-minute times (e.g. `@ 0:50` not `@ :50`) — Commit doesn't require it but it is the preferred format
- Rest: written as part of the set description, e.g. `2:00 rest` on its own line
- Intensity: write `sprint` or `all out` — never `allout`, `spr`, or `max`

### 3.2 Set formatting
- Commit's parser understands natural language descriptions — write sets plainly
- Numbered groupings within a repeat are supported: `12 x 50 1-4 kick @ 1:00, 5-8 paddle pull @ 0:55, 9-12 swim @ 0:50`
- Odds/evens constructions are understood: `6 x 100 on 1:25 odds sprint evens easy`
- Distance-based breakdowns: `300 choice (50 drill 25 swim)` — parser assigns distances correctly
- Timed sets without a distance: `6:00 easy swimming, focus on technique` — parser implies distance from default 100 pace; use this format intentionally
- Rest periods: `2:00 rest` on its own line — adds time to total without implying a swim distance

### 3.3 Bracket rules — CRITICAL
Brackets `[ ]` block the parser from interpreting their contents. Use them in two distinct ways:

**Coaching notes** — always on their own line:
```
  4 x 50 @ 0:50 fly
  [strong underwater kick off the wall]
```

**Protecting descriptions from misparse** — inline on the set line, when a word like `kick`, `drill`, `pull` appears in a description but should NOT change how the set is classified:
```
  4 x 50 @ 0:50 fly - [strong underwater kick off the wall]
```
Without brackets, the word `kick` would cause Commit to classify this as a kick set.
With brackets, the parser ignores the bracketed content — the set is read as fly.

**Drill naming** — when a drill appears in a set, `drill` must be outside brackets so the parser recognizes it as a drill set; the drill name goes inside brackets:
```
  4 x 200 @ 3:30 as 50 drill [I-Y-Scoop] / 150 free
```

### 3.4 Circuits
Commit supports circuits (rounds-based sets) using indentation. The circuit header is the repeat count; everything tabbed beneath it is included in the circuit.

**Basic circuit:**
```
3x
  100 kick
  200 pull
```
The repeat count line (`3x`) can be written as `3x`, `3 x`, or `3 times through` — Commit reads the leading number and treats the indented lines as the circuit contents.

**Nested circuits** — circuits inside circuits are supported and calculated correctly:
```
2x
  100 kick
  4x
    3 x 50s
    100 easy
```
Commit calculates total volume through all nesting levels correctly.

**Circuit sendoffs** — sendoffs are applied to individual lines inside the circuit, not to the circuit header line. How Commit aggregates these is still being worked out, so keep sendoffs on the individual set lines within a circuit for now.

**Use circuits when**: a set has multiple components done as rounds, or when a main set has a complex inner repeat structure. Circuits keep the workout clean and ensure Commit's volume calculation is accurate.

### 3.5 Section headings and indentation
- Any text can be a section heading — Commit does not require specific heading names
- **Warning**: if a section heading contains `kick`, `pull`, or an energy system name (e.g. `Aerobic`, `Threshold`), Commit will apply that classification to every set in that section. Use neutral headings or bracket the problematic word to avoid this
- **Sets must be indented exactly 6 spaces beneath their section heading** — fewer spaces will not parse correctly
- Coaching notes also indented 6 spaces, on their own line
- No extra blank lines between sets within a section
- One blank line between sections

### 3.6 Workout header
The header line must be wrapped in square brackets so the parser ignores it entirely:
```
[SPP Week 14 — Freestyle catch mechanics / A-II aerobic development]
```
Include: phase, week number, primary technical focus, primary zone.

### 3.7 Example — correct formatting

```
[SPP Week 14 — Breaststroke pull / A-I aerobic base]

Warm Up
      400 as 100 free / 100 kick / 100 back / 100 choice
      [easy effort, long stroke]
      12 x 50 1-4 kick @ 1:00, 5-8 drill [Catchup Drill] @ 1:00, 9-12 free @ 0:50

Main Set
      4 x 200 @ 3:30 as 25 drill [Two Kick One Pull] / 75 breast
      [elbows high, hands inside elbows on the scoop]
      6 x 100 @ 1:25 odds sprint evens easy

Cool Down
      300 choice (50 drill 25 swim)
      2:00 rest
```

---

## 4. Design Principles

### 4.1 General
- Never prescribe drills outside the approved drill library (see `references/drills.md`)
- When introducing a drill, choose one that fits the session's technical objective
- Sendoffs on drill-heavy sets should be conservative — give athletes time to execute
- Sprint Salo-style sets (short bursts, generous rest) are a deliberate design choice, not a default — use when the session calls for speed development

### 4.2 Freestyle focus areas
- EVF (Early Vertical Forearm) / catch mechanics
- Stroke count / DPS (distance per stroke)
- High-elbow recovery

### 4.3 Breaststroke focus areas
- I-Y-Scoop pull pattern (three phases: catch → high-elbow → inward scoop)
- Typically performed with pull buoy + snorkel
- Full stroke timing: pull, breathe, kick, glide sequencing

### 4.4 Volume and intensity balance
- Drill-heavy sessions skew toward A-I/A-II
- Speed sessions (R-III/S-IV/S-V) require adequate warm-up and recovery ratios
- Work-to-rest ratio for sprint sets: minimum 1:2, often 1:3 or greater

---

## 5. Seasonal Phase Awareness

When a seasonal plan is active, confirm the current week/phase before building a session.
Match the session's technical objective and intensity zone emphasis to the plan.

| Phase | Primary Focus |
|---|---|
| GPP (General Prep) | Aerobic base, stroke volume, drill introduction |
| SPP (Specific Prep) | Race-pace exposure, stroke efficiency under load |
| Comp (Competition Prep) | Race specificity, sharpening, low volume / high intensity |
| Taper | Volume reduction, speed maintenance, feel for water |

---

## 6. Output Checklist

Before finalizing any workout, confirm:
- [ ] All formatting matches Commit conventions (Section 3)
- [ ] All drills are from the approved library (`references/drills.md`)
- [ ] Total volume was provided or confirmed before building — never assumed
- [ ] Coaching notes are on their own lines in square brackets
- [ ] Section headings are flush left
- [ ] Intensity zones are appropriate for the session's phase and objective
- [ ] Sendoffs on drill sets are conservative

---

## Reference Files

- `references/drills.md` — Approved drill library by stroke and focus
- `references/set-templates.md` — Reusable set structures and Sprint Salo formats
