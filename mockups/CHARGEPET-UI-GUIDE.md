# ChargePet — UI & Game Mechanics Guide

> A gamified companion pet that reacts to your real phone charging data.
> Charge well → pet thrives. Overcharge or starve → pet suffers.

---

## Table of Contents

1. [Overview](#overview)
2. [Screen Layout](#screen-layout)
3. [Core Game Mechanics](#core-game-mechanics)
   - [Battery Health (HP)](#battery-health-hp)
   - [Experience Points (XP) & Rank Up](#experience-points-xp--rank-up)
   - [Session Classification](#session-classification)
4. [Pet Visual States](#pet-visual-states)
5. [The 20-Tier Rank System](#the-20-tier-rank-system)
6. [Tabs](#tabs)
   - [Stats Tab](#stats-tab--charging-heatmap--journey-map)
   - [Tips Tab](#tips-tab)
   - [Logs Tab](#logs-tab)
7. [Charging Heatmap](#charging-heatmap)
   - [Heatmap Modes](#heatmap-modes)
   - [Color Legend](#color-legend)
   - [Time-Travel on Tap](#time-travel-on-tap)
8. [Journey Map](#journey-map)
9. [AI Pet Chat](#ai-pet-chat-gemini-powered)
10. [Formula Reference Table](#formula-reference-table)

---

## Overview

ChargePet turns your phone's charging history into an RPG adventure. You load your real CSV charging data, and the app replays every charge session — rewarding good habits with XP and HP, and punishing bad habits (overcharging, battery starvation) with HP loss.

Your pet starts as a **Rank 1 Miles** with 100 HP. As you earn XP, it ranks up through 20 tiers, gaining new armor/equipment at each batch milestone. The goal: reach **Rank 20 — King/Emperor**.

---

## Screen Layout

```
┌─────────────────────────────────┐
│  [★ RANK 4]         [📅 Week 3] │  ← Header (sticky)
├─────────────────────────────────┤
│                                 │
│        💬 Speech Bubble         │
│         🟠 Pet Avatar           │  ← Pet Stage (animated)
│     (ears + body + equipment)   │
│                                 │
│  Asset: Centurion Cat (Batch 1) │
│          SPARKY                 │
│     [🐱 Cat] [🐶 Dog] [🐰]     │  ← Species selector
│  [Status: Thriving] [Talk ✨]   │
│                                 │
│  Battery Health (HP)    72/100  │  ← HP bar
│  ██████████████░░░░░░░░░░░░░░░  │
│  Exp to Rank Up         45/100  │  ← XP bar
│  █████████████░░░░░░░░░░░░░░░░  │
├─────────────────────────────────┤
│  [STATS]  [TIPS]  [LOGS]        │  ← Tab bar
├─────────────────────────────────┤
│                                 │
│  (Tab content area — scrollable)│
│                                 │
└─────────────────────────────────┘
```

### Header
- **Rank Badge** (left): Shows current rank number (e.g., "RANK 4"). *Easter egg: tap 5 times to unlock the Dev/Sandbox tab.*
- **Week Counter** (right): Shows how many weeks the CSV data spans.

### Pet Stage
- The pet is a CSS-drawn avatar with ears, eyes, snout, mouth, and stripes.
- **Species**: Switch between Cat, Dog, and Rabbit (cosmetic only — changes ear/snout shape).
- **Equipment**: Armor/gear drawn around the pet changes based on the current **batch** (see Rank System below).
- **Aura**: A glowing circle behind the pet that changes color with health state.

### Progress Bars
- **Battery Health (HP)**: 0–100. Visual representation of your pet's overall wellbeing.
- **Exp to Rank Up**: 0–100. When XP reaches 100, the pet ranks up and XP resets to 0.

---

## Core Game Mechanics

### Battery Health (HP)

HP represents your pet's overall health. It starts at **100** and changes with every charging session.

```
HP = clamp(HP + hpDelta, 0, 100)
```

HP determines the **pet's visual state**:

| HP Range | State | Pet Appearance |
|----------|-------|----------------|
| **61–100** | Thriving | Normal colors, floating animation, green aura, happy mouth |
| **31–60** | Tired | Desaturated, no float, gray aura, squinting eyes |
| **0–30** | Sick/Critical | Red-tinted, shaking animation, red aura, frowning mouth |

**HP bar color also changes:**
- HP > 60 → Green (`#81B29A`)
- HP 31–60 → Orange (`#F9A215`)
- HP ≤ 30 → Coral/Red (`#E07A5F`)

### Experience Points (XP) & Rank Up

XP is earned from every charging session. When XP reaches **100**, the pet **ranks up** and XP resets to 0.

```
xp += xpDelta
while (xp >= 100 AND rankIndex < 19):
    rankIndex++     // promote to next rank
    xp -= 100       // carry over excess XP
```

- At **Rank 20** (max), XP is capped at 100 — no further rank-ups.
- Total XP needed to reach Rank 20 from Rank 1: **1,900 XP** (19 rank-ups × 100 XP each).

> **Key insight**: Even bad charging sessions give *some* XP. Overcharges give 5 XP, starvation gives 10 XP. So the pet always progresses — just much slower with bad habits.

### Session Classification

Every charging session (plug in → plug out) is classified into one of **4 types** based on start and end battery percentages:

| Type | Condition | HP Change | XP Change | Meaning |
|------|-----------|-----------|-----------|---------|
| **🌟 Perfect Charge** | Start ≥ 20% AND End ≤ 85% | **+5 HP** | **+30 XP** | The ideal 20–80 range — pet loves this |
| **⚡ Standard Charge** | None of the other conditions | **+2 HP** | **+15 XP** | Acceptable, nothing special |
| **⚠️ Critical Starvation** | Start ≤ 10% | **−10 HP** | **+10 XP** | Battery dropped dangerously low — pet starved |
| **🔴 Overcharge Penalty** | End ≥ 95% | **−15 HP** | **+5 XP** | Charged to near-100% — pet overheated/burned |

**Priority order**: Overcharge is checked first, then Critical Start, then Perfect. A session cannot be both.

#### Session Score (for Heatmap)

Each session also gets a **score** used by the heatmap's Score mode:

| Type | Session Score |
|------|--------------|
| Perfect Charge | 100 |
| Standard Charge | 70 |
| Critical Starvation | 30 |
| Overcharge | 10 |

A day's **average score** = sum of session scores ÷ number of sessions that day.

---

## Pet Visual States

The pet has 3 health states that change its appearance:

### Thriving (HP > 60)
- Full color, no filters
- Gentle floating animation (up and down)
- Green pulsing aura behind the pet
- Normal round eyes, happy curved mouth
- Status badge: green "Thriving"

### Tired (HP 31–60)
- Desaturated (40% grayscale, dimmed)
- No floating — pet sits slightly lower
- Gray pulsing aura
- **Squinting eyes** (narrowed to 4px height)
- Status badge: gray "Tired"

### Sick / Critical (HP ≤ 30)
- Heavy color distortion (grayscale + sepia + hue shift → sickly red-green)
- **Shaking animation** (trembling side to side)
- Red pulsing aura (faster pulse)
- Frowning mouth (flipped upside down)
- Status badge: coral "Critical"

---

## The 20-Tier Rank System

Ranks are grouped into **5 batches**, each with different pet equipment/armor:

### Batch 1 — Basic (Ranks 1–6)
Brown leather collar

| Rank | Title |
|------|-------|
| 1 | Miles |
| 2 | Decanus |
| 3 | Optio |
| 4 | Centurion |
| 5 | Primus Pilus |
| 6 | Squire |

### Batch 2 — Knight Armor (Ranks 7–10)
Silver helmet with plume + dark chest armor with silver studs

| Rank | Title |
|------|-------|
| 7 | Knight Bachelor |
| 8 | Knight Banneret |
| 9 | Prefect |
| 10 | Tribune |

### Batch 3 — Officer Gear (Ranks 11–14)
Silver chest plate with visor + green shoulder capes

| Rank | Title |
|------|-------|
| 11 | Hazarapatis |
| 12 | Castellan |
| 13 | Legatus Legionis |
| 14 | Baivarapatis |

### Batch 4 — Noble Regalia (Ranks 15–18)
Gold crown + dark red royal cape + silver belt with gold medallion

| Rank | Title |
|------|-------|
| 15 | Baron/Lord |
| 16 | Earl/Count |
| 17 | Duke |
| 18 | Marshal |

### Batch 5 — Emperor (Ranks 19–20)
Triple gold crown spires + purple royal robe with white trim

| Rank | Title |
|------|-------|
| 19 | Grand Master |
| 20 | King/Emperor |

---

## Tabs

### Stats Tab — Charging Heatmap + Journey Map

The Stats tab has two sections stacked vertically:

1. **Charging Heatmap** (top, always visible) — GitHub-style grid showing daily charging patterns
2. **Journey Map** (below, scrollable) — Weekly progress timeline with star ratings

Both sections appear after running a simulation.

---

### Tips Tab

Contains:
- **"Ask Pet for Advice" button** — Uses Gemini AI to generate a personalized performance review in caveman-speak based on your charging stats
- **General Wisdom cards** — Static tips about the 20–80 rule and overnight charging dangers

---

### Logs Tab

- **CSV input area** — Pre-loaded with sample data or accepts file upload
- **"Run Simulation" button** — Processes the CSV and replays every session
- **Timeline cards** — Each charge session appears as a card showing:
  - Date and time
  - Start % → End %
  - Session type badge (Perfect / Standard / Critical / Overcharge)
  - HP and XP change

---

## Charging Heatmap

A GitHub-style contribution grid showing charging activity by day. Appears at the top of the Stats tab after running a simulation.

### Layout
- **Rows** = weeks (flipped from GitHub for mobile readability)
- **Columns** = Mon through Sun (M T W T F S S)
- **Month labels** on the left when the month changes
- Empty days (no charging data) shown as muted beige cells with lighter borders

### Heatmap Modes

Three toggle buttons let you switch what the cell colors represent:

#### 🔢 Sessions Mode
Colors based on **how many** charging sessions occurred that day.

| Sessions | Color | Hex |
|----------|-------|-----|
| 0 (no data) | Muted beige | `#E8E5D1` |
| 1 session | Orange | `#F9A215` |
| 2 sessions | Green | `#81B29A` |
| 3+ sessions | Gold | `#FACC15` |

#### ⭐ Quality Mode
Colors based on the **worst** session type that day (most punishing event wins).

| Worst Event | Color | Hex |
|-------------|-------|-----|
| Has overcharge | Coral (bad) | `#E07A5F` |
| Has critical starvation | Orange (warning) | `#F9A215` |
| All standard | Green (good) | `#81B29A` |
| All perfect | Gold (excellent) | `#FACC15` |

#### 📊 Score Mode (Default)
Colors based on the **average session score** for that day.

| Avg Score | Quality | Color | Hex |
|-----------|---------|-------|-----|
| ≥ 90 | Perfect day | Gold | `#FACC15` |
| 65–89 | Good day | Green | `#81B29A` |
| 40–64 | Okay day | Orange | `#F9A215` |
| < 40 | Bad day | Coral | `#E07A5F` |

### Color Legend

Below the grid, a legend strip shows:
```
Less  [■][■][■][■][■]  More
       ↑  ↑  ↑  ↑  ↑
      gray coral orange green gold
```

Plus a summary: *"10 active days · 20 sessions"*

### Time-Travel on Tap

Tapping any colored heatmap cell triggers a **full time-travel flashback**:

1. **Popover** appears showing:
   - Day name and date (e.g., "Thu, Oct 23")
   - Number of sessions
   - Average score (in Score mode)
   - Session type breakdown (✦ perfect, ⚠ overcharge, ▼ critical, ● standard)
   - ⚔ Rank and HP at end of that day

2. **Pet reacts** to that day's data:
   - **Health state changes** (thriving/tired/sick based on day's average score)
   - **HP bar updates** to show that day's cumulative HP
   - **XP bar updates** to show XP progress on that day
   - **Rank badge updates** to show what rank the pet was at
   - **Equipment/armor swaps** to match that day's batch
   - **Speech bubble** appears with a caveman-style comment about the day

3. **Tap outside** to dismiss — everything restores to the final simulation state.

#### Pet Comment Categories

| Day Quality | Pet State | Example Comment |
|-------------|-----------|-----------------|
| All perfect | Thriving | *"Me dance this day! You best human!"* |
| Mostly good | Thriving | *"This day me nap peacefully. You charge okay."* |
| Overcharges | Sick | *"Me cry this day! Why you do 100%?! WHY?!"* |
| Critical starvation | Sick | *"Me STARVE this day! You let battery die!"* |
| Mixed (good + bad) | Tired | *"This day confusing! Sometimes good, sometimes you hurt me!"* |

---

## Journey Map

A vertical timeline below the heatmap showing **weekly progress** with star ratings.

### Weekly Star Rating (Angry Birds Style)

Each week gets 1–3 stars:

| Stars | Condition |
|-------|-----------|
| ⭐⭐⭐ | Zero overcharges AND >50% of sessions are perfect |
| ⭐⭐☆ | 2 or fewer overcharges |
| ⭐☆☆ | Everything else |

### Weekly Node Colors

| Stars | Color |
|-------|-------|
| 3 stars | Gold (`#FACC15`) |
| 2 stars | Orange (`#F9A215`) |
| 1 star | Coral (`#E07A5F`) |

### Path to Emperor Banner

Above the journey map, a banner shows:
- **Estimated weeks to Emperor**: Based on average XP earned per week
- **Progress bar**: Cumulative XP earned / 1,900 total XP needed

---

## AI Pet Chat (Gemini-Powered)

Two Gemini AI features:

### "Talk ✨" Button (Pet Stage)
- Pet speaks a **1-sentence quote** in caveman English
- Content adapts to health state (boastful when thriving, complaining when sick)
- Quotes explain *why* the pet feels that way (overcharges vs. starvation)

### "Ask Pet for Advice ✨" (Tips Tab)
- Generates a **2-paragraph performance review** based on charging stats
- Praises good habits, scolds bad ones, gives actionable tips
- Uses bold markdown formatting for emphasis

Both features use `gemini-2.5-flash` with exponential backoff retry (up to 5 attempts).

---

## Formula Reference Table

| Metric | Formula | Range |
|--------|---------|-------|
| HP after session | `clamp(HP + hpDelta, 0, 100)` | 0–100 |
| XP after session | `XP + xpDelta` | 0–100 per rank |
| Rank up | Every 100 XP | Ranks 1–20 |
| Total XP to max rank | 19 × 100 = **1,900** | — |
| Week number | `floor((sessionDate - firstDate) / 7) + 1` | 1+ |
| Day avg score | `sum(sessionScores) / sessionCount` | 10–100 |
| Weekly stars | See star rating table above | 1–3 |
| Weeks to Emperor | `ceil((1900 - totalXP) / avgXpPerWeek)` | 1–∞ |

### HP Delta per Session Type

| Type | HP Delta | XP Delta | Session Score |
|------|----------|----------|---------------|
| Perfect (start ≥ 20, end ≤ 85) | +5 | +30 | 100 |
| Standard (default) | +2 | +15 | 70 |
| Critical (start ≤ 10) | −10 | +10 | 30 |
| Overcharge (end ≥ 95) | −15 | +5 | 10 |

### Pet Health Thresholds

| HP Range | Visual State | HP Bar Color |
|----------|-------------|-------------|
| 61–100 | Thriving | Green `#81B29A` |
| 31–60 | Tired | Orange `#F9A215` |
| 0–30 | Sick/Critical | Coral `#E07A5F` |

### Heatmap Day State (for pet reaction on tap)

| Avg Score | Pet State |
|-----------|-----------|
| ≥ 80 | Thriving |
| 50–79 | Tired |
| < 50 | Sick |
