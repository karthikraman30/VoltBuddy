# ChargePet — Agent Context

This file provides context for AI agents (Codex, Claude subagents) working on this project.

## What this codebase is

Flutter hackathon demo. Real CSV data from 5 phone users. Scoring formula maps
charging behavior to a pet health score. Pet is animated via Lottie. No backend.
No auth. No live telemetry. V1 = precomputed JSON + 4 screens.

## Critical: read JOURNEY.md before touching any scoring code

The scoring formula has gone through 3 rounds of review and has specific boundary
conditions that look wrong but are intentional. Do not "fix" them without checking
JOURNEY.md first.

The most dangerous boundaries:
- `endPct < 80` is STRICT (not ≤). Sessions ending at exactly 80% get a deduction
  but NOT the bonus. This is intentional (D10).
- Phantom cap: if `(endPct - startPct) < 5`, cap score at 20 AFTER clamping to [0,100].
  Do not remove this post-processing step (D17).
- Pet state thresholds: SICK = 30-59, not 20-39. The old thresholds are wrong (D18).

## Project structure

```
chargepet/              ← Flutter app root (run commands from here)
  assets/csv/           ← source CSVs (do not modify)
  assets/animations/    ← Lottie JSON files (download if missing)
  assets/data/          ← users.json (generated, do not commit)
  scripts/              ← precompute.dart lives here
  lib/                  ← Flutter source
  test/                 ← scoring_engine_test.dart (run first)
```

## Commands

```bash
# From chargepet/
flutter pub get
dart run scripts/precompute.dart   # must run before flutter run
dart test test/scoring_engine_test.dart
flutter run
```

## Scoring formula reference

See CLAUDE.md for the full Dart implementation. Do not change without running
all tests and updating JOURNEY.md with the decision.

## JSON schema

See JOURNEY.md D16 for the full JSON schema including all fields.
The schema includes: `schemaVersion`, `generatedAt`, `formulaVersion` at root.
Per-session: `start`, `end`, `startTime`, `endTime`, `durationMins`, `crossesMidnight`, `score`, `flags`.

## Do not

- Parse CSVs at Flutter runtime (they are in assets/csv/ for precompute only)
- Add the `csv` package to pubspec.yaml (precompute.dart runs as a Dart script, not Flutter)
- Use `provider` or `riverpod` for state (StatefulWidget is enough for hackathon)
- Implement `weeklyScore` using a sliding window of active days (it must be calendar-based — D19)
