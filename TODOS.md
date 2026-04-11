# ChargePet — Deferred TODOs

Items identified during review but deferred from V1.

---

## T1: Verify all 5 user arcs after precompute runs

**What:** Run `dart scripts/precompute.dart`, then manually check each user's pet health arc.

**Why:** The scoring formula was calibrated against User 7 (bad charger) and User 1 (improving). Users 2, 5, and 8 haven't been manually verified. A flat arc (pet stays in one state the whole time) is boring for the demo. A chaotic arc (random state every day) is unreadable.

**Expected outcomes:**
- User 1: arc from SICK/TIRED (Oct) → HAPPY (Nov after improvement)
- User 7: CRITICAL/SICK throughout — this is the negative narrative anchor
- User 8: 73 sessions over ~40 days — should have the richest arc
- Users 2, 5: verify they're distinct enough to be interesting

**What to do if flat:** Adjust pet state thresholds in `pet_state.dart`. E.g., if all users end up TIRED (40-59), shift the TIRED/SICK boundary from 40 to 50.

**Depends on:** T0 (precompute script built and running)

**Effort:** ~15 minutes

---

## T2: V2 — Real-time device telemetry

**What:** Flutter background service that reads battery status API, records charging events, computes scores in real-time.

**Why:** The CSV demo proves the concept. Real-time turns it into a product.

**Why deferred:** Platform channels, background service, battery permissions — too complex for a 1-2 day sprint. The CSV data is real enough.

**Where to start:** `battery_plus` Flutter package for battery state. Store events in SQLite locally. Run precompute logic incrementally on each event.

---

## T3: Carbon impact per session

**What:** Calculate CO2 equivalent per overcharge session. Show "Your bad habits this month = X grams CO2."

**Why:** Makes the battery health angle tangible for judges who care about environmental impact.

**Why deferred:** Requires a credible CO2/kWh conversion model. Not enough time to research proper numbers for a hackathon.

---

## T4: Social comparison — share your pet

**What:** Share pet health score and arc as a card image. "My pet is SICK this week. How's yours?"

**Why deferred:** No backend. Sharing would require image generation + share sheet integration.
