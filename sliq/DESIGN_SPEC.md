# Sliq Design Specification

**Version 1.1 — last checked against the code 2026-09-03**
**Art direction: "Toy workshop, drawn simple."**

This is the single source of truth for how Sliq looks, moves, and feels. Every screen and
component change must trace back to a rule in this document. When a rule here conflicts
with existing code, this document wins. When something isn't covered, extend this document
first, then implement.

---

## 1. Direction

Sliq's identity is the **illustrated toy-workshop hardware** — the teddy bear, the numbered
wooden tiles, the pipes, the Dream splash animation. Everything else on screen is **flat,
quiet, geometric chrome** whose job is to frame those hero pieces, never to compete with them.

Rules of thumb:

- **One hero per screen.** Menu → logo + falling tiles. Gameplay → the board. Splash → the
  animation. Game over → the bear + stars. All other elements are flat UI.
- **Stylised simple, not barebones simple.** Flat surfaces still get the treatment: rounded
  corners, the outline stroke, considered spacing, a press animation. "A rectangle with
  Arial text" is never acceptable.
- **The cartoon line.** Illustrated assets share a dark outline. Flat UI components echo it
  with a 2–3 pt stroke in Ink (see palette) on interactive elements.
- **Warmth over slickness.** Cream instead of white, ink instead of black, easing curves
  with a little bounce. It should feel like a wooden toy, not a fintech app.

---

## 2. Palette

Derived from the shipped art assets (sampled, not invented). Never introduce colors outside
this table without adding them here first.

### Core neutrals

| Token | Hex | Usage |
|---|---|---|
| `teal900` | `#1E5560` | Darkest teal — shadows on teal, pressed states of teal surfaces |
| `teal700` | `#2D7486` | **Primary background** (matches Dream video edge exactly) |
| `tealSurface` | `#3A8195` | Raised surface on teal (cards, sheets sitting on bg) |
| `cream` | `#F2E8C9` | Primary light surface, button fills, text on teal |
| `creamDim` | `#E4D9BA` | Pressed cream, secondary light surface |
| `ink` | `#2B2B2B` | Outlines, text on cream, the "cartoon line" |
| `inkSoft` | `#4B4B4B` | Secondary text on light surfaces |

### Value colors (the game's language)

Each tile value owns a hue. Saturated = border edges & accents; Pale = tile faces & fills.

| Value | Accent | Pale | Name |
|---|---|---|---|
| 1 | `#70C030` | `#E0F0E0` | Leaf |
| 2 | `#F09020` | `#F5EEDA` | Amber |
| 3 | `#F05050` | `#F5E4E4` | Cherry |
| 4 | `#0070F0` | `#E0EEF5` | Sky |
| 0 (obstacle) | `#9090A0` | `#D0D0D0` | Stone |

Semantic reuse: **Leaf** doubles as success/positive, **Cherry** as danger/destructive,
**Amber** as warning/attention. Do not add separate semantic colors.

### Support

| Token | Hex | Usage |
|---|---|---|
| `pipeBlue` | `#A0D0E0` | Decorative, matches pipes illustration |
| `bearBrown` | `#907050` | Decorative, matches teddy |
| `gold` | `#F0C030` | Stars, celebration moments only |

### Contrast requirements

- Text on `teal700`: `cream` only (7.2:1 ✓). Never `inkSoft` on teal.
- Text on `cream`: `ink` (10.9:1 ✓) or `inkSoft` (7.4:1 ✓).
- Value accents are **never** text colors on teal; they appear as fills, strokes, chips.

---

## 3. Typography

Single family: **Avenir Next** (system-installed, already partially in use).

| Style | Font | Size | Usage |
|---|---|---|---|
| `display` | AvenirNext-Heavy | 40 | Screen titles (SELECT LEVEL, SETTINGS) |
| `title` | AvenirNext-Bold | 28 | Card titles (EASY), overlay titles |
| `body` | AvenirNext-DemiBold | 18 | Descriptions, control labels |
| `caption` | AvenirNext-Medium | 14 | Meta text (12 LEVELS, 0/18 ★) |
| `hudValue` | AvenirNext-Heavy | 30 | Score numerals in game |
| `button` | AvenirNext-Bold | 22 | Button labels |

Rules:
- **Arial and Courier are banned.** (They're the current HUD/labels — replace on sight.)
- ALL-CAPS for titles and buttons, with +1.5 tracking where the API allows.
- Numerals in the HUD must not cause layout shift — right-align score, fixed-width fields.

---

## 4. Layout & spacing

- **8 pt grid.** All margins/padding are multiples of 8 (4 allowed inside components).
- **Screen margins:** 24 pt horizontal. Content never touches screen edges.
- **Safe areas respected.** Top bar sits below the notch/Dynamic Island; nothing interactive
  within 16 pt of the home indicator. (Current screens hard-code `size.height - 60` — replace
  with safe-area-derived anchors.)
- **Top bar pattern** (every non-menu screen): back chip on the left, screen title centered
  (`display` if the screen has no other header, `title` otherwise), optional action right.
- **Corner radii:** 20 pt cards & sheets · 12 pt small cards/chips · full pill for buttons.
- **Stroke:** interactive elements carry a 2.5 pt `ink` outline at 25% opacity on teal
  surfaces, 100% on cream surfaces.
- **Shadows:** one style only — `ink` at 18% opacity, y-offset 4, blur 8. Cards and primary
  buttons get it; text and chips don't.

---

## 5. Components

### 5.1 Primary button ("PillButton/primary")
- Pill shape, `cream` fill, 2.5 pt `ink` stroke, standard shadow.
- Label: `button` style, `ink`.
- Optional leading glyph (play triangle, bear head) drawn in `ink`.
- Size: height 64, min width 220, content padding 32 h.
- Press: scale to 0.96 over 90 ms + shadow collapse; release springs back (see Motion).
- The **only** pulsing element allowed on a screen (idle pulse 1.00→1.03, 1.6 s loop, menu
  PLAY only).

### 5.2 Secondary button ("PillButton/secondary")
- Pill, transparent fill, 2.5 pt `cream` stroke at 70%, label `cream`.
- Height 56. Same press behavior. No shadow, no pulse.

### 5.3 Icon button
- 44×44 tap target, 36 pt visible circle, `cream` at 12% fill, glyph in `cream`.
- Glyphs are code-drawn vector shapes (existing cog/podium style is fine) — consistent
  2.5 pt stroke weight, rounded caps.

### 5.4 Card
- Rounded 20, `tealSurface` fill, 2 pt `cream` stroke at 20%, standard shadow.
- Locked state: 40% content opacity + padlock glyph top-right + unlock requirement in
  `caption` (e.g. "Earn 18 ★"). No random rotation jitter — cards sit straight.
- Selected/completed accents use the bundle's value color (see 6.3).

### 5.5 Level tile
- Square card, radius 12. Level number in `title`.
- Star row: three 16 pt bear-head stars (existing `filled-head`/`empty-head` assets)
  under the number.
- Category chip: 20 pt circle, value-color fill, white glyph — max ONE chip (the level's
  primary category), top-right.
- Locked: as Card.

### 5.6 Progress bar
- Track: full pill, height 12, `teal900` fill.
- Fill: value-color of the current bundle, rounded.
- Star markers: small bear-head ticks at 33% / 65% / 100% positions, `gold` once passed.
- This bar is the canonical score-vs-target display in the HUD.

### 5.7 Toggle / Stepper
- Toggle: standard pill toggle, `Leaf` when on, `teal900` track when off, `cream` knob.
- Stepper: `−` value `+` — icon buttons (5.3) flanking a `body` value, value min-width 64
  centered so buttons don't shift.

### 5.8 Overlay sheet
- Full-screen dim: `teal900` at 80%.
- Panel: `cream`, radius 20, `ink` stroke, max-width 340, content padding 24.
- Text on panel: `ink`/`inkSoft`. Buttons inside follow 5.1/5.2 (primary gets `Leaf`
  fill + `cream` text when it's a celebratory CTA).
- Enter: scale 0.9→1.0 + fade, 250 ms easeOutBack. Exit: 150 ms fade.

### 5.9 Coach mark (tutorial)
- `cream` speech bubble (radius 16, `ink` stroke) with bear-head icon, `body` text in `ink`.
- *Not built:* the pointer wedge toward the referenced element and the dimmed cutout
  spotlight over the rest of the screen. `TutorialScene` currently shows the bubble alone,
  pinned near the bottom of the screen.

---

## 6. Screens

### 6.0 Splash
- Launch screen (static, storyboard): full-bleed `#2D7486` with frame 1 of the Dream
  animation centered at the same size the animated scene will use. Seamless handoff.
- Splash scene: plays the 28-frame Dream sequence once at 24 fps (1.17 s) as a texture
  animation — **no video playback**. Background `#2D7486`, which matches the art's own
  edges so it reads as full-bleed.
- The frames ship cropped to the band the bear and the wordmark move in
  (`splash-band-NN.jpg`, cut by `scripts/crop-splash-frames.py`); the scene draws that
  band over the same teal, landing exactly where the full square frame did. The rest of
  each source frame was one flat colour, and holding twenty-eight copies of it cost about
  90 MB of the launch footprint. `SplashLayoutTests` guards the alignment.
- Hold the final frame 300 ms, then 400 ms crossfade to Menu.
- Total cold-open budget: under 2.5 s. Tap to skip anyway.

### 6.1 Menu
- Background: `teal700`. Pipes illustration footer at 60% opacity (up from 40% — it's
  hero-adjacent, stop hiding it). Falling-tile ambient animation behind UI (existing,
  keep; pause under Reduce Motion).
- Layout, top → bottom:
  - SLIQ logo (existing asset) centered at ~55% width, upper third.
  - PLAY — primary button with play glyph (the screen's pulse element).
  - TUTORIAL — secondary button, bear-head glyph.
  - Bottom bar: settings cog (left), leaderboard podium (right) as icon buttons, aligned
    to safe area.
- The teddy sits on the pipes footer, bottom-right, breathing (scale 1.00→1.02, 2 s loop).
- **No FPS/node debug overlay. Ever.**

#### Resume banner

If the player was part-way through a game when they last left, the menu opens
with a cream card (5.4 stroke and radius, `ink` on `cream`) pinned under the
notch, above the logo:

- Eyebrow: "PICK UP WHERE YOU LEFT OFF" in `caption`-weight `inkSoft`.
- Title: the level in its bundle accent ("MEDIUM · 7"), or "FREE PLAY".
- Right-aligned detail: "62% of the way there" on a level, "Score 1240" in free
  play, which has no target to measure against.
- A progress bar in the bundle accent for levels only.
- Tapping the card resumes; a `×` in the top-right throws the game away.

It is a one-time offer: resuming, dismissing, starting any other game, finishing
one, or quitting to the levels all clear the saved game.

### 6.2 Bundle select
- Top bar: back chip + "SELECT LEVEL" `display` title.
- Four cards (5.4): EASY / MEDIUM / HARD / FREE PLAY.
  - Bundle accent mapping: Easy=Leaf, Medium=Amber, Hard=Cherry, Free Play=Sky.
  - Card content: title (accent-colored underline bar 4 pt), one-line description `body`,
    meta row `caption`: star progress "14/36 ★" or "12 LEVELS".
  - Locked cards state the requirement: "Earn 18 ★ to unlock".
- Card enter animation: staggered 60 ms slide-up + fade on first present.

### 6.3 Level select
- Top bar: back chip + bundle name in the bundle's accent color.
- Sub-header `caption`: "★ 14 / 36 collected".
- 3×4 grid of level tiles (5.5), 16 pt gutters, straight alignment.
- Tap flows directly into gameplay (existing behavior) with the standard transition.

### 6.4 Settings
Sections on `teal700`, `caption` section headers in `cream` 60%:
- GAME: Sound Effects (toggle), Music (toggle), Haptics (toggle).
- PROGRESS: Reset Progress (secondary button, `Cherry` stroke/text; confirmation overlay
  5.8 with explicit "This erases all stars" copy).
- ABOUT: version `caption`, "Made by Harry" if desired.
- DEBUG MODE toggle exists **only in DEBUG builds**, in its own section at the bottom.

### 6.5 Free play setup
- Top bar + "FREE PLAY" title.
- **Presets row first**: three small cards — CHILL / CLASSIC / CHAOS — that fill the
  steppers below; CUSTOM activates automatically when any stepper is touched.
- Sectioned steppers (5.7) matching the *real* engine knobs only (post-wiring-fix): pace,
  rotation, tile mix, target. Anything the engine ignores must not appear here.
- START — primary button **pinned to the bottom safe area**, content scrolls beneath it
  with a 96 pt bottom inset (fixes the current overlap bug).

### 6.6 Gameplay (HUD)
Background: `teal700` — **the same world as the menus** (replaces `#F5F5F5`). The board's
white grid + pale tiles pop against it; the pipes/bear/conveyor illustrations finally sit
on their native color.

Top → bottom:
- Top bar: back chip (pause glyph — it opens a pause overlay, not instant exit; overlay
  offers Resume / Restart / Quit via 5.8) + level chip ("EASY · 4") in bundle accent.
- **Score module** (the HUD centerpiece, replaces the three Arial labels):
  - Current score in `hudValue`, right-aligned against target "/ 240" in `body` `cream` 60%.
  - Progress bar (5.6) underneath, full content width, star markers at 33/65/100%.
  - On score: bar fills with 200 ms easeOut; passing a star marker pops the bear-head
    tick (scale 1→1.4→1, `gold`).
- Board block: border edges, grid, tiles.
- **Rotation countdown**: a slim 7 pt column in the right margin, board height, draining
  from full to empty as the next auto-rotation approaches. Bundle accent, switching to
  `cherry` in the final fifth. While the walls are actually turning it holds full and
  dims to 30%, which is also the input-locked state.
- Illustration block (`FactoryNode`): pipes, the valve wheel, the funnel, the conveyor and
  the bear (who grows with progress). **The valve on the right rotates the board early** —
  a real trade, since it also costs a spawn wave and restarts the countdown. It replaced
  the old lever, which is gone.
- Floating "+N" score text: `hudValue`, `Leaf`, floats from the scored tile — not from the
  bear (the current version spawns at the bear; tie feedback to the action's location).

### 6.7 Game over
Overlay (5.8) over the frozen board:
- **Win:** "LEVEL COMPLETE!" (or "BUNDLE COMPLETE!" on level 12) in `title`, `Leaf`;
  three bear-head stars popping in sequence 200 ms apart in `gold`; a progression line
  ("★ 24 / 36 in MEDIUM") rather than a score, since on a win the score is just the
  target met; NEXT LEVEL (celebratory) + PLAY AGAIN (secondary).
- **Loss:** "BOARD FULL!" `title` (never "GAME OVER" — losses still award stars); earned
  stars shown the same way; copy acknowledges progress: "2 ★ banked — 78% of target";
  RETRY (celebratory) + LEVELS (secondary).
- The secondary never duplicates the primary: once the primary is already PLAY AGAIN, the
  secondary goes to the levels instead.
- **Beating Hard 12 for the first time** replaces this sheet entirely, once: the bear eats
  a mountain of tiles one by one while tile confetti rains over the screen.
- *Not built:* the bear reacting to a normal win or loss. Never mock the player.

### 6.8 Tutorial
Scripted level using coach marks (5.9): swipe to move → value drops per move → match the
border color to score → the walls rotate on their own and drop fresh tiles in → a matching
bottom edge scores by itself → don't fill the board.
Skippable at every step ("Skip" text link, top-right). Offered automatically on first
launch after splash; always available from the menu.

---

## 7. Motion

| Class | Duration | Easing | Used for |
|---|---|---|---|
| Tap feedback | 90 ms | easeOut | Button/tile presses (scale 0.96) |
| State change | 200–250 ms | easeOut | Bar fills, chip swaps, toggles |
| Enter | 250 ms | easeOutBack (small overshoot) | Overlays, cards, stars |
| Scene transition | 400 ms | crossfade | **All** scene changes (replaces mixed push-up/down) |
| Ambient loop | 1.6–6 s | easeInEaseOut | Menu pulse, bear breathing, falling tiles |

- Every button press pairs with a **light haptic** (`UIImpactFeedbackGenerator(style: .light)`);
  scoring = `.medium`; win = success notification haptic.
- **Reduce Motion:** disable ambient loops and confetti; keep functional transitions.

---

## 8. Sound (hooks; assets in task 11)

Every motion class above maps to a sound slot so audio lands consistently later:
`sfx.tap`, `sfx.swipe`, `sfx.score`, `sfx.rotate`, `sfx.lever`, `sfx.spawn`, `sfx.star`,
`sfx.win`, `sfx.loss`, `music.menu`, `music.game`. Settings toggles gate them (6.4).

---

## 9. Accessibility

- Tap targets ≥ 44 pt (level tiles, the valve wheel, steppers included).
- Contrast per §2. Body text ≥ 4.5:1 always.
- **Color-independence:** tiles carry numerals (good). Border edges are currently color-only
  — add subtle value pips (1–4 dots) etched into edge segments so matching never requires
  color vision. This is a gameplay-critical fix, not a nicety.
- Dynamic Type is out of scope for SpriteKit, but all `caption` text must remain legible at
  actual size on a 4.7" screen — verify at 375 pt width.

---

## 10. Engineering conventions for this spec

- All tokens live in `Utils/Theme.swift` (`Theme.Color`, `Theme.FontName`,
  `Theme.FontSize`, `Theme.Metric`, `Theme.Motion`) — no literal colors, fonts or spacing
  in scene code.
- Components live in `UI/` as reusable node classes (`PillButton`, `Card`, `Toggle`,
  `ProgressBar`, `TopBar`, `OverlaySheet`, `StepperRow`, `CycleChip`, `CategoryBadge`,
  `GameOverlays`). Scenes compose them; they never restyle them. If a component needs to
  look different in some context, that variant belongs on the component.
- Art loads through `Utils/Textures.swift`, never `SKTexture(imageNamed:)` directly — it
  caches, and it keeps the file extension in one place.
- Every scene and component ends with a `#Preview`, built on the helpers in
  `UI/PreviewSupport.swift`, so the whole interface is inspectable in the Xcode canvas.
- No `print()` in shipped code paths; wrap dev logging in `#if DEBUG`.
- `showsFPS`/`showsNodeCount` are `#if DEBUG`-only, default off (`-SLIQ_SHOW_PERF 1`).
- DEBUG launch args: `-SLIQ_SCENE menu|game|bundles|levels|settings|freeplay|tutorial`,
  `-SLIQ_BUNDLE n`, `-SLIQ_LEVEL n`, plus `-SLIQ_CELEBRATE`, `-SLIQ_VALVETEST` and
  `-SLIQ_SPITTEST` for visual QA. Used by screenshot automation and by hand.

---

## Appendix A — pre-redesign baseline

Screenshots of the app before this spec existed are in `../../docs/baseline/` in the
workspace folder: a teal menu with a green PLAY, glass-panel selects with rotation jitter,
a near-empty settings screen, Free Play with an overlapping START, and an off-white
gameplay screen with an Arial HUD. Kept only to show what the spec moved away from.
