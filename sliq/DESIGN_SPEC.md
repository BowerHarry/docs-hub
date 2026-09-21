# Sliq Design Specification

**Version 1.3 — last checked against the code 2026-09-05**
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

- **One hero per screen.** Menu → logo over the pipe network. Gameplay → the board. Splash → the
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

### Plumbing

Sampled from `Assets/Textures/pipes.png` rather than invented, so pipe that is
**drawn** (the settings manifold, §5.10) and pipe that is **painted** (the artwork
under the board) read as the same plant. Full strength — `PipeBackground` mixes its
own toward the background because it is scenery *behind* the menu's furniture, and
that is the exception, not the rule.

| Token | Hex | Usage |
|---|---|---|
| `pipeBore` | `#A9D8E8` | The inside of a pipe |
| `pipeShade` | `#33505A` | The hairline where the bore meets its outline |
| `pipeLine` | `#07121A` | The cartoon line on plumbing — near-black, as the artwork draws it |
| `pipeCollar` | `#345966` | Collar bands and junction blocks |
| `pipeFlange` | `#EFCF9E` | The tan flare on every open end |

The **proportions** are sampled too, and matter as much as the colours: the outline is
0.16 of the bore each side, a collar stands 1.12 of the pipe's outer width across and
0.38 along it, a flange flares to 1.34 of the outer width over half of it in depth, and
a belt is 0.58 of a pipe. They live in `Plumbing.Ratio`, not at call sites.

### Contrast requirements

Measured, not estimated. The figures below are computed WCAG ratios against the
hex values in this document.

- Text on `teal700`: `cream` only, at **4.34:1** — enough for `display`, `title`,
  `body` and `button`, and marginal for `caption`, so `caption` on teal is never
  set below full-strength cream. (Version 1.1 claimed 7.2:1 here. It was wrong,
  and the error licensed a lot of faint text: `cream` at 50% measures 2.6:1,
  which is what every section header in Settings used to be.) Never `inkSoft` on teal.
- Text on `cream`: `ink` (10.9:1 ✓) or `inkSoft` (7.4:1 ✓).
- Text on a value accent: **`ink` only** (`Theme.Color.onAccent`). Ink on `leaf`
  is 6.2:1 and on `sky` 3.1:1. `cream` on `leaf` is 1.9:1 and white on `leaf`
  2.3:1 — both were shipping, on the win sheet's headline and its primary
  button respectively.
- Value accents are **never** text colors on teal; they appear as fills, strokes,
  chips and spines. This rule was in version 1.1 and was broken on five screens:
  `FREE PLAY` in `sky` on `teal700` measures **1.15:1**, which is very nearly the
  background. Where a screen belongs to a bundle, the accent arrives as a rule
  under the title (`TopBar(accent:)`) or as a spine down a card.
- White is not in this palette. `cream` is the light colour; `ink` is the dark one.

---

## 3. Typography

Single family: **Avenir Next** (system-installed, already partially in use).

| Style | Font | Size | Usage |
|---|---|---|---|
| `display` | AvenirNext-Heavy | 40 | Screen titles (SELECT LEVEL, SETTINGS) |
| `title` | AvenirNext-Bold | 28 | Card titles (EASY), overlay titles |
| `body` | AvenirNext-DemiBold | 18 | Descriptions, control labels |
| `caption` | AvenirNext-Medium | 14 | Meta text (0 / 36 collected, legend rows) |
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

### 5.0 Depth

One shadow style, everywhere: a solid offset copy in `teal900` (or black on a
cream panel), 4pt down, **no blur** — a blurred shadow belongs to a different
drawing. `Theme.addShadow(under:on:)` is the only way to make one. Filled
buttons and cards get it; text, chips and outlined buttons don't. Pressing a
button or card sinks it onto its shadow and releasing lifts it back.

### 5.1 Primary button ("PillButton/primary")
- Pill shape, `cream` fill, 2.5 pt `ink` stroke, standard shadow.
- Label: `button` style, `ink`.
- Optional leading glyph (play triangle, bear head) drawn in `ink`.
- Size: height 64, min width 220, content padding 32 h.
- Press: scale to 0.96 over 90 ms + shadow collapse; release springs back (see Motion).
- **Nothing idles.** The menu's PLAY used to breathe on a 1.6 s loop; it was removed. It
  never checked Reduce Motion, and once the menu had a moving background behind it a
  second ambient loop in front was one too many. There is no idle animation on any
  button on any screen.

### 5.2 Secondary button ("PillButton/secondary")
- Pill, transparent fill, 2.5 pt `cream` stroke at 70%, label `cream`.
- Height 56. Same press behavior. No shadow.

### 5.2b Menu buttons ("PipeButton") — menu only

The menu's PLAY and TUTORIAL are not pills. With the pipe network behind them
(6.1) a cream pill read as a slab dropped on top of the drawing, so they are
lengths of the plant's own pipework instead: a bore between two bracket
collars, label stencilled along it, and **a run of the network passing straight
through** — in at one bracket, out at the other. Every other screen keeps
PillButton.

- 200 × 46 primary, 200 × 40 secondary, corner radius 8. Against the pill's
  64 × 260 this is the lighter button it replaced it to be: a rectangle covers
  more of its bounds than a pill does, so matching the pill's height would have
  made a heavier control, not a calmer one.
- **No end brackets.** They stood for the flanges a real pipe section joins at,
  and they were the darkest thing on the screen after the outlines. The run now
  meets the plate's bare end, which is what a pipe entering a vessel looks like.
- Primary: `cream` face, `ink` stroke and `heavy` label, standard shadow.
  Secondary: `teal900` face, `cream` stroke at 70% and `cream` label, no shadow.
  Both are **opaque** — the secondary cannot be a transparent pill here, because
  the run behind it would show through its own label.
- A button shaped like the scenery risks not reading as pressable. Three things
  separate it: it is cream, not the network's blue; it carries a hard shadow the
  network's runs do not; and it sinks onto that shadow when pressed. Interaction
  is PillButton's contract unchanged.

### 5.2c Bundle signs ("BundlePlate") — bundle select only

The bundle screen (6.2) is the landing screen with four buttons instead of two,
so its buttons are PipeButton in the bundle's paint: an enamel sign bolted onto
a run of the network, name stencilled across it, star count at the far end.

- 224 × 54, corner radius 12 — a fixed size, like PipeButton's, not stretched to
  the screen. A plate wide enough to reach the side channels hides the run it is
  bolted to, which is the whole picture the screen is making.
- **The face is the bundle's accent.** This is the one screen where all four
  appear at once, so the colour goes on the plate rather than into a stripe
  beside it. Which settles the label: `ink` on every face and nothing else
  (`Theme.Color.onAccent`, §2) — `cream` on `leaf` is 1.9:1.
- Four `ink` bolts through the corners, at 55%: the fixing that holds the sign
  on. Same hardware the title signs hang on (6.1, §4).
- It carries a name and a count. Nothing else — no description line, no tile
  portrait, no level total. Those were a card's content, and this is a sign.
- Locked: the accent taken 62% to its own grey and then 42% to `teal700`, and
  **the price in place of the count** — "72" then a head, the other way round
  from the count's head then "12 / 36". A head in front labels what is being
  counted; a head behind is a unit, and a price wants a unit. Never alpha: the
  network runs behind the plate, and a translucent one shows a pipe and two
  collars through the word.

### 5.2d Level signs ("LevelSign") — level select only

The same sign, smaller, for one level of one bundle (6.3): 84 × 60, radius 12,
`ink` marks, four corner bolts, hard shadow.

- **The face is `pipeBore`**, the light blue the plant's own pipework is drawn
  in and the same fill the header pipe has, so a sign reads as a plate cut from
  the stock it is bolted to. Not the bundle's accent: a bundle sign is coloured
  because the bundle screen is the one place all four appear together, and
  twelve signs on one bundle's screen have nothing to tell apart. Twelve plates
  of one saturated accent was the loudest thing in the app. The bundle still
  owns the screen — its colour is on the header plate.

- The number at 24 heavy across the top, the three bear heads under it at 15.
  An unearned head is drawn in ink, which on an accent face reads as a blot
  rather than an empty socket, so it is taken back to 35%.
- The category disc (6.3) **replaces the top-right bolt** rather than sitting
  next to it. A sign this size has one corner spare, not two things to put in
  it.
- Locked: the same grey as a locked bundle sign, and a padlock where the heads
  would be. Three empty sockets would say it twice, and would read as a level
  played badly rather than one not yet reached.
- Finished (three heads): **nothing**. Every sign wears the same `ink` line.
  It carried an outline of its own for a while — `gold`, then `cream` on the
  accent faces this started as, then a heavy `pipeCollar` band — and every
  version was a second way of saying what three filled heads out of three
  already says, on a sign small enough that the heads are most of it.

### 5.2e Record plates ("RecordPlate") — best scores only

The best-scores screen's plate (6.9): a mode's name, the best score ever set on
it, and one line underneath saying when — or, on RANKED, where in the world it
stands.

- 300 × 54 is not enough for that, so it is **300 × 70**. Everything else is
  SignPlate's hardware: the accent is the face, `ink` is the only mark on it,
  four bolts through the corners, one hard shadow, and it sinks onto that
  shadow when pressed.
- RANKED is `gold` and is still the only sign in the app that is. The **rank
  goes in the line the other two use for a date** — "#1 · 12 SEP", the number
  in `ink` at 13 heavy and the rest at 11 in ink 70%. Not beside the score:
  a rank grows to "#12,904" and the three scores have to stay a column.
- A mode never played prints an **em dash**, not a zero, on a plate in the same
  paint a locked bundle wears. A zero is a score somebody got.

### 5.11 The keep bolt ("KeepBolt")

The fixing that decides whether a played game survives (6.10). Filled `gold`
with a scored head: kept, bolted to the wall, never swept off. An empty socket
at `ink` 30%: one of the fifty, and on its way off the shelf. The same idiom a
level sign uses for a star not yet earned — the hardware is there either way,
and whether it is filled is the state.

It is 14 points across inside a 44-point target, and it is the only control in
the app that shares a row with another one.

### 5.12 Scroll gauge ("ScrollGauge")

How far down a scrolling list you are, as the board's own pressure gauge (§6.6)
mounted vertically in the margin: a capped tube, `teal900` bore, `pipeLine`
outline, `pipeBore` charge and a cap at each end. The charge's **length** is how
much of the list the window shows and its **position** is how far down you are.

The play history is the first screen in the app that scrolls, and a grey capsule
belongs to the operating system rather than to this drawing. Nothing else may
scroll without a reason as good as fifty games.

### 5.3 Icon button
- 44×44 tap target, 36 pt visible circle, `cream` at 12% fill, glyph in `cream`.
- Glyphs are code-drawn vector shapes (existing cog/podium style is fine) — consistent
  2.5 pt stroke weight, rounded caps.

### 5.3b Disc button ("DiscButton")
The disc bolted onto the plant: a solid `tealSurface` face, an `ink` ring, four
`creamDim` rivets at 45° and one mark. It is the header's back and pause (§4, 32 pt)
and the three controls on a shutter (5.8: 68 pt centre, 46 pt sides). Never smaller
than a 44 pt target. No handwheel spokes — a wheel says "turn me", and this is a press.
- Seven marks, one 3.2 pt pen scaled with the disc: back, pause, play, next (play up
  against a bar), restart, home, lock.
- A **locked** disc is duller hardware (`#2A5B69`, mark `#9DB5B4`), never a translucent
  one — a shutter's seams run behind it. It refuses a tap with the valve's flicker.

### 5.4 Card
- Rounded 20, `tealSurface` fill, 2 pt `cream` stroke at **45%**, standard shadow.
  `tealSurface` on `teal700` is a 1.25:1 step, so the stroke and the shadow are
  what make a card an object; at 20% and with no shadow they were ghosts.
- Locked state: 40% content opacity + padlock glyph top-right. No random
  rotation jitter — cards sit straight. (The bundle and level screens no longer
  use Card; their signs dim by mixing paint, not by alpha — see 5.2c.)
- Selected/completed accents use the bundle's value color (see 6.3).

### 5.5 Level tile — superseded by 5.2d

The square card a level used to be. Kept here for the one rule that outlived
it: the category chip is a 20 pt circle with a value-colour fill and an **ink**
glyph, one per level (its primary category), and the five categories map onto
the five palette colours — they may not introduce their own.

### 5.6 Progress bar
- Track: full pill, height 12, `teal900` fill.
- Fill: value-color of the current bundle, rounded.
- Star markers: bear-head ticks **sitting on the bar** at 33% / 65% / 100%,
  `gold` once passed. They come from `Star`, like every star in the game. On the
  dark track the unearned head is tinted `cream` (`Star.socket(onDark:)`), since
  the ink-drawn asset disappears into `teal900`.
- This bar is the canonical score-vs-target display in the HUD.

### 5.7 Toggle / Stepper / Cycle chip
All three are drawn from the same materials as everything else here. Nothing on
this screen is Apple's control in Sliq's colours.
- Toggle: pill track with the 2.5 pt `ink` outline, `Leaf` when on, `teal900`
  when off, `cream` knob with an ink line scored across it. The knob rolls as it
  travels, so the line shows the movement.
- Stepper: `−` value `+` — **filled** icon buttons (5.3) flanking a `body` value,
  value min-width 64 centered so buttons don't shift.
- Cycle chip: `cream` fill, 2 pt `ink` stroke, `ink` label, and a chevron —
  without it nothing said the chip cycled. The new value slides in from the
  right, in the direction the chevron points.

### 5.8 Sheets

Two kinds. Everywhere but the board screen, a confirmation is the **panel**
(`OverlaySheet`); on the board screen every sheet is the **shutter** (`ShutterSheet`).

#### The panel — settings confirmations
- Full-screen dim: `teal900` at 80%.
- Panel: `cream`, radius 20, `ink` stroke, max-width 340, content padding 24.
- Text on panel: `ink`/`inkSoft`. Buttons inside follow 5.1/5.2 (primary gets `Leaf`
  fill + **`ink`** text when it's a celebratory CTA — `cream` on `leaf` is 1.9:1).
- Enter: scale 0.9→1.0 + fade, 250 ms easeOutBack. Exit: 150 ms fade.

#### The shutter — pause, results, the celebration (6.7)

A roller shutter drops over the board. It replaced the panel on this screen because a
panel says *the app has interrupted the game* and a shutter says *the machine has
stopped* — and because through an 80% dim every tile is still readable, which made
pausing free planning time. The shutter hides the board.

All five sheets are this one component with different paint, and these are the rules
that keep it one:

- **One surface.** It covers the whole board *assembly* — ring and rotation gauge
  (6.6) — which is the thing the scene centres, so it lands dead centre. Nothing is laid
  on top of it: no plate, no panel. Everything else, header included, dims to 40%
  (`#123A44` at 60%) *as it falls*, and comes back as it rolls up.
- **Two kinds of mark.** **Paint** is `cream` lettering laid on the slats, so the seams
  are drawn back over it. **Hardware** — discs and bear heads — sits above the seams.
  Nothing on a sheet is anything else.
- **Three colours.** Slat teal (`#1F515D` / `#245E6B`, alternating), `cream`, and the
  `gold` of a head. A bundle's accent appears only as charge in a pipe, the one way the
  HUD already uses it.
- **One face, three sizes.** Heavy caps, tracked: headline 26, caption 13, disc label
  10.5. No mixed case, nothing rotated.
- **It does not repeat the header.** The level's name is on the sign hung above it,
  which stays readable through the dim; a sheet never says it again.
- **Snapped to the slats.** 16 pt pitch. Small text sits inside one slat and is never
  cut; the headline is centred *on* a seam so the joint runs through it, which is what
  paint on a roller shutter does.
- **Three slots**, the same on every sheet (`DiscButton`, 5.3): left, *this level
  again*; centre, 68 pt, *the way on*; right, *leave*. 46 pt either side. All
  `tealSurface` — the primary is marked by size and position, not by a colour.

Rows, top to bottom: headline · reward · one caption · discs · their labels. The discs
stand 9 pt clear of their labels, hard shadow included — any closer and a label reads
as part of its disc rather than as the word under it. It is laid out once in a
378 × 352 space and scaled to the assembly by whichever axis asks for more (0.69 on an
SE, 1.1 on a Pro Max); a disc's 44 pt target is measured on screen, not in that space.

**Motion.** Drops in 280 ms easeIn — it is heavy, it gathers speed — lands 4 pt past the
rail and settles in 100 ms while the slats rattle; `lever` + a medium knock. The paint
is on the slats, so it unrolls with them. Hardware bolts on after landing, centre disc
first. Rolls up in 240 ms easeOut: faster than it fell, because the player asked. A tap
during the entrance lands everything at once, silently. Under Reduce Motion it is a
150 ms fade with no drop and no rattle.

**It is how the game changes level.** NEXT, AGAIN and RESTART rebuild the board *behind*
the shutter and roll it up on the result: no scene cut anywhere in the loop, and nobody
watches the old board vanish. LEVELS and MENU leave it down and crossfade — the last
thing seen of a level is a shut hatch.

### 5.9 Coach mark (tutorial)
- `cream` speech bubble (radius 16, `ink` stroke) with bear-head icon, `body` text in `ink`.
- A pointer wedge under the bubble, aimed at the cell the step is about and
  clamped to stay within the bubble's width.
- A dimmed spotlight (`teal900` at 34%) over everything except that cell, drawn
  as four panels around the hole rather than an inverted mask.

### 5.10 Drawn plumbing (`Plumbing`)

The vocabulary for pipe that is drawn rather than cropped from artwork, at the
values and proportions in §2. Three primitives and two fittings:

- **Run** — one path stroked three times: outline, a hairline of shade, then the
  bore. One path, so a bend is a bend rather than two lengths meeting at a corner.
  Butt caps, because every open end in this drawing is closed by a flange.
- **Collar** — a band standing proud of the run. **A run carries one kind of
  fitting at a time**: a length that ends in a junction has its fitting already,
  and collars belong in clear pipe. Ignoring that is what made the first draft of
  the settings manifold read as clutter — fifteen collars on a screen with seven
  switches.
- **Flange** — the tan flare on an open end. Nothing is left as a cut-off tube.
- **Cargo** — a tile riding the plant, corners masked off. A tile in a pipe has
  no grid to line up with, and a hard corner reads as a chip of something rather
  than as cargo.

Every layer carries an **explicit `zPosition`**. See §10.

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
- Background: `teal700`, with a **pipe network** drawn across the whole screen and
  tiles running through it. It replaced a strip of the pipes illustration along
  the bottom edge and tiles falling from the top: two ambient ideas in opposite
  corners, neither of them the game's own image. Tiles moving through plumbing is
  what the board does, so the menu now shows the same thing the game is about.
- Layout, top → bottom:
  - The **title sign** (below) hanging from a pipe, upper third.
  - PLAY — primary PipeButton (5.2b), with a run of the network through it.
  - TUTORIAL — secondary PipeButton.
  - Settings cog and leaderboard podium, as **handwheels bolted to the pipework**
    (below) rather than an icon bar along the bottom edge.
- **No FPS/node debug overlay. Ever.**

#### The title sign

The wordmark is not loose on the background; it is a dark enamel plate hung off
a pipe on two chains, with rivets at the corners. The right chain is 16 points
longer than the left, so the sign hangs a few degrees out of true — the plant
has been running a while and nobody has been up a ladder.

- **The tilt is derived, not chosen.** It is the angle those two chain lengths
  force (`atan(sag / chain spread)`), and the plate hangs from where the chains
  end rather than being positioned separately. Lengthen a chain and the sign
  tips by exactly as much as it would if it were real; the parts cannot drift
  out of agreement.
- The face is `teal900` because the wordmark art is cream with an ink shadow. On
  a cream plate it needs tinting to a flat silhouette, which throws away the
  shadow the letters are drawn with.
- It is placed by centring the whole assembly — hanger point down to the plate's
  lowest corner — in the gap between the resume banner and PLAY, not at a
  fraction of the screen height. On a 4.7" screen that gap is barely bigger than
  the sign.
- It is the **widest** piece of furniture, so it and not the buttons sets how
  much room the side channels have for vertical pipes.

#### The tool wheels

Settings and leaderboard are handwheels (`ValveButton`) mounted where the
network was already drawing decorative valve gear. There is no bottom bar and
no tool rack: two painted-on wheels and two real buttons somewhere else was one
kind of control too many, so the gear on the pipes became the gear you press.

- Pressing turns one **a third of a turn clockwise** against resistance — a slow
  quarter to break it loose, then the rest easing out, which is the motion and
  the sound (`sfx.lever`) the board's own valve has.
- A scene transition pauses the scene it is leaving, so the screen waits 0.3 s
  before navigating. Without it the wheel freezes on its first frame and is
  never seen to move.
- Every 17–37 s one **slips a few random degrees anticlockwise**. Three spokes
  make the wheel symmetrical every 120°, so a third of a turn lands looking
  identical and any neat sixth looks the same whichever way it went — the
  movement would read and the result never would. Odd degrees do not divide
  into 120, so they accumulate: the wheel drifts off square and stays there,
  and being visibly out of true is what makes every later turn legible.
- The bottom band gives up room for the wheels before its rungs are placed. A
  handwheel stands half again as proud of its pipe as the bore is wide, and
  without that the upper one fouls the TUTORIAL plate on a 4.7" screen.

#### The pipe network

Drawn, not cropped from artwork: it has to bend around furniture that sits at
fixed point sizes on screens from 375 to 440 points wide, and a bitmap can only
be cropped or stretched. The style is measured off the pipes illustration —
light bore, ink outline, dark collars, cream valve gear — so the two read as the
same plant. Colours are *mixed toward* `teal700` rather than drawn at reduced
alpha: the network crosses itself constantly, and translucent strokes let every
crossing show through the run on top of it.

- **Six runs, and no more.** It was ten. A schematic that fills every gap reads
  as clutter, and the junction block at each crossing made it worse: they were
  the darkest thing on screen after the outlines. Blocks are now mixed halfway
  to the background and sized `bore + 5`, so they sit as a shadow band on the
  pipe rather than a hole punched through it. A run carries one kind of fitting
  at a time: a block where runs meet, or a collar out in clear pipe. A collar
  keeps a whole bore clear of any block, or the two read as joints crowding
  each other rather than as pipe with its joins in it.
- **The rule it is built on:** verticals only run in the two side channels the
  furniture leaves free, and horizontals only cross in the bands between pieces
  of it. Every line is derived from a gap between two of them —
  `MenuScene.Furniture` is the one set of numbers both the scene and the network
  lay out from, and `PipeBackgroundTests` checks no pipe ever crosses the
  wordmark or a button on any screen size.
- The resume banner is the exception: it is an opaque card
  with their own shadows, and the network runs behind them the way the pipes
  artwork used to run behind the tools.
- Drawing order is pipe → collars → **tiles** → junction blocks and valve gear, so
  a tile disappears into a fitting and comes out the far side. That is what puts
  it inside the pipe rather than on top of it.
- Five tiles on screen at once, at 78–104 pt/s, entering and leaving off-frame.
  A run takes ten to fifteen seconds to cross, so without a ceiling the menu
  fills for as long as it is left open and ends up reading as a screen of tiles
  with some pipes behind them.
- Under Reduce Motion the plumbing stays and nothing moves: it is scenery, not
  animation. The travelling tiles, the handwheel (one revolution a minute) and
  the gauge needle are all gated together.

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

The landing screen with four buttons instead of two. Same background: the
**pipe network** across the whole screen with tiles running through it, plumbed
to its own plan (`BundlePipes`), and four **bundle signs** (5.2c) bolted onto
four of its runs. It used to be four cards floating on flat teal above a strip
of the pipes illustration — the screen you arrive at from the menu looked like
it belonged to a different game.

- Top bar: the standard hung sign, "SELECT LEVEL", with the back button on its
  pipe (§4). It is the only part of the screen that is not the menu's furniture.
- Four signs: EASY / MEDIUM / HARD / FREE PLAY, accents Leaf / Amber / Cherry /
  Sky. Each carries its name and its bear-head star count and nothing else.
  Free play has no count — it is endless — so that end of its sign is bare.
- Locked bundles print their price where an open one prints its count: "36 🐻"
  or "72 🐻". One currency, one number, on the sign itself — it used to be a
  sentence ("Earn 18 in EASY — 4 so far") set underneath in another colour,
  which is a lot of words for a gate.
- Enter animation: staggered 60 ms slide-up + fade, each sign landing on its
  pipe in turn.

#### The network here is quieter than the menu's

Four plates down the middle of a screen cannot take the density the menu's two
can — the same pipework behind them reads as four signs in a thicket. Every
dimension is one step down from 6.1: a thinner bore (20 pt against 26), one run
per plate and nothing else crossing the screen, no small-bore work in the top
band at all, and collars 8 bores apart instead of 5.4. What is left is the
skeleton — one main round three sides and four rungs with a sign on each.

The main is the menu's read the other way round: in at the top **right**, down
the right channel, along the floor, and a short way up the left one. As on the
menu, neither side runs corner to corner, or the pair draws a border round the
screen.

Where the menu sizes its side channels from its furniture — the title sign is a
fixed width and the pipes take what is left — this screen does the opposite.
Its plates are a fixed size and the channels are fixed at one main and its air,
so the bare run showing at each end of a sign is what varies with the screen.
`BundlePipeTests` checks the rule that makes it hold together on all four
phones: verticals only in the side channels, and a run only crosses a plate end
to end, along the plate's own axis.

### 6.3 Level select

Twelve **level signs** (5.2d) bolted onto a pipe that snakes through all of
them in order: in at level 1, along the row, down at the end, back the other
way, and out along the floor past 12.

It was a 3×4 grid of square cards. A grid says twelve levels exist; it does not
say which one is next, and it certainly does not say that finishing 3 is what
opens 4. The snake says both — the drops at the ends of the rows are the
gates, drawn. The order is the drawing.

- Top bar: back chip + bundle name on a plate in the bundle's accent.
- **No star total.** The bundle screen prints one for every bundle, and the
  twelve signs here are that same count spelled out; a running total set over
  them was the only line on the screen nothing pointed at.
- Rows alternate direction, so level 4 sits under level 3 and 7 under 6. The
  scene's furniture lists the signs **in level order**, not in reading order.
- A **legend** in the band between the last row and the floor run names
  whichever categories appear on the screen. A disc a player meets on level 4
  with no explanation anywhere in the app is decoration, not information. It is
  kept clear of the side channels, because the snake drops through them.
- Tap flows directly into gameplay with the standard transition.

#### The traffic is the progress bar

The snake is composed as two runs that meet **under a sign**: the part that is
unlocked, which carries tiles, and the part that is not, which does not. Tiles
enter at level 1 and travel as far as the furthest level reached, where they
vanish into that sign — it is opaque and four times the bore, so a tile
arriving is inside it before it stops. Both runs share that point and are drawn
identically, so the pipe reads as one unbroken line while the flow along it
does not.

The result is that how far the player has got is legible from across the room
without reading a number: colour and movement stop where they do.

### 6.4 Settings

The plant's control panel: a **main** down the right margin with a **branch** off it to
every switch, and a **dock** at the bottom where the main turns, drops through a flanged
outlet onto a conveyor and runs off the end of it. Throwing a switch on sends one down.

It used to be four toggles, a reset and a version string on bare teal — correctly
structured and completely inert, with nothing on it that belonged to Sliq rather than to
any app. The menu had already found the answer: the game owns a picture of itself, and a
screen that uses it stops looking like chrome. This is the board's own assembly
(`FactoryNode`) re-composed — same belt, same tiles — so the screen is not a
metaphor for the game, it is the game's hardware doing a second job.

- **Three kinds of row, and the pipework says which is which.** A **switch**
  carries the thick branch off the main and sends a tile down it when it goes
  *on*, because it changed how the game behaves — only *on*, so the screen has a
  direction. A **header** carries the thin instrument rule against its name. A
  **link** — Sign In, Replay Tutorial, Privacy Policy, Reset Progress — carries
  no pipe at all and sends nothing: a chevron and the row itself is the whole
  control. Nothing flows down a link.
- **A tile leaves level with the switch that sent it.** The height comes from
  SpriteKit's own coordinate conversion, not from the `y` the row was laid out
  at — rows are spread after layout, so the laid-out value is stale by exactly
  the gap the row moved, and tiles entered the pipe above the switch that threw
  them. Reduce Motion reads the old value before writing the new one, so its own
  last tile still makes the trip before the line goes still.
- **The tile is the row's place in its section**, so its colour follows from its value
  the way it does everywhere else, and the panel teaches the tile palette while it is
  used. It travels the pipe small and grows as it drops out of the outlet — a 17 pt bore
  and a legible tile cannot both be true otherwise, and it is what `launchBeltTile`
  already does on the board.
- **Neither the bear nor the belt is here.** Both were: he ate each tile off the
  end of it, and he was the best thing on the screen and the wrong thing on it —
  a settings panel had acquired a pet, a fullness meter and a chomp. The belt
  went with him, because a conveyor carrying tiles to nobody is a prop. What is
  left is what the plant would actually do: the tile **drops out of the flange
  and goes**, opening to full size as it clears the lip and turning a hundredth
  of a turn or so either way on the way down — enough that no two fall
  identically, not so much that it reads as spinning.
- **Reset Progress is a link like any other**, with a subheading saying what it
  erases. It was a red pill sitting alone across the middle of the page, which is
  a lot of shouting for a row nobody wants to press; the confirmation sheet (5.8)
  is where this gets to look dangerous.
- **Nothing scrolls, and it is two levels.** The root is a short menu of
  sections — name, a line saying what is inside, and a chevron in. Tapping one
  opens it; the back chip closes it, then leaves the screen. One control, two
  meanings, because a second "up one level" affordance is one more thing to
  explain on a screen whose whole job is explaining things.
  A single scrolling list came first and needed a pan recogniser, a clipped band
  and per-row touch reach — a lot of machinery for fifteen rows. A row of tabs
  came next and was worse: four chips on a 375-point screen are 77 points each,
  which is not enough to name a section in.
  **`SettingsLayoutTests` measures the menu and every section on every screen
  size the game supports.** A section that outgrows the space fails the build, at
  which point it gets split — it does not get a scrollbar.
- **The pipework does not animate, and neither does a switch.** A row is its
  plumbing plus a body, and only the body arrives — name, caption, chevron. A
  branch sliding in from the right while the main it joins stands still reads as
  a drawing coming apart; and a switch is a **valve on that branch**, so hardware
  arriving after the pipe it is bolted to has the same problem. The plant is laid
  down first and the words are what move.
- **Rows are packed, then spread.** They are laid out at a pitch the 4.7" screen
  can take, then opened out into whatever room the screen actually has, capped so
  the result reads as composed rather than as reaching for the floor. What is
  left below the last row is the dock's.
- An open section shows its name with the instrument line running out of it into
  the main, and starts its rows a little lower than the menu does, because the
  menu has no header to clear. Taking that room off the shared top would have
  cost the menu twelve points it does not have on a 4.7" screen.
- **A menu row is a header, not a takeoff.** It carries the same thin instrument
  line against its name, no branch off the main, and it sends no tile: opening a
  section is navigation, and the plant runs when a setting *changes*, not when
  someone goes looking for one.
- **No button on a menu row or a link — the row is the target.** A filled chevron
  button said "press this small circle" when the whole row is pressable, and put
  a control on a line that controls nothing. What is left is a bare cream
  chevron: a disclosure mark, not a control, and smaller than the back chip's —
  that one is a control with a 44 pt target, this is punctuation after a word.
  It sits **after the name**, because out at the right margin it landed on top of
  the instrument line and a chevron straddling a pipe reads as a fault, and it is
  set on the name's *optical* centre rather than its line box, which includes
  descender space the word does not use and left the mark sitting visibly low.
  Pressing anywhere on the row dims it.
- **The menu carries the one-liners; an open section does not.** The section
  header names the page, and repeating its description under it is telling
  someone where they are after they have arrived.
- Rows arrive **from the main** going in and from the left coming back, so the
  direction of travel says which way through the screen you moved.

Sections, top to bottom:

| Section | Rows |
|---|---|
| SOUND & FEEL | Sound Effects · Haptics · Reduce Motion |
| PLAYING | Edge Marks (§9) · Rotation Warning (§6.6) |
| PROGRESS | Game Center · Replay Tutorial · Reset Progress (confirmation overlay 5.8, explicit "erases all stars" copy) |
| ABOUT | Version **and build** · Privacy Policy · Send Feedback · Rate Sliq |
| DATA | Record Play Log · Share Play Log · **in DEBUG builds only**, Unlock Everything |

Grouped by what a player came here to do. AUDIO was one switch on its own; Game
Center sat in a section of its own and now shares PROGRESS, because signing in
and your stars are the same errand.

**Game Center is one row with two states**: signed out it offers the sign-in,
signed in it names the player and opens the game's Game Center page. A line of
status text used to sit above it saying the same thing in worse words — and, when
signed out, saying it twice.

The build number is in ABOUT because it is the first thing a tester is asked for.
Share Play Log is no longer behind `#if DEBUG`: the log is written in release builds by
design, and a tester who is asked for theirs has to be able to find it. Record Play Log
is the opt-out — it never leaves the device, which is a reason to be relaxed about it,
not silent.

Still open, and deliberately absent rather than dead: **Bold Numerals** needs new tile
art (the numerals are baked into `N-tile.png`, which `scripts/make-textures.py` does not
generate), and **Show Matches** needs an engine query plus the ranked-run guard.

### 6.5 Free play

**Free play is endless.** No target score, no win, no stars: the board fills or
it does not, and the number you finish with is the whole result. A score attack
with a finish line is two games in one, and the one nobody was playing was the
second. `LevelDifficulty.noTarget` is how that is spelled to the engine (6.6).

#### The board — four ways to play

The same wall of pipes the bundle screen uses (`SignBoard`) with the same signs
bolted to it (5.2c): **RANKED / CLASSIC / CHAOS / CUSTOM**.

- The first three carry a whole config and **start a run on the tap**. CLASSIC
  and CHAOS wear PLAY's paint — `cream` face, `ink` stencil — because that is
  what they do.
- **RANKED is `gold`**, and it is the only sign in the app that is. Three cream
  plates in a column say the three are interchangeable, and they are not: one of
  them is the leaderboard's, played on a fixed board. Gold is already what this
  game paints a thing that counts — a bear head, a bundle finished — and `ink`
  on it is 6.9:1.
- CUSTOM wears TUTORIAL's — `teal900` face, `cream` label and bolts — because it
  opens a screen rather than starting a game. A player who has met the landing
  page already knows which of these is which. The label colour has to follow the
  face: `ink` on the dark plate is a word you cannot read.
- Only RANKED reaches Game Center, and it plays Hard 5 exactly — a leaderboard
  is worth nothing unless every score on it came off the same board.

It was a settings screen you had to get through before you could play: four
preset chips, seven knobs and a START. Three of those presets never needed the
knobs, which is the whole point of a preset.

#### CUSTOM — the knobs

The one piece of pipework on it is the **rule after each group header** —
instrument tubing at a third of a main's bore, running out to the margin. It
had a whole manifold for a while: a main down the right margin, a takeoff to
every knob, and a tile sent down it whenever one was turned. That is the
settings screen's picture (6.4), and settings earns it because a switch there
is a thing that happens; a knob here is a thing you are still deciding. Six of
them behind six takeoffs read as a plant to operate rather than a form to fill
in, which is the wrong invitation on the screen you have to get through to play.

- Steppers and cycle chips (5.7) matching the *real* engine knobs only, grouped
  under PACE and BOARD. Anything the engine ignores must not appear here —
  which is now also true of the target score.
- **Every knob carries a one-line consequence** underneath in `caption`-weight
  cream. "Pressure builds: SLOW" is not something a player can reason about, and
  a setting they can't reason about is one they leave alone.
- START is **PLAY** — the same PipeButton, cream face and ink stencil. Free
  play's three fixed modes are already PLAY-coloured on the board before this
  one; this is the fourth of them, at the end of the six knobs you just set. No
  run through it: there is no plant on this screen for one to come from.
- Opens on CLASSIC's values: the middle of the three fixed modes, and the one a
  player is most likely to be adjusting away from rather than toward.
- **The row pitch comes out of the gap** between the header and START, so six
  knobs and their notes fit on a 4.7" screen. The fixed 56 pt pitch this
  replaced ran the last two rows off the bottom of that display.

### 6.6 Gameplay (HUD)

On an **endless** run (`LevelDifficulty.isEndless`, i.e. free play) the score
module loses its second half: no "/ target", no progress bar under it, and the
belt bear stops filling — there is nothing to fill toward. The score centres and
goes up a size, because it is the only number the run will produce. The result
sheet drops its star row for the same reason and prints the score instead of a
percentage of a target that does not exist.

Background: `teal700` — **the same world as the menus**.

The **board bed is cream** (`empty-board.png`), with ink rules at ~22% and no
outer frame under the ring. It used to be `#F8F8F8` graph paper with grey pencil
lines: the largest object in the game, in a colour that appears nowhere else in
it, and carrying a doubled hairline at row 1 column 8 that shipped in every
App Store screenshot. Tiles carry a hard `ink` shadow at 20% so they read as
pieces sitting on a tray — the pale faces and the bed are close in value by
design, and depth is what separates them, not hue.

The board, its border ring and the rotation gauge are **one assembly**, and it is
the assembly that gets centred on screen, not the grid.

Top → bottom:
- Top bar: back chip (pause glyph — it opens a pause overlay, not instant exit; overlay
  offers Resume / Restart / Quit via 5.8) + level chip ("EASY · 4") in bundle accent.
- **Score module** (the HUD centerpiece, replaces the three Arial labels):
  - Current score in `hudValue`, right-aligned against target "/ 240" in `body` `cream` 60%.
  - Progress bar (5.6) underneath, full content width, star markers at 33/65/100%.
  - On score: bar fills with 200 ms easeOut; passing a star marker pops the bear-head
    tick (scale 1→1.4→1, `gold`).
- Board block: border edges, grid, tiles.
- **Rotation countdown**: a 13 pt **pressure gauge** bolted to the ring's right
  face — a capped pipe with `pipeBlue` collars and the ink outline, board height,
  draining as the next auto-rotation approaches. Bundle accent, switching to
  `cherry` in the final fifth. While the walls are actually turning it holds full
  and dims to 30%, which is also the input-locked state. A rotation flashes the
  gauge as it refills, so the countdown the valve just spent visibly comes back.
  At 7 pt and unoutlined it was the exact weight of a system scroll indicator.
  **Rotation Warning** (§6.4, on by default) says the final fifth out loud as well:
  one tick, one light knock and a single pulse of the gauge, armed once per
  rotation window. The colour change is for eyes that are on the gauge; a player's
  eyes are supposed to be on the board.
- Illustration block (`FactoryNode`): pipes, the valve wheel, the funnel, the conveyor and
  the bear (who grows with progress). **The valve on the right rotates the board early** —
  a real trade, since it also costs a spawn wave and restarts the countdown. It replaced
  the old lever, which is gone.
- **No floating "+N".** The score module, the progress bar and the bear filling
  up are the feedback for a score; a number flying off the board on top of them
  was one channel too many.
- The border edge a scoring tile pushes through **flexes** and springs back, so a
  score reads as something that happened to the board rather than only to the
  tile that vanished. Scaled off the ring's thickness.

### 6.7 Pause and game over

Every one is the shutter (5.8). What differs is the paint and which marks the discs wear:

| Sheet | Headline | Reward | Caption | Left · **centre** · right |
| --- | --- | --- | --- | --- |
| Pause | PAUSED | — | — | RESTART · **RESUME** · LEVELS |
| Win | LEVEL COMPLETE | three heads | "24 / 36 IN EASY" | AGAIN · **NEXT LEVEL** · LEVELS |
| Finale | BUNDLE COMPLETE | three heads | "MEDIUM BUNDLE UNLOCKED" | AGAIN · **MEDIUM · 1** · LEVELS |
| Loss | BOARD FULL | heads banked | "61% OF TARGET" | MENU · **RETRY** · LEVELS |

- The **heads** sit bare on the slats at 54 pt. They are there, dim, as the shutter comes
  down, and the earned ones **light** 200 ms apart with the gauge's own pop — they do
  not pop in from nothing. A win is always three (the third head *is* the win
  condition), so it is only on BOARD FULL that they carry information. Never "GAME
  OVER": losses still award heads.
- On a win the score is just the target met, so the caption is where that leaves you.
  On a loss it is only how close you came: the heads above it already say what was
  banked, and the line used to say it a second time.
- **Nothing moves behind a result.** The engine stops itself when a level ends, but a
  scored tile takes up to six seconds to reach the bear, so the plant under the sheet
  went on visibly scoring. What is still in the pipes fades in the quarter-second the
  background takes to dim (`FactoryNode.wrapUp`), and the whole gameplay layer freezes
  once it has. Pause freezes it too, mid-flight, because there it all resumes.
- **A finale says what it opened.** "MEDIUM BUNDLE UNLOCKED", and the centre disc takes you to
  its first level; or "3 MORE TO OPEN MEDIUM", and the centre disc is a padlock that
  refuses the way the valve does. `Progress` always knew; the sheet used to make the
  player walk back out to the bundle screen to find out. After Hard there is nowhere
  further, and the centre disc is MENU.
- **Loss** lights the **alarm**: a red beacon bolts onto the right-hand end of the header
  pipe, above the dim, and sweeps for as long as the sheet is up (`AlarmBeacon`). On a
  loss *again* is the way on, so RETRY takes the centre and MENU keeps the left slot
  from standing empty.
- Free play has no heads: the score is painted in the reward row instead.
- **Beating Hard 12 for the first time** gets the same sheet, once, with the wordmark
  where the headline goes and a small pipe per bundle charging to what was actually
  collected. The party is *outside* the card: everything dims except **the bear**, relit
  on the belt he has stood at all game and fed a heap of tiles for as long as the sheet
  is up, under tile confetti (§7). Centre disc is MENU. Under Reduce Motion there is no
  party, and the sheet stands on its own.
- The bear does **not** appear on a sheet. The heads are the reaction on it; he is the
  reaction in the plant.

### 6.8 Tutorial
Scripted level using coach marks (5.9): swipe to move → value drops per move → match the
border color to score → the walls rotate on their own and drop fresh tiles in → a matching
bottom edge scores by itself → don't fill the board.
Skippable at every step ("Skip" text link, top-right). Offered automatically on first
launch after splash; always available from the menu.


### 6.9 Best scores

Your best score in each of free play's fixed modes, on the same plant every
other screen is drawn on.

It was LEADERBOARDS: a ranked high score in a Card, three more Cards counting
bear heads per bundle, and a GAME CENTER pill under them, all on flat teal. The
bundle rows counted the same stars the bundle screen already prints, and it was
the one screen in the app that had never been drawn as part of the factory.

- Top bar: the standard hung sign, "BEST SCORES".
- Three **record plates** (5.2e) — RANKED, CLASSIC, CHAOS — and the dark
  `PLAY HISTORY` plate under them, in TUTORIAL's and CUSTOM's paint, because it
  opens a screen.
- **CUSTOM has no plate.** A record is a number you can compare with the one
  below it, and a custom run's rules are whatever the player set, so a best
  custom score is a best at nothing in particular. Custom runs live in the
  history (6.10), where the rules they were played on are one tap away.
- **RANKED is the only plate that leaves the app**: pressing it opens the
  free-play board in Game Center. Which is why the rank is on the plate — the
  plate says where you stand and pressing it shows you who is around you. There
  is no separate GAME CENTER button; RANKED is the door.
- No rank to print — not signed in, nothing posted yet, Apple unreachable —
  and the line says how to get one instead. Those are not worth telling apart
  on a plate.
- Enter animation: the bundle screen's, each plate landing on its pipe in turn.

#### The network here is the level snake straightened out

One main down the **left** channel with a rung off it through every plate, and a
short small-bore drop off the bottom rung into the floor (`BestScorePipes`).

The level screen threads one run through all twelve signs in order and stops the
traffic at the furthest level reached, so progress is legible without reading a
number. Here there is no order to draw and nothing to be part-way through — a
record does not advance — so the line that visits everything becomes a main with
a takeoff to each plate, and the traffic runs the whole length. Same weight as
the bundle screen's, and the same rule, which `BestScorePipeTests` checks on all
four phones: verticals only in the side channel, and a run only ever crosses a
plate end to end along the plate's own axis.

### 6.10 Play history

Every free-play game the player has finished, newest first, in two sections.

- **KEPT** holds the games bolted down. Unlimited, and not part of the count.
- **RECENT** carries `42 / 50` at the end of its rule. When the fifty-first
  game ends, the oldest of them goes.

Nothing has to be said in a sentence: the **keep bolt** (5.11) on each row says
which pile it is in, and pressing one moves it — and the count on the RECENT
rule moves with it, while the rows stay where they are. A list re-flowing under
the finger that just pressed it is a worse answer than an order that waits for
the next visit.

**The shelf is only ever swept when a game is played.** Taking a bolt out puts
that game back among the fifty and can leave fifty-one; the next game finished
is what trims it. A row disappearing because the player took a bolt out of it
would be a second rule they never asked for.

- A row is 64 deep because it carries two targets: the bolt takes its 44 points
  at the left, and the rest of the row opens the rules the game was played on
  (6.11). Every row opens — a chevron on all of them — which is the only place
  in the app that ever says what CHAOS actually is.
- Rows are `cream`, except a **ranked** one, which is `gold`: the same rule the
  free-play board and 6.9 follow, and what makes ranked runs findable in a list
  of fifty.
- Each row reads its mode, then `TODAY · 4:12` / `YESTERDAY · 4:12` /
  `4 SEP · 4:12`, then the score. The two recent days are named because that is
  how a player thinks of them.
- Nothing played yet: one line where the first row would be. There is no empty
  illustration in this app and this is not the screen to invent one on.

#### It scrolls, and both consequences are answered

This is the first screen in the app that scrolls. Settings refused to on
purpose — a section that outgrows its space is split, and one that cannot be is
a build failure — but fifty games cannot be split into pages anyone would thank
you for.

- **The network cannot run behind it.** Scenery that stays still while content
  slides over it reads as the drawing coming apart. What is left is the header
  pipe, the run along the floor, and the instrument rule after each section
  name — which is the CUSTOM screen's own treatment, and this list is full of
  CUSTOM's games.
- **The scroll position is a gauge** (5.12), not a grey capsule.
- The list is clipped to the band between the header and the floor, so a row
  leaving the window goes under the drawing rather than over it.

### 6.11 The rules of a past game

The CUSTOM screen (6.5) with the knobs taken off. Same two groups, same six
names in the same words, same instrument rule running out of each header — but
the steppers are gone and the cycle chips have lost their chevrons, which is
exactly what said they cycled. A player who has set a custom game up reads this
without being told anything.

- Top bar: the mode's name on a plate in free play's `sky`, as CUSTOM already
  wears it.
- **The result** on a Card at the top: score at 44 heavy, the day and the length
  under it, and KEEP — the same bolt (5.11) the list uses, so the two screens
  agree about what kept looks like.
- The six rows are `ReadoutChip`: the cycle chip's cream face, ink outline and
  fixed width, without the chevron — and **four points shallower at 30**. A
  cycle chip is a control and owes a thumb 44 points of target; a readout owes
  nothing, and those four points are what let six rows fit here. **No
  consequence lines.** A knob's consequence is for someone still deciding;
  this is a record.
- **PLAY AGAIN** is START's own object and paint (`PipeButton/primary`), and it
  is the point of the whole feature: a kept custom game becomes a preset to go
  back to. No run through it — there is no plant on this screen for one to come
  from.
- **Nothing here is written down.** Six rows, two headers and a result plate
  share one column, and a 4.7" screen has 200 points less of it than a Pro Max,
  so every number is derived from that column: what gives is the result plate's
  depth (110 where there is room, 72 where there is not, and the score drops
  from 44 to 36 and loses its caption with it) and then the row pitch.
  **`GameRulesLayoutTests` measures it on every screen the game supports** — a
  rules screen whose rows touch fails the build, the way a settings section
  that outgrows its page does.
- **Ranked is the exception it always is.** Every ranked run has to be the same
  board as every other one on the leaderboard, so playing that one again plays
  the ladder's rung rather than these read-back knobs. Five of the six rows are
  the engine's own fields read straight back; the sixth, the tile mix, is named
  by what a player perceives about a mix — whether dead 0-tiles turn up in it,
  and how often.

---

## 7. Motion

| Class | Duration | Easing | Used for |
|---|---|---|---|
| Tap feedback | 90 ms | easeOut | Button/tile presses (scale 0.96) |
| State change | 200–250 ms | easeOut | Bar fills, chip swaps, toggles |
| Enter | 250 ms | easeOutBack (small overshoot) | Overlays, cards, stars |
| Scene transition | 400 ms | crossfade | **All** scene changes (replaces mixed push-up/down) |
| Ambient loop | 1.6–6 s | easeInEaseOut | Bear breathing, tiles through the menu pipes |
| Belt settle | 90 ms | easeOut | The border ring overshooting a turn by 6 pt and coming back |

Two signature motions carry the machinery, and neither is decoration:
- **The rotation** runs the ring out past its stop and settles it back, with a
  light haptic on the knock. A belt driven by a valve has mass; without the
  overshoot the ring reads as colours being swapped.
- **The valve** turns against resistance — a slow quarter to break it loose, then
  the rest easing out. Turning early is a real trade (it costs a spawn wave), and
  a valve that spins freely the moment you touch it costs the player nothing.

- Every button press pairs with a **light haptic** (`UIImpactFeedbackGenerator(style: .light)`);
  scoring = `.medium`; win = success notification haptic.
- **Reduce Motion:** disable ambient loops and confetti; keep functional transitions.
  Scenery that does not move is not an ambient loop — the menu's pipe network and
  the settings manifold stay drawn, and only the tiles running through them and
  the valve gear stop.
- It is read **through `Theme.Motion.isReduced` and nowhere else.** The default is
  *unset* until the player touches the switch, and until then it follows
  `UIAccessibility.isReduceMotionEnabled`: someone who turned Reduce Motion on
  system-wide has already said what they want and should not have to find Sliq's
  own switch to be heard, and someone who then turns Sliq's back on has said
  something more specific. There used to be three loose `UserDefaults` reads, one
  screen honouring them, and confetti that rained regardless.

---

## 8. Sound (hooks; assets in task 11)

Every motion class above maps to a sound slot so audio lands consistently later:
`sfx.tap`, `sfx.swipe`, `sfx.score`, `sfx.rotate`, `sfx.lever`, `sfx.spawn`, `sfx.star`,
`sfx.win`, `sfx.loss`, `music.menu`, `music.game`. Settings toggles gate them (6.4).

---

## 9. Accessibility

- Tap targets ≥ 44 pt (level tiles, the valve wheel, steppers included).
- Contrast per §2. Body text ≥ 4.5:1 always.
- **Color-independence: answered, behind a switch.** Tiles carry numerals. Border
  edges carried their value by colour alone, which made the central rule of the
  game — match the tile to the edge — unplayable without colour vision.
  **Edge Marks** (§6.4, off by default) notches the value into each segment: one
  to four background-coloured bites taken out of the segment's *outer lip*, in a
  compact group near its middle.
  Etching pips *into* the segments was built and reverted first: making dots
  legible needed the ring at ~15 pt with an ink outline, and that ring dominated
  the screen. Cutting **out** of pixels the ring already owns adds no weight at
  all — the ring stays 4 pt. They cluster rather than spread because a group of
  one to four is counted at a glance and a spread has to be scanned.
  `EdgeMarkTests` checks every segment carries exactly its own value in marks and
  that no mark reaches past the segment it is cut from.
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
- **One symbol for a star**, and it comes from `UI/Star.swift`. There is no `★`
  in the codebase. There used to be four stars: a system glyph on four screens, a
  red bear head on two, an outlined head on the progress bar, and gold in this
  document only — with two of them 200 pt apart on the win sheet.
- `empty-board.png` and the two head textures are generated by
  `scripts/make-textures.py` from the palette in §2. Edit the script, not the PNG.
- Every scene and component ends with a `#Preview`, built on the helpers in
  `UI/PreviewSupport.swift`, so the whole interface is inspectable in the Xcode canvas.
- **The view runs with `ignoresSiblingOrder`, so every stacked layer needs an
  explicit `zPosition`.** Nodes sharing one are drawn in whatever order the
  renderer likes — including a child against its own parent. This is not
  theoretical: the border's edge marks were all present, correctly placed and
  invisible, drawn behind the segments they were meant to be cut out of, and
  `Plumbing`'s three stacked strokes had the same exposure. If something is built
  and does not appear, check this before checking anything else.
- **`SKShapeNode` does not rasterise reliably below about 4 points.** An edge mark
  is 3 × 2, and as a shape node it drew nothing at all. Small solids are
  `SKSpriteNode(color:size:)`, which is also the cheaper primitive — a full ring
  is 32 segments and up to 128 marks.
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

`../../docs/ui/screens/` holds the current capture of every screen, and is the
set to re-shoot (`scripts/capture-ui.sh`) after any change to this document.
