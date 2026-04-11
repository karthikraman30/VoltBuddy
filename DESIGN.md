# DESIGN.md — ChargePet Visual Design System (Neo-Brutalist RPG)

## Brand Identity

**Voice:** Bold, tactile, and rewarding. The UI feels like a premium indie game menu or a tabletop character sheet. Physical metaphors: thick lines, heavy shadows, high-contrast "paper" surfaces.

**Emotional arc:** The pet's story should feel like a real RPG — the user is the hero or villain of their own battery narrative. CRITICAL is not a warning modal, it's a death screen.

---

## Color Palette

### Core RPG Colors

| Token | Hex | Use |
|-------|-----|-----|
| `primary` | `#f9a215` | Score gold, level-up highlights, XP bars |
| `rpgBg` | `#F4F1DE` | App background (aged parchment) |
| `rpgText` | `#3D405B` | Borders, shadows, headings, body text |
| `rpgSurface` | `#81B29A` | Healthy states, positive feedback |
| `rpgAccent` | `#2A9D8F` | Buttons, links, positive CTAs |
| `rpgMuted` | `#E07A5F` | Penalties, overcharge warnings (coral) |

### Pet State Color Mapping

| State | Color | Hex | Meaning |
|-------|-------|-----|---------|
| THRIVING | `rpgSurface` | `#81B29A` | Full health — green |
| HAPPY | `rpgAccent` | `#2A9D8F` | Good condition — teal |
| TIRED | `primary` | `#f9a215` | Caution — gold/amber |
| SICK | `rpgMuted` | `#E07A5F` | Penalty — coral red |
| CRITICAL | `#C1440E` | `#C1440E` | Emergency — deeper red-orange |

> `CRITICAL` uses a darkened variant of `rpgMuted` for maximum urgency. Never use `rpgSurface` or `rpgAccent` for states below HAPPY.

### Score Bar Colors

| Score | Color |
|-------|-------|
| 80–100 | `rpgSurface` (#81B29A) |
| 60–79 | `rpgAccent` (#2A9D8F) |
| 40–59 | `primary` (#f9a215) |
| 20–39 | `rpgMuted` (#E07A5F) |
| 0–19 | `#C1440E` |

---

## Typography

**Font:** Space Grotesk (via `google_fonts` package)

| Role | Size | Weight | Transform | Tracking |
|------|------|--------|-----------|---------|
| Display (pet state label) | 48sp | 900 (black) | UPPERCASE | normal |
| Score (number) | 36sp | 900 (black) | — | normal |
| Heading | 24sp | 700 (bold) | UPPERCASE | wide |
| Body | 16sp | 400 (regular) | — | normal |
| Label / metadata | 10sp | 500 (medium) | UPPERCASE | widest |

```dart
// Usage in Flutter
import 'package:google_fonts/google_fonts.dart';

TextStyle displayStyle = GoogleFonts.spaceGrotesk(
  fontSize: 48, fontWeight: FontWeight.w900,
);
TextStyle labelStyle = GoogleFonts.spaceGrotesk(
  fontSize: 10, fontWeight: FontWeight.w500,
  letterSpacing: 2.0,
);
```

---

## UI Principles

### 1. Neo-Brutalist Construction

**Borders:** All primary containers and buttons: `Border.all(color: Color(0xFF3D405B), width: 4)`

**Hard Shadows:** Solid offset shadow — no blur. Simulates 2D-layered depth.
```dart
BoxDecoration(
  color: cardColor,
  border: Border.all(color: Color(0xFF3D405B), width: 4),
  boxShadow: [
    BoxShadow(
      color: Color(0xFF3D405B),
      offset: Offset(4, 4),
      blurRadius: 0,  // NO BLUR
    ),
  ],
)
```

**Tactile Feedback:** On press, translate +4px down/right AND remove shadow. Use `GestureDetector` + `AnimatedContainer`:
```dart
// pressed = true on tap down, false on tap up
offset: pressed ? Offset(4, 4) : Offset(0, 0),
shadow: pressed ? [] : [BoxShadow(...)],
```

**Rounded corners:** `BorderRadius.circular(16)` for cards (`rounded-2xl`). Buttons use `BorderRadius.circular(8)`.

### 2. Narrative States & Feedback

**Active states (pet is suffering or thriving):** Use `rpgMuted` or `rpgSurface` border with a subtle pulse animation. Apply sparingly — only on the pet health bar and the current state label.

**Retired/History items:** Timeline days far in the past use reduced opacity (0.7) and desaturated tint to signal "this is history, not now."

**Penalty indicators:** SICK and CRITICAL states use `rpgMuted` coral text. Include `⚠` icon alongside color — never color alone (accessibility).

### 3. Iconic Language

| Concept | Icon (Material) | Color |
|---------|----------------|-------|
| Score / Gold | `monetization_on` | `primary` (#f9a215) |
| Health | `favorite` or `battery_charging_full` | state-color |
| XP / Weekly | `bolt` | `rpgAccent` |
| Monthly | `calendar_month` | `rpgText` |
| Warning | `warning_amber` | `rpgMuted` |
| Sweet spot | `check_circle` | `rpgSurface` |
| Overcharge | `electrical_services` | `rpgMuted` |

---

## Component Patterns

### Vitals Header
Persistent anchor at top of PetHomeScreen. Thick bottom border separates it from content.

```
┌──────────────────────────────────────────────────┐  ← 4px border
│  🪙 Today: 25/100   ⚡ Week: 42/100   📅 Mo: 38/100 │  ← rpgText, Space Grotesk 14sp
└──────────────────────────────────────────────────┘
  4px solid bottom border
```

### Score Card (Neo-Brutalist)
```
┌─────────────────────────────────────┐  ← 4px border, 4px hard shadow
│  PET HEALTH                         │  ← label 10sp UPPERCASE tracking-wide
│  34                                 │  ← 36sp bold, state color
│  ████████░░░░░░░░░░░░  34%          │  ← progress bar, state color fill
└─────────────────────────────────────┘
  shadow: 4px 4px 0px #3D405B
```

### Quest Card (Session Card in Day Detail)
```
┌─────────────────────────────────────────────────┐  ← 4px border
│  SESSION 1  •  19:00 – 13:08  (86m)  🔴 FAIL   │  ← label row
│  01% → 99%  (+98%)  Score: 5/100                │  ← body 16sp
│  ⚠ CRITICAL LOW START (1%)                      │  ← coral warning
│  ⚠ CHARGED TO NEAR-FULL (99%)                   │
└─────────────────────────────────────────────────┘
  shadow: 4px 4px 0px #3D405B
  left border accent: 4px solid CRITICAL color for bad sessions
  left border accent: 4px solid rpgSurface for sweet spot sessions
```

### Timeline Day Chip
```
┌──────────────┐
│  🟥  SICK    │  ← 2px border, state color background tint
│  Oct 23      │  ← 10sp label
│  5/100       │  ← score, state color
└──────────────┘
  border: 2px solid rpgText
  shadow: 2px 2px 0px rpgText
  selected: border widens to 4px, shadow becomes 4px
```

### Tip Box (Recommendation Card)
```
┌────────────────────────────────────────────────┐
│  ⚔ BATTLE LOG                                  │  ← RPG framing for tips
│  Your pet took heavy damage from charging to   │  ← body 14sp rpgText
│  100% overnight. Unplug at 80% to recover.     │
└────────────────────────────────────────────────┘
  background: rpgBg (#F4F1DE)
  left border: 4px solid rpgMuted (bad tip) / rpgSurface (good tip)
  shadow: 2px 2px 0px rpgText
```

### Floating Navigation Belt
Centered bottom nav with 3 actions: Pet (home), History (timeline), and Users.

```
       ┌────────────────────────────────────┐
       │  🐾 PET  │  📜 HISTORY  │  👥 USERS │
       └────────────────────────────────────┘
         4px border, hard shadow, rpgBg fill
         active tab: primary (#f9a215) underline 4px
         inactive: rpgText at 50% opacity
```

```dart
// Implementation: BottomNavigationBar or custom Row
Container(
  decoration: BoxDecoration(
    color: Color(0xFFF4F1DE),
    border: Border.all(color: Color(0xFF3D405B), width: 4),
    boxShadow: [BoxShadow(color: Color(0xFF3D405B), offset: Offset(4, 4), blurRadius: 0)],
    borderRadius: BorderRadius.circular(16),
  ),
)
```

---

## Screen Layouts

### Pet Home Screen
```
┌─────────────────────────────────────────────────┐
│  [VITALS HEADER: Today | Week | Month scores]   │ ← border bottom
│                                                 │
│          [Lottie Pet Animation]                 │ ← 55% screen height
│                                                 │
│    SICK                                         │ ← 48sp, rpgMuted, UPPERCASE
│    Health: 34/100                               │ ← score card
│    ████████░░░░░░  RECOVERING...                │ ← progress bar
│                                                 │
│  ┌────────────────────────────────────────────┐ │
│  │  ⚔ BATTLE LOG: 3 sessions this week        │ │ ← tip box
│  │  Charged to 100% twice. Unplug at 80%.     │ │
│  └────────────────────────────────────────────┘ │
│                                                 │
│       [FLOATING BELT: Pet | History | Users]    │ ← bottom nav
└─────────────────────────────────────────────────┘
Background: rpgBg (#F4F1DE)
```

### Timeline Screen
```
┌─────────────────────────────────────────────────┐
│  < User 7 — October 2025                      > │ ← header with month nav
│  ─────────────────────────────────────────────  │ ← 4px border
│                                                 │
│  [Oct 23] [Oct 24] [Oct 25] [Oct 26] [Oct 27]  │ ← horizontal scroll chips
│  CRITICAL  TIRED   SICK    SICK    TIRED        │
│  5/100     50/100  47/100  28/100  40/100       │
│                                                 │
│  ── Oct 23 ──────────────────────────────────── │ ← selected day detail
│  [Quest Card: Session 1]                        │
│  [Quest Card: Session 2]                        │
│  [Quest Card: Session 3]                        │
│  Daily Score: 10/100  ← rpgMuted, big bold      │
│                                                 │
│       [FLOATING BELT]                           │
└─────────────────────────────────────────────────┘
```

### User Select Screen
```
┌─────────────────────────────────────────────────┐
│  CHOOSE YOUR HERO                               │ ← 24sp UPPERCASE heading
│  ─────────────────────────────────────────────  │
│                                                 │
│  ┌───────────────────────┐                      │
│  │  USER 1               │  ← quest card style  │
│  │  🐾 RECOVERING        │  ← pet state          │
│  │  Oct 2025 – Nov 2025  │  ← date range        │
│  │  Health: 68/100       │                      │
│  └───────────────────────┘                      │
│  ┌───────────────────────┐                      │
│  │  USER 7               │                      │
│  │  💀 CRITICAL          │  ← CRITICAL state    │
│  │  Oct 2025             │                      │
│  │  Health: 29/100       │                      │
│  └───────────────────────┘                      │
│  ... (users 2, 5, 8)                            │
└─────────────────────────────────────────────────┘
Background: rpgBg, cards use rpgText border + hard shadow
```

---

## Animation Guidelines

- **Pet entrance:** fade in + scale from 0.8 → 1.0, 300ms ease-out
- **State change:** crossfade between Lottie files, 400ms
- **Tactile button press:** translate +4px, shadow disappears, 80ms; release: reverse, 120ms
- **Score count-up:** animate from 0 to final value on screen enter, 600ms
- **Pulse (active state):** subtle scale 1.0 → 1.02 → 1.0, 2s loop, only on the pet Lottie widget
- **Timeline chip selection:** border thickens 2px → 4px, 100ms

---

## Flutter Setup

### pubspec.yaml additions required

```yaml
dependencies:
  google_fonts: ^6.2.1   # Space Grotesk
  lottie: ^3.1.0
  intl: ^0.19.0
```

### app_theme.dart structure

```dart
class AppTheme {
  static const rpgBg      = Color(0xFFF4F1DE);
  static const rpgText    = Color(0xFF3D405B);
  static const rpgSurface = Color(0xFF81B29A);
  static const rpgAccent  = Color(0xFF2A9D8F);
  static const rpgMuted   = Color(0xFFE07A5F);
  static const primary    = Color(0xFFf9a215);
  static const critical   = Color(0xFFC1440E);

  static Color petStateColor(PetState state) => switch (state) {
    PetState.thriving => rpgSurface,
    PetState.happy    => rpgAccent,
    PetState.tired    => primary,
    PetState.sick     => rpgMuted,
    PetState.critical => critical,
  };

  static BoxDecoration cardDecoration({Color? color}) => BoxDecoration(
    color: color ?? rpgBg,
    border: Border.all(color: rpgText, width: 4),
    borderRadius: BorderRadius.circular(16),
    boxShadow: [BoxShadow(color: rpgText, offset: Offset(4, 4), blurRadius: 0)],
  );

  static BoxDecoration buttonDecoration({bool pressed = false}) => BoxDecoration(
    color: primary,
    border: Border.all(color: rpgText, width: 4),
    borderRadius: BorderRadius.circular(8),
    boxShadow: pressed ? [] : [BoxShadow(color: rpgText, offset: Offset(4, 4), blurRadius: 0)],
  );
}
```

---

## Accessibility (minimum bar)

- Color is never the only indicator — state label text (SICK / THRIVING) always visible
- Score numbers always shown alongside color bars
- Pet state label is `Semantics`-wrapped: `semanticsLabel: "Pet state: SICK"`
- Minimum tap target: 44×44px for all interactive elements
