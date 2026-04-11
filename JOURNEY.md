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

### D9: Precompute scores to JSON — no runtime CSV parsing
**Decision:** Run a one-time Dart/Python script that reads all 5 CSVs, computes sessions + scores + daily health, and outputs `assets/data/users.json`. Flutter loads JSON only.

**Why:** Codex outside voice caught that runtime parsing adds timezone bugs, cross-midnight session handling, and parser fragility for zero benefit — the data is fixed and never changes. Precomputing eliminates that entire risk layer.

**Script:** `scripts/precompute.dart` (or `.py`) — runs once before `flutter run`. Output format:
```json
{
  "users": [
    {
      "id": 1,
      "days": [
        { "date": "2025-10-23", "dailyScore": 12.5, "health": 12.5, "state": "sick",
          "sessions": [{ "start": 2, "end": 8, "score": 20, "flags": ["low_start"] }] }
      ]
    }
  ]
}
```

**Not in V1:** Runtime CSV parsing, CsvParser class, timezone handling.

---

### D10: Formula recalibrated to match narrative
**Decision:** Adjust scoring weights so User 7 actually lands SICK/CRITICAL after a week, and User 1's improving sessions score 70+.

**Root cause:** Original formula had conflicting end=80% rules (deduction of -10 AND bonus of +10 cancel out). Plus the base-50 was too generous — bad sessions still scored ~35 even with deductions.

**Recalibrated formula:**
- Base: 0 (not 50) — start from zero and earn the score
- Deductions become the rule for bad behavior, bonuses for good
- OR: Keep base 50 but fix the bonus/deduction overlap at end=80% boundary
  - end% < 80 → +10 (strictly less than, not ≤)
  - This way end=80% gets only the deduction (-10), not the bonus too

**Impact on User 7 (1%→99%):**
- start<15 → -25, end≥90 → -20 (tiered exclusive). Score: 50 - 25 - 20 = 5. Daily: 5.
- After 7 days of similar bad sessions → rolling health stays <20 → CRITICAL. ✓

**Impact on User 1 (17%→80%):**
- start<20 → -15, end≥80 → -10. No sweet spot (start<20). Score: 50 - 15 - 10 = 25.
- But Nov 12 (17%→80%) is an 8-hour overnight session — the *improvement* is visible in the trend, not one day's absolute score.

**Boundary fix for end=80%:** Change bonus condition from `end% ≤ 80` to `end% < 80`.

---

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

## 2026-04-12 — Eng Review decisions

### D11: Cross-midnight sessions → attribute to start date
**Decision:** Sort all events per user by absolute timestamp. Pair connected→disconnected in order. Attribute session to the date of the `power_connected` event.

**Why:** User 1 has 3 confirmed cross-midnight sessions, including the Nov 12 showcase session (17%→80%, 8h overnight). If grouped naively by the CSV `date` column, pairing would fail. Start date is more natural — the charging *decision* happened on Nov 12.

**Algorithm:**
```dart
// pseudocode for precompute.dart
final events = allEventsForUser.sortedBy(absoluteTimestamp);
for (i in 0..events.length-2 step 2) {
  if events[i].type == connected && events[i+1].type == disconnected {
    sessions.add(Session(startDate: events[i].date, ...));
  }
}
```

---

### D12: Orphaned events → skip + warn
**Decision:** If a `power_connected` event has no matching `power_disconnected`, skip it and print a warning.

**Why:** Known-clean data will produce zero warnings. For robustness, don't crash or create invalid session records.

---

### D13: Rolling health initialization → normalize by available weights
**Decision:** For the first 1–4 days of a user's data, normalize the rolling health weights by however many days are available. Don't zero-fill missing history.

**Normalization:**
- Day 1: health = D(today)  (only weight: 0.40 / 0.40 = 1.0)
- Day 2: health = (0.40×D(today) + 0.25×D(yesterday)) / 0.65
- Day 3: health = (0.40×D(3) + 0.25×D(2) + 0.15×D(1)) / 0.80
- Day 4: health = (0.40 + 0.25 + 0.15 + 0.10) / 0.90
- Day 5+: full formula (includes days 4-7 avg term, weight 0.10)

---

### D14: Inertia rule removed
**Decision:** Remove `D(day) = D(prev_day) × 0.95` for days with no sessions.

**Why:** Creates fake data. User 1 has a 9-day gap (Oct 29–Nov 7) with no sessions. The inertia rule would fill that gap with manufactured scores. Days with no sessions simply have no JSON entry. Rolling health on a day after a gap skips the empty days and normalizes over available data.

---

### D15: Weekly + monthly scores in V1
**Decision:** Keep `weeklyScore()` and add `monthlyScore()` to the ScoringEngine. Show all three (daily, weekly, monthly) on PetHomeScreen.

**Why:** Daily score alone is not very insightful for the judge demo. Weekly and monthly scores show the arc more clearly.

**Formulas:**
- `weeklyScore`: weighted avg of last 7 days with session data. `weight(d) = 7 - d` (today = 7, oldest = 1)
- `monthlyScore`: same pattern, 30-day window. `weight(d) = 30 - d`
- Both normalize over available days (skip days with no sessions)

---

### D16: JSON schema additions
**Decision:** Add time fields and score fields to the session and day objects.

**Updated session schema:**
```json
{
  "start": 1, "end": 99,
  "startTime": "11:42", "endTime": "13:08", "durationMins": 86,
  "score": 5,
  "flags": ["critical_discharge", "near_full"]
}
```

**Updated day schema:**
```json
{
  "date": "2025-10-23",
  "dailyScore": 10.0,
  "health": 4.0,
  "weeklyScore": 10.0,
  "monthlyScore": 10.0,
  "state": "critical",
  "sessions": [...]
}
```

**Canonical flag values:** `overcharged`, `near_full`, `borderline`, `critical_discharge`, `low_discharge`, `redundant`, `phantom`, `sweet_spot`

---

### D17: Phantom session score capped at 20
**Decision:** After computing a session score, if `(endPct - startPct) < 5`, cap the score at max 20.

**Why:** Without the cap, the `end<80 (+10)` and `start≥20 (+10)` bonuses exactly cancel the phantom `-20` penalty. A `42→42` session (0% gain) scored 50 — the same as a decent session. With the cap, it scores 20.

---

### D18: New pet state thresholds (shifted upward)
**Decision:** Shift all thresholds up by ~15 points.

| Health | Old | New |
|--------|-----|-----|
| CRITICAL | 0–19 | 0–29 |
| SICK | 20–39 | 30–59 |
| TIRED | 40–59 | 60–74 |
| HAPPY | 60–79 | 75–89 |
| THRIVING | 80–100 | 90–100 |

**Why:** Actual formula output for User 7 peaks at ~45. Old thresholds put that in TIRED. New thresholds put it in SICK, matching the narrative. Cross-validated by Codex outside voice.

---

### D19: Weekly + monthly scores are calendar-based
**Decision:** `weeklyScore` = last 7 calendar days (empty days = 0). `monthlyScore` = last 30 calendar days (empty days = 0). Calendar windows, not active-day windows.

**Why:** Active-day windows make gaps invisible. User 1's 9-day gap (Oct 29–Nov 8) should drag the weekly score down, not be silently skipped. "This week: 32/100" means the last 7 actual calendar days.

**Rolling health stays active-day-normalized** (different semantics: the pet's current health is about momentum, not accountability).

---

### D20: Precompute parser uses state machine (not index arithmetic)
**Decision:** The pairing algorithm uses an explicit `pendingConnect` variable.

```dart
DateTime? pendingConnect; String? pendingConnectDate;
for (event in sortedEvents) {
  if (event.type == 'power_connected') {
    if (pendingConnect != null) {
      // double connect — warn, replace with newer connect
      print("WARN: double connect for user ${event.userId} at ${event.date} ${event.time}");
    }
    pendingConnect = event;
  } else { // power_disconnected
    if (pendingConnect == null) {
      // orphaned disconnect — warn and skip
      print("WARN: disconnect without connect for user ${event.userId} at ${event.date}");
    } else {
      sessions.add(Session(start: pendingConnect, end: event));
      pendingConnect = null;
    }
  }
}
if (pendingConnect != null) {
  print("WARN: orphaned connect at end of data for user ${pendingConnect.userId}");
}
```

**Why:** Naive "pair by index" desynchronizes on any anomalous event. State machine is robust.

---

## Open questions
- [ ] Which Lottie files to use? (Search lottiefiles.com before Day 2)
- [ ] Should the pet be a cat or a dog? (User preference — add a picker?)
- [x] ~~How to handle orphaned CSV events~~ → skip + warn (D12, D20)
- [x] ~~What timezone to normalize to~~ → all events are +05:30, sort by absolute timestamp (D11)
