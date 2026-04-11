# Project Brief — ChargePet Demo

## What is this?
A Flutter hackathon demo that visualizes real user charging behavior as a pet narrative. The pet (cat or dog) thrives or suffers based on how responsibly the user charged their phone.

## Source data
5 CSV files: `user_1.csv`, `user_2.csv`, `user_5.csv`, `user_7.csv`, `user_8.csv`

Format: `user_id, event_type, percentage, date, time, timezone`
Events: `power_connected` and `power_disconnected`

## Goal
Import CSV data, compute scores, show each user's story via:
1. An animated pet that reflects current health state
2. A timeline view (day-by-day) showing pet states over time
3. Session detail — what charging events happened and why they were good/bad

## What this is NOT
- Not a live app / real-time telemetry
- Not a production product
- Not connected to any backend

## Tech stack
- Flutter (mobile)
- Lottie (animation via `lottie` Flutter package)
- CSV parsed from assets (no network)

## Key files
- `JOURNEY.md` — all key decisions and reasoning
- `DESIGN.md` — visual design system
- `memory-bank/techContext.md` — technical details
- `memory-bank/activeContext.md` — what's being worked on right now
- `memory-bank/progress.md` — what's done, what's next
