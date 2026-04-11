# ChargePet — Journey Log

Key decisions, pivots, and ideas. Updated as we build.

---

## 2026-04-12 — Project kickoff (office hours + CEO review session)

### What we're building
A Flutter hackathon demo that presents real user charging data (5 CSV files) as a narrative — told through a pet that thrives or suffers based on how responsibly the user charged their phone.

**This is NOT a live app.** It is a data visualization/storytelling demo. No real-time telemetry, no backend, no auth. Import CSV, compute scores, tell the story.

---

### Key decisions

#### D1: Lottie for animation (not Rive, not sprites)
**Decision:** Use `lottie` Flutter package with 5 pre-made Lottie files from lottiefiles.com.

**Why:** Solo 1-2 day hackathon. Creating or wiring Rive state machines takes days. Lottie files are drop-in assets — find a JSON, reference it, done. 5 states = 5 files, swap on state change.

**Where to find:** lottiefiles.com, search "cat sad", "cat happy", "cat sick", "sleeping cat"

**Rejected:** Custom Rive animation (too slow to create). Rive community files (state machine wiring adds risk). Sprite sheets (lowest wow factor).

---

#### D2: Scoring formula
**Decision:** Session-level score (0–100) with deductions for bad patterns and bonuses for sweet-spot behavior. Rolled up into daily, rolling pet health (7-day weighted), and weekly scores.

**The formula:**

Session base = 50

Deductions:
- end% = 100 → -30 (overcharge)
- end% >= 90 → -20
- end% >= 80 → -10
- start% < 15 → -25 (deep discharge)
- start% < 20 → -15
- start% > 85 → -15 (redundant charge)
- (end% - start%) < 5 → -20 (phantom session)

Bonuses:
- 20 ≤ start% ≤ 50 AND 60 ≤ end% ≤ 80 → +30 (sweet spot)
- end% ≤ 80 → +10
- start% ≥ 20 → +10

Daily score = mean of session scores that day.
No sessions: D(day) = D(prev) × 0.95

Rolling pet health (7-day weighted):
```
Health = 0.40 × D(today)
       + 0.25 × D(yesterday)
       + 0.15 × D(2 days ago)
       + 0.10 × D(3 days ago)
       + 0.10 × avg(D(days 4–7))
```

Weekly score = weighted avg with more recent days counting more (weight 7 → 1).

Pet states:
- 80–100: Thriving
- 60–79: Happy
- 40–59: Tired
- 20–39: Sick
- 0–19: Critical

**Why this formula:** Captures the two worst behaviors (overcharging + deep discharge), rewards the "20-80% zone" that battery research consistently shows is optimal, and the inertia in rolling health makes the pet's arc feel organic rather than jerky.

---

#### D3: Data-first build order
**Decision:** Build CSV parser + scoring engine first. Animation and UI last.

**Why:** The math is the product. If scoring_engine.dart works, the demo works. The narrative is just presentation on top. Chasing animations on Day 1 is the #1 risk to shipping.

**Build order:**
1. CsvParser (event pairs → ChargingSession)
2. ScoringEngine (session → daily → weekly → health)
3. Unit tests with User 7 (should score ~5-15/day)
4. UserSelectScreen
5. PetHomeScreen (static scores first)
6. Download Lottie files
7. PetWidget (Lottie wrapper)
8. TimelineScreen
9. DayDetailScreen
10. Polish

---

#### D4: Multiple users as contrast
**Decision:** Show all 5 users selectably, not just one.

**Why:** The story is more compelling with contrast. User 7 (consistently bad) vs User 1 (improving) is a narrative arc judges can follow. "Here's someone who learned" is more interesting than "here's someone who's bad."

---

#### D5: Storyboard — the two stories to demo
**User 7 (bad charger):** Arrives, charges 1% → 100% on day 1. Pet is immediately sick. Continues bad habits. Pet health stays in 5–20 range all month.

**User 1 (improving charger):** Starts badly (100% charges, deep discharges). November shows improvement — starts charging 20–40%, stops at 80%. Pet health climbs from ~20 to ~70+ by end of month.

These two stories should be the demo script.

---

### What we decided NOT to build
- Live telemetry / real-time device data
- User auth / login
- Custom animation creation (Rive from scratch)
- Push notifications
- Backend / cloud sync
- App store build

---

### Landscape context (from session research)

Most gamified sustainability apps fail at habit persistence post-engagement. But this is a retrospective narrative demo, not a live product — so drop-off mechanics don't apply. The pet suffering from bad behavior IS the dramatic point, not a retention problem.

Virtual pet guilt mechanics (pet death) cause drop-off in live apps. For a demo, the drama is a feature.

Apple's Clean Energy Charging is the gold standard for real behavior change: automation + no friction. Out of scope for this demo but worth noting as the "ideal world" end state.

---

### Ideas for later (not in scope now)
- Rive animation with proper state machine (post-hackathon if this gets productized)
- Real-time device telemetry via a Flutter background service
- Social features: compare your pet's health with friends
- Push notifications: "Your pet is getting sick — unplug!"
- Carbon impact calculation per overcharge session (g CO2 equivalent)
- Battery degradation simulation: cumulative damage visible in pet's "age"

---

---

## 2026-04-12 — CEO Review decisions

### D8: Scoring deductions are tiered exclusive (not cumulative)
**Decision:** Overcharge/discharge deductions use if-else tiers. A session that ends at 100% gets -30 (not -10-20-30).

**Why:** Cleaner to explain to judges. Formula is transparent. Both approaches clamp to 0 for worst-case sessions anyway.

**Code pattern:**
```dart
// end% deductions — tiered exclusive
if (endPct == 100) { score -= 30; }
else if (endPct >= 90) { score -= 20; }
else if (endPct >= 80) { score -= 10; }
// start% deductions — tiered exclusive
if (startPct < 15) { score -= 25; }
else if (startPct < 20) { score -= 15; }
```

---

### D6: V1 is CSV-only, no live telemetry
**Decision:** This build does not implement any live device data gathering. V1 = import CSV, compute score, show story. Real-time telemetry is V2.

**Why:** Solo 1-2 day hackathon. Adding live device telemetry increases complexity massively (platform channels, background service, permissions). The CSV data is real — it's enough to tell a compelling story.

**V2 plan:** Real-time scoring using Flutter background service + battery status API. Record session data locally, compute scores incrementally.

---

### D7: Tip box / recommendations on score screen
**Decision:** Add a contextual "How to improve" tip box on PetHomeScreen and DayDetailScreen.

**Format:**
- PetHomeScreen: 1-2 tips based on the most recent week's worst pattern
- DayDetailScreen: per-session advice line (already in design, confirmed)

**Tip content per pattern:**
- Overcharge (end 100%): "Unplug at 80% to extend battery life"
- Deep discharge (start <15%): "Charge before reaching 15% to reduce battery stress"
- Redundant charge (start >85%): "Your battery was already full — no need to plug in"
- Phantom session (<5% gain): "Short plug-in sessions wear the battery without helping it"
- Good session: "Sweet spot! Staying between 20–80% is ideal for battery health"

---

## Open questions
- [ ] Which Lottie files to use? (Search lottiefiles.com before Day 2)
- [ ] Should the pet be a cat or a dog? (User preference — add a picker?)
- [ ] How to handle orphaned CSV events (connected with no disconnect)?
- [ ] What timezone to normalize to for timeline display?
