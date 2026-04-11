# Technical Context

## Flutter project structure
```
lib/
├── main.dart
├── data/
│   ├── csv_parser.dart       # Raw CSV → ChargingEvent list, pair into sessions
│   └── models.dart           # ChargingEvent, ChargingSession, DayScore
├── scoring/
│   └── scoring_engine.dart   # All math: session, daily, weekly, rolling health
├── pet/
│   ├── pet_state.dart        # PetState enum + PetStateMapper
│   └── pet_widget.dart       # Lottie wrapper, swaps asset on state change
├── screens/
│   ├── user_select_screen.dart
│   ├── pet_home_screen.dart
│   ├── timeline_screen.dart
│   └── day_detail_screen.dart
└── theme/
    └── app_theme.dart
```

## Key dependencies (pubspec.yaml)
```yaml
dependencies:
  flutter:
    sdk: flutter
  lottie: ^3.1.0        # Lottie animation player
  csv: ^6.0.0           # CSV parsing
  intl: ^0.19.0         # Date formatting
  provider: ^6.1.2      # State management (or riverpod, your call)
```

## Scoring formula
See JOURNEY.md → D2 for full formula.

Summary:
- Session score: 0–100, base 50, deductions for overcharge/deep-discharge/redundant/phantom, bonus for sweet-spot (20–50% start, 60–80% end)
- Daily score: mean of session scores
- Rolling health: 7-day weighted (40% today, 25% yesterday, 15% 2d ago, 10% 3d ago, 10% 4-7d avg)
- Weekly score: weighted avg, recent days count more

## Pet states
```dart
enum PetState { thriving, happy, tired, sick, critical }
```
Thresholds: 80–100 thriving, 60–79 happy, 40–59 tired, 20–39 sick, 0–19 critical

## Animation assets
`assets/animations/`:
- `pet_thriving.json`
- `pet_happy.json`
- `pet_tired.json`
- `pet_sick.json`
- `pet_critical.json`

Source: lottiefiles.com — search "cat happy", "cat sad", "cat sick", "sleeping cat"

## CSV structure
Columns: `user_id, event_type, percentage, date, time, timezone`
- `event_type`: `power_connected` or `power_disconnected`
- `percentage`: battery % at event time
- `date`: YYYY-MM-DD
- `time`: HH:MM:SS
- `timezone`: +HH:MM offset

Parse into `ChargingEvent`, pair consecutive connected/disconnected events into `ChargingSession(start%, end%, startTime, endTime)`.

Edge cases to handle:
- Orphaned event (no matching pair): skip
- Very short sessions (same minute): flag as phantom if delta < 5%
- User 7 has 99% → 100% session (plugged in at 99%): scores as -30 -15 = 5
