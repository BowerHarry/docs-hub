# ClickWheel — UI Review & Redesign Plan

A full evaluation of the current UI with a proposed direction, per-area before/after
recommendations, a future-proofed navigation architecture, and a prioritized roadmap.
Mockups referenced here live in `docs/ui-review/` (SVG, open in any browser).

> Scope: this is a **design document only** — no code has been changed on this branch yet.

---

## 1. Philosophy — what the app already gets right

The app has a genuinely good, coherent visual language that suits an iPod-era power tool:

- **Flat, utilitarian macOS chrome** — light‑grey window (`#c7c7c7`), lighter‑grey sidebar
  (`#e8e8e8`), white content, a single blue accent (`#4d8cc7`).
- **Tight, consistent geometry** — 1px borders, **3pt corners** everywhere (`ChromeCard`),
  no gradients.
- **Dense, content-first lists** — the Albums/Artists/Songs views, the album detail
  tracklist, and the info panel are clean because they are *quiet*: striped rows,
  aligned columns, borders only where they earn their keep, one accent colour for state.

**This is the style anchor.** The problems are not the language — they're (a) a handful of
localised deviations that break it, and (b) two structural issues (navigation and how
Profiles are integrated). Everything below pulls the weak areas *toward* the parts you
already like, rather than inventing a new look.

---

## 2. Design principles (the rubric for every change)

1. **One chrome system, everywhere.** If it isn't the flat white‑card‑on‑grey, 3pt‑corner,
   1px‑border look, it's wrong. Kill every exception.
2. **Content first, zero gimmicks.** No rings, gauges, glassy translucency, or decorative
   device art competing with the data. The device screen should feel like an *inspector*,
   not a dashboard.
3. **Structural clarity, future‑proofed.** Top‑level nav is a proper macOS source list that
   scales to Music / Photos / Videos + Devices + Profiles.
4. **Convention & balance.** Follow platform norms (segmented tabs, search on the right
   expanding rightward, one tidy toolbar) so nothing feels ad‑hoc.

---

## 3. Global fixes (small, high-impact, apply once)

These are cheap and make the whole app feel tighter immediately:

| Fix | Where | Why |
|---|---|---|
| **Unify corner radius to 3pt** — remove the lone **4pt** on the iPod illustration | `SyncStatusView.swift:18,21` | The biggest curve on the page; makes the device art look bulbous vs. the crisp 3pt everywhere else |
| **Remove "glassy" translucent-white overlays** — use solid fields | device name field `SyncStatusView:291`, firmware editor `:590`, sync summary `:853` | `white.opacity(0.85/0.65)` reads as dated 2014-era gloss; conflicts with the flat system |
| **Stop nesting cards inside cards** — a `ChromeCard` should not contain more bordered `ChromeCard`/inset cards | Profiles items `ProfilesRouteView:522,601`; device sync summary `SyncStatusView:851` | Double borders muddy the hierarchy; use a divider or a plain inset row instead |
| **Standardise interaction feedback** — one hover/pressed treatment | mixed opacity vs. background today | Consistency; currently some controls dim, others swap background |

---

## 4. Navigation & the future (Device / Music / Photos / Videos + Profiles)

**This is the most important change** and the one that answers "how do Profiles fit."

### Before (today)
The left sidebar stacks a **device status card** ("Connected ● / hamburger" that opens a
popover list of devices — the "top-left dropdown" you dislike) above three flat nav rows:

```
┌───────────────┐
│ ● Connected  ⏏ ≡ │  ← device dropdown (popover) — the bit you don't like
├───────────────┤
│  LIBRARY        │
│  ▸ Device       │
│  ▸ Music        │
│  ▸ Profiles     │
├───────────────┤
│  v1.0           │
└───────────────┘
```
Problems: the device selector is a fussy popover hidden in a card; "Device" is a nav row
*and* there's a separate device dropdown (two ways to think about devices); the flat
"LIBRARY" group doesn't scale to content types.

### After — a proper macOS source list
Model it on Finder/Music sidebars. **Devices become first-class rows** (killing the
dropdown entirely — they're always visible with inline connection status), content types
get their own group, and **Profiles stays its own screen**, grouped with Devices because a
profile *is* "what to sync to a device."

```
┌────────────────────┐
│  LIBRARY            │
│   ♪  Music          │
│   ▣  Photos   (soon)│
│   ▶  Videos   (soon)│
│                     │
│  DEVICES            │
│   ●  iPod Classic   │  ← connected: solid status dot + accent when selected
│   ○  Harry · Nano 2G│  ← disconnected: hollow dot, dimmed
│   ⧉  Sync Profiles  │  ← its own screen, logically with Devices
└────────────────────┘
```

Why this works:
- **Removes the device dropdown** — devices are permanent sidebar rows (the standard macOS
  pattern), with status shown inline (dot + colour), selectable directly.
- **Future-proof** — Photos/Videos slot into LIBRARY with no structural change; multiple
  devices just add rows.
- **Profiles keeps a dedicated view** (as you want) but is clearly associated with the
  Device workflow instead of floating as a peer of "Music."
- Uses the geometry you already like: selected row = accent fill + `accentBorder`, 3pt
  corners, same as the profile list rows today.

> Mockup: `docs/ui-review/01-navigation.svg`

**Open question for you:** should **Sync Profiles** be a single global screen (as now), or
should selecting a device show *its* assigned profile inline and "Sync Profiles" be the
place to manage the library of profiles? My recommendation: keep one global Profiles screen
(your preference), and on each **device** screen show a compact "Active profile: ___"
selector that deep-links into it.

---

## 5. Music tab — sub-tabs & toolbar

### Before
`Albums · Artists · Songs · Playlists` sit as four **floating** buttons pinned left
(selected = white fill + border, unselected = transparent, so unselected tabs read as
weak); then a `Spacer`; then a size slider + a hamburger menu + a search field that
**expands leftward** (backwards for macOS). The row splits into two unrelated clusters.

```
[Albums][Artists] Songs  Playlists            [▭slider] [≡]      Filter⌕
   ▲ selected = white+border   ▲ unselected = faint      ▲ cramped, right-aligned, odd
```

### After
Keep the exact content panes (you like them — untouched). Fix only the control row:

- **Group the tabs** into one segmented control (shared visual container) so they read as a
  set, not four loose buttons. Reuse the *same* tab component the Profiles card uses (which
  already feels more finished) so Music and Profiles match.
- **One right-aligned toolbar**, in conventional order: **search field (always visible,
  expands rightward)**, then sort, filter, and (albums-only) grid/list + size — collapsed
  into a single tidy control cluster, not scattered.

```
┌ Albums │ Artists │ Songs │ Playlists ┐                 ⌕ Filter…   ⇅ Sort  ▚ View
└─────────────────────────────────────┘
```

> Mockup: `docs/ui-review/02-music-tabs.svg`

The win is *balance and convention* — same information, less ad-hoc. Apply the identical
pattern to the Profiles editor tabs so there's one tab style app-wide.

---

## 6. Device screen — from "dashboard" to "inspector"

### Before
Two big `ChromeCard`s side by side. Left = a **192×256 iPod illustration** (with the 4pt
corner), device name (glassy edit field), a stats list, and a rounded-cap storage bar.
Right = sync heading + gear, a profile checkbox list, a **nested inset card** sync summary,
progress, settings summary, and the Sync button. The illustration is the visual centrepiece
and the translucency + nested borders are the "gimmicky" feel.

### After
Treat it like a settings/inspector page — quiet, dense, all controls, no hero art:

- **Shrink or drop the iPod illustration.** Replace the big device drawing with a compact
  device header: small flat glyph + **name / model / connection status** on one line. Give
  the reclaimed space to the things that matter (sync + storage).
- **Flatten everything to 3pt**, remove the glassy fields (solid white with a 1px border),
  and **un-nest** the sync summary (a plain inset panel separated by a divider, not a
  bordered card inside a card).
- **Storage as a flat bar** (square ends, 1px track), matching the restraint of the library.
- **Two calm sections** instead of two competing cards: *Device* (identity + storage +
  last-synced) and *Sync* (active profile + summary + the primary Sync button). Same
  `ChromeCard` shell, far less inside each.

**Finalised layout (v3, agreed):** one calm pane — no separate boxed/“headed” Sync section.

```
┌───────────────────────────────────────────────┐
│ [glyph] iPod Classic                ● Connected │  ← small glyph; status top-right (Option A)
│         Video 5th gen · 512 GB                  │
│                                                 │
│ [██████░░░░░░░]  43.8 GB of 512 GB · 2,584      │  ← flat storage bar
│ Last synced: Entire Library · today 15:25       │
│ ─────────────────────────────────────────────  │
│ DETAILS                                         │  ← version/serial on-screen, above sync
│   Model     iPod Video (5th gen)                │
│   Capacity  512 GB · Firmware 1.1.2             │
│   Serial    9C7X…    · Format iTunesDB          │
│ ─────────────────────────────────────────────  │
│ Profile  [ Entire Library      ▾ ]  ⚙ Sync settings │  ← everyday controls; ⚙ → sheet
│ 2,584 songs · 43.8 GB · 3 playlists             │
│                    [          Sync          ]   │  ← button reads “Sync” (primary, anchored bottom)
└───────────────────────────────────────────────┘
```

> Order: identity + storage → **Details** → Profile + Sync (Sync is the primary action,
> anchored at the bottom of the pane).

Decisions locked here:
- **Small header glyph** (not the hero illustration).
- **Connected = Option A** — quiet, top-right of the header, aligned with the name.
- **Button = “Sync”** (not “Sync Now”).
- **No boxed/headed “Sync” section** — profile + Sync flow directly under the device info,
  separated only by a light divider.
- **Version / serial / model / firmware stay on-screen** in a quiet `DETAILS` block (not
  hidden behind a click; may collapse later if it feels heavy).
- **Advanced sync settings** (audio quality/bitrate, artwork resize, full-clean-sync, album-
  artist, feat-normalize) live behind **⚙ Sync settings → a sheet**, keeping the page calm.

> Mockups: `docs/ui-review/03c-device-screen-v3.svg` (final), `03-device-screen.svg` /
> `03b-device-screen-v2.svg` (earlier iterations).

---

## 7. Profiles — integration & consistency

Profiles is actually the **most "finished"** screen (tabs live inside the card header, which
is why it reads as organised). Keep the structure; align it to the anchor style:

- Adopt the **same tab component** as the redesigned Music tab (§5) so there's one pattern.
- **Un-nest** the album/playlist item cards (§3) — each row shouldn't be its own bordered
  card inside the content card; use dividers/striping like the song list.
- Keep the **profile colour bar** identity marker and the profile list rows (these are
  good). When nav moves to the source list (§4), the Profiles *screen* stays; only its entry
  point changes (a sidebar row under DEVICES instead of a peer of Music).

---

## 8. Smaller notes across the app

- **Bottom status strip** (sidebar/panel toggles + now-playing) is already clean — leave it.
- **Album detail panel** uses `cornerRadius: 0` (flat bottom) — fine and intentional, but
  make the *choice* consistent (either the detail pane is a bordered card or a flush pane,
  not half-and-half).
- **Empty/placeholder states** (e.g. "No device selected") are good; reuse that calm,
  centered pattern for empty Music/Photos/Videos too.
- **Genre pills** (translucent white) are the one place light translucency reads fine — but
  once we ban glassy overlays elsewhere, make these solid `insetCard` too for consistency.

---

## 9. Prioritized roadmap

**Phase 1 — Global polish (low risk, high impact, ~1 sitting):**
1. Corner radius unification (kill 4pt). 2. Remove glassy overlays. 3. Un-nest cards.
4. One hover/pressed style. — *Nothing moves; the app just gets crisper.*

**Phase 2 — Music tab control row (§5):** segmented tabs + right-aligned conventional toolbar;
extract a shared `SectionTabBar` component (also used by Profiles).

**Phase 3 — Device screen (§6):** shrink the illustration, flatten, un-nest, inspector layout.

**Phase 4 — Navigation source list (§4):** the structural change — device rows replace the
dropdown, LIBRARY (Music/Photos/Videos) + DEVICES (devices + Profiles) groups. Do this once
the content types are on the horizon so it's built for them from the start.

Phases 1–3 are independent and shippable on their own; Phase 4 is the bigger structural move
and the right moment to design for Photos/Videos.

---

## 10. Decisions I need from you

1. **Nav grouping (§4):** happy with `LIBRARY {Music, Photos, Videos}` + `DEVICES {devices…,
   Sync Profiles}` as a source list, devices as rows (dropdown removed)?
2. ~~iPod illustration~~ — **DECIDED: small header glyph.** Connected tag = **Option A**.
   Button = **“Sync”**. Sync settings behind **⚙ → sheet**. Version/serial in on-screen
   `DETAILS`. (§6 finalised.)
3. **Profiles entry point (§4):** one global Profiles screen (my rec) vs. per-device profile
   assignment shown on the device screen?
4. Priority order — start with Phase 1 global polish, or jump to the nav/device structural
   work first?
