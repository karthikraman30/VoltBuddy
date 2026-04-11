# DESIGN.md — ChargePet Visual Design System

<!-- Import this into your design tool -->

## Brand voice
Warm, playful, slightly anxious. The pet is real. The data is real. We want judges to feel the narrative, not just read numbers.

## Color palette

### Primary
| Token | Hex | Use |
|-------|-----|-----|
| `petGreen` | `#4CAF50` | Thriving / healthy states |
| `petYellow` | `#FFC107` | Happy / neutral |
| `petOrange` | `#FF9800` | Tired / mild concern |
| `petRed` | `#F44336` | Sick / bad session |
| `petCritical` | `#B71C1C` | Critical / emergency |

### Backgrounds
| Token | Hex | Use |
|-------|-----|-----|
| `bgDark` | `#1A1A2E` | App background (night sky mood) |
| `bgCard` | `#16213E` | Card backgrounds |
| `bgSurface` | `#0F3460` | Surface / elevated elements |

### Text
| Token | Hex | Use |
|-------|-----|-----|
| `textPrimary` | `#EAEAEA` | Main text |
| `textSecondary` | `#9E9E9E` | Subtitles, metadata |
| `textAccent` | `#E94560` | Highlights, score callouts |

## Typography
- **Display (pet state):** 48sp, bold, petGreen/petRed based on state
- **Score:** 36sp, bold, textAccent
- **Body:** 16sp, regular, textPrimary
- **Caption:** 12sp, regular, textSecondary
- Font: System default (Roboto/SF Pro — no custom fonts for hackathon speed)

## Pet state color mapping
```
THRIVING   →  petGreen    (#4CAF50)
HAPPY      →  petYellow   (#FFC107)
TIRED      →  petOrange   (#FF9800)
SICK       →  petRed      (#F44336)
CRITICAL   →  petCritical (#B71C1C)
```

## Component patterns

### Score card
```
┌─────────────────────┐
│  Pet Health         │   ← caption, textSecondary
│  34 / 100           │   ← display, textAccent
│  ████░░░░░░░░  34%  │   ← progress bar, petRed
└─────────────────────┘
```

### Timeline day chip
```
┌─────────┐
│  😿     │   ← pet emoji or Lottie thumbnail
│  Oct 23 │   ← caption
│  5/100  │   ← score, colored by state
└─────────┘
```

### Session card (day detail)
```
┌──────────────────────────────────────┐
│  Session 1  •  19:00 – 20:02  🔴    │
│  01% → 41%  (+40%)  Score: 15       │
│  ⚠ Started critically low (1%)      │
└──────────────────────────────────────┘
```

## Screen layouts

### Pet Home Screen
- Dark background (bgDark)
- Centered Lottie pet (60% screen height)
- Health score below pet (large, colored)
- State label (SICK / HAPPY etc.)
- 7-day trend line (small chart, bottom)
- "View Timeline" button (bottom)

### Timeline Screen
- Top: user name / ID
- Horizontal scroll: day chips (compact, colored by state)
- Selected day: session list below
- Each session: card with start%, end%, score, flags

### Day Detail Screen
- Date header
- List of sessions
- Each session: timing, % delta, score, advice text
- Daily score summary at bottom

## Animation guidelines
- Pet entrance: fade in + scale up (300ms)
- State change: crossfade between Lottie files (400ms)
- Timeline scroll: physics-based fling
- Score update: count-up animation (500ms)
- No gratuitous animations — every motion has a purpose

## Iconography
- Overcharge: ⚡ or 🔋 with red indicator
- Deep discharge: 📉
- Sweet spot: ✅ or 💚
- Phantom session: ⏱
- Critical: 🆘

### Tip box (recommendation card)
```
┌────────────────────────────────────────────┐
│  💡 This week's tip                        │   ← icon + label
│  Your pet got sick from charging to 100%   │   ← body text, textPrimary
│  on Tuesday. Try unplugging at 80%.        │
└────────────────────────────────────────────┘
```
- Background: bgCard with left border accent (petRed for warnings, petGreen for praise)
- Show on: PetHomeScreen (below score), DayDetailScreen (per session)
- Logic: most frequent bad pattern in last 7 days → one tip
- If no bad patterns: "Great week! Your pet is thriving."

Tip text library:
| Pattern | Tip |
|---------|-----|
| end=100 | "Try unplugging at 80% — charging to 100% shortens battery life" |
| start<15 | "Your battery drained too low. Charging before 20% reduces battery stress" |
| start>85 | "No need to plug in — your battery was already full" |
| phantom | "Short charging sessions don't help much. Wait until you're below 40%" |
| sweet spot | "Perfect charging session! Staying between 20–80% is ideal." |

## Accessibility (minimum bar for hackathon)
- Color is never the only indicator (always pair with icon or label)
- Pet state label text always visible alongside animation
- Score numbers always present, not just color
