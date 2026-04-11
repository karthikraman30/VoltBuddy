# Active Context

## Current focus
Pre-build planning phase. Design doc complete, project structure scaffolded.

## Next action
Start building: `flutter create chargepet` → copy CSVs → build `csv_parser.dart` → build `scoring_engine.dart`.

## What's decided
- Lottie for animation (not Rive)
- 5 pet states: thriving / happy / tired / sick / critical
- Scoring formula finalized (see JOURNEY.md → D2)
- Build order: data layer first, UI second, animation last
- Demo stories: User 7 (bad) vs User 1 (improving arc)

## What's open
- Which Lottie files to use (search lottiefiles.com before Day 2)
- Cat vs dog vs choice (can add a picker on UserSelectScreen)
- Timezone normalization for timeline display

## Risks
1. Animation sourcing — don't start Day 2 without Lottie files downloaded
2. Orphaned CSV events — handle gracefully in parser, don't crash
3. Timeline scroll UX — keep it simple, horizontal list is fine
