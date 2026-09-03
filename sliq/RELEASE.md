# Sliq — Release & TestFlight Guide

**Status 2026-09-01: the build side is done.** A distribution-signed, TestFlight-ready
IPA exists at `build/export/Sliq.ipa` (6.7 MB). Everything remaining is App Store
Connect setup, which needs your Apple account.

## Before every upload

```bash
xcodebuild test -scheme Sliq -destination 'platform=iOS Simulator,name=iPhone 17 Pro'
```

The suite is fast (well under a second) and covers the rules engine, the level table and
saved progress. Then bump the build number (see below) and archive.

## What's already verified

| Item | State |
|---|---|
| Team | `S7K2GRB2RN` ("Andy Bower") — set on project + target |
| Bundle ID | `com.bowerharry.sliq` — **registered** with Apple |
| Game Center capability | **enabled** on the App ID (confirmed in the profile) |
| Distribution signing | `Apple Distribution: Andy Bower (S7K2GRB2RN)` — cert auto-created |
| TestFlight entitlement | `beta-reports-active: true` |
| Export compliance | `ITSAppUsesNonExemptEncryption = NO` (no upload prompt) |
| App icon | embedded and generating all sizes |
| Privacy manifest | `PrivacyInfo.xcprivacy` — no tracking, no data collected |
| iPad | portrait-locked with `UIRequiresFullScreen` |
| Version / build | `1.0` / `7` (`MARKETING_VERSION` / `CURRENT_PROJECT_VERSION`) |

## ⚠️ One thing to check first

During export Xcode reported:

> No provider associated with App Store Connect user

That means the Apple ID signed into Xcode (`bowerharry@icloud.com`) may not yet have
App Store Connect access on the "Andy Bower" team. If uploads bounce, fix it at
[appstoreconnect.apple.com](https://appstoreconnect.apple.com) → **Users and Access** —
the Account Holder must invite that Apple ID with at least the **App Manager** role.
(Developer-portal access and App Store Connect access are separate things; signing
already works, which is why the archive succeeded.)

## Step 1 — Create the app record

App Store Connect → **My Apps** → **+** → **New App**

| Field | Value |
|---|---|
| Platform | iOS |
| Name | `Sliq` — must be globally unique; if taken, try `Sliq Puzzle` |
| Primary Language | English (UK) |
| Bundle ID | select `com.bowerharry.sliq` from the dropdown |
| SKU | `SLIQ001` (internal only, never shown) |
| User Access | Full Access |

**The build upload will fail without this record** ("No suitable application records
were found").

## Step 2 — Create the two Game Center leaderboards

App Store Connect → your app → **Features** → **Game Center** → **Leaderboards** → **+**
→ *Single Leaderboard*.

The **Leaderboard ID must match the code exactly** or submissions silently vanish
(see `Sliq/Utils/GameCenter.swift`):

### Leaderboard 1 — bear heads

| Field | Value |
|---|---|
| Reference Name | Total Bear Heads |
| **Leaderboard ID** | `sliq.bears.total` |
| Score Format Type | Integer |
| Score Submission Type | Best Score |
| Sort Order | High to Low |
| Score Range | 0 – 108 (36 levels × 3) |
| Localization (English) | Display Name: `Bear Heads Collected`, Score Format: Integer |

### Leaderboard 2 — ranked free play

| Field | Value |
|---|---|
| Reference Name | Best Free Play Score |
| **Leaderboard ID** | `sliq.freeplay.best` |
| Score Format Type | Integer |
| Score Submission Type | Best Score |
| Sort Order | High to Low |
| Score Range | leave open |
| Localization (English) | Display Name: `Best Free Play Run`, Score Format: Integer |

Only untouched **RANKED** preset runs submit to leaderboard 2 — changing any stepper or
chip marks the run custom and it won't post. That keeps the board comparable.

### ⚠️ The localization is not optional

A leaderboard with no localization shows in Game Center as **\*MISSING TITLE\***
followed by Apple's internal reference number. Each leaderboard needs at least one
language added under **Leaderboard Localization**:

| Field | Leaderboard 1 | Leaderboard 2 |
|---|---|---|
| Language | English (UK) | English (UK) |
| Display Name | `Bear Heads Collected` | `Best Free Play Run` |
| Score Format | Integer | Integer |
| Score Format Suffix (singular) | ` bear head` | *(leave blank)* |
| Score Format Suffix (plural) | ` bear heads` | *(leave blank)* |

### ⚠️ Adding a third leaderboard? Check this first

Both boards above are **Best Score**, and both values only ever go up — total
stars collected, best ranked run. The app leans on that. A submission that fails
(offline, signed out, Apple having a moment) is not queued and replayed; the app
just remembers the highest value it still owes each board as a single number,
and sends that again on sign-in and on every return to the foreground. It cannot
go stale or run out of retries, because a later value supersedes an earlier one
by definition.

**That reasoning breaks the moment a leaderboard isn't monotonic.** A "most
recent run" board, a per-session score, a Score Submission Type of anything but
Best Score — for any of those, one number per board is wrong, because the value
you failed to send is not superseded by the next one, and collapsing them loses
a result. Such a board needs a real queue of pending submissions.

So: keep new leaderboards **Best Score with a value that only grows**, or change
`GameCenter.swift` to queue properly for the one that doesn't. Don't add the
board and assume the existing retry covers it.

### "Pre-release — not yet live" is expected

Game Center shows this banner on every leaderboard until the app version is actually
released on the App Store. Scores still submit and read back normally in sandbox
(development and TestFlight builds), so it is **not** a fault — it disappears at launch.

## Step 3 — Upload the build from Xcode

1. `open Sliq.xcodeproj` (or it's already open)
2. Set the run destination to **Any iOS Device (arm64)** — not a simulator
3. **Product → Archive**
4. When the Organizer opens: select the archive → **Distribute App**
5. Choose **TestFlight & App Store** → **Upload**
6. Accept the automatic signing prompts; Xcode uploads and processing begins

Processing takes 5–15 minutes. You'll get an email when the build is ready.

> Already-built alternative: the signed IPA at `build/export/Sliq.ipa` can be dragged
> into Apple's free **Transporter** app instead of re-archiving.

## Step 4 — TestFlight

- **Internal testing** (up to 100 people on your team, **no review required**): TestFlight
  tab → Internal Testing → add testers → select the build. Available within minutes.
- **External testing** (up to 10,000, needs a one-time **Beta App Review**): requires
  test information — What to Test, a description, contact email, and a demo account
  if login were needed (it isn't here).

## Build numbers

Every upload needs a **unique** `CFBundleVersion`. Before each new upload bump
`CURRENT_PROJECT_VERSION` in the project (or run):

```bash
cd /Users/harry/Sliq/Sliq && agvtool next-version -all
```

`MARKETING_VERSION` (1.0) only changes for user-facing releases.

## App Store submission (later — not needed for TestFlight)

### Metadata draft

**Subtitle** (30 chars max)
> Slide, match, feed the bear

**Promotional text** (170)
> 36 hand-tuned levels of sliding, matching and planning ahead. Every rotation is
> predictable — so every mistake is yours.

**Description**
> Sliq is a puzzle game about reading the board two moves ahead.
>
> Tiles carry a number that is both their score and their remaining moves. Slide them
> into a matching coloured wall to score — but every slide costs a point, so the tile
> you save is worth more than the tile you spend.
>
> The walls rotate on a timer, always in a direction you can predict. The edge arriving
> at the bottom in two rotations is the one worth planning for. Get it right and tiles
> tumble into the pipes and off to the bear. Get it wrong and the board fills up.
>
> • 36 levels across Easy, Medium and Hard, each tuned by simulation
> • At least 8 seconds to think on every single turn — never a reflex test
> • Free Play with your own rules, plus a ranked mode for the leaderboard
> • Three bear heads to earn on every level — 108 in total
> • No ads, no timers, no purchases. Just the puzzle.

**Keywords** (100 chars, comma-separated, no spaces)
> puzzle,tiles,logic,brain,strategy,match,slide,relaxing,offline,indie,thinking,board

**Category**: Games → Puzzle (secondary: Games → Strategy)
**Age rating**: 4+ (no objectionable content)
**Privacy**: *Data Not Collected* — the app stores progress only in local UserDefaults.
Game Center identities are handled by Apple, not collected by the app.
**Support URL / Marketing URL**: required — a simple GitHub Pages or personal page works.

### Screenshots (required for App Store, not TestFlight)

**Already generated** at 1320×2868 (6.9" iPhone — Apple's required size) in
`docs/appstore/`:

| File | Screen |
|---|---|
| `01-menu.png` | Title screen with falling tiles |
| `02-gameplay.png` | Mid-level, tiles riding the belt to the bear |
| `03-levels.png` | Level select showing 22/36 bear heads earned |
| `04-bundles.png` | Bundle select with unlock requirements |
| `05-freeplay.png` | Free Play with the RANKED preset |
| `06-celebration.png` | The completion feast (note: spoils the ending) |

Drag them into App Store Connect → your app → the 6.9" Display slot. iPad 13"
(2064×2752) is only needed if you keep iPad support in the listing.

## Known gaps before a public release

- **No music track** — SFX are in; the music toggle exists but has no asset.
- **Untested on device** — everything so far is simulator-verified.
- **Game Center cannot be tested in the Simulator at all.** Verified from the logs:
  `Could not create endpoint for service name: com.apple.GameOverlayUI.dashboard-service`
  — the simulator has no Game Center overlay services, so the dashboard never appears
  no matter what the app does. Test on a real device, signed into Game Center in
  Settings, running a TestFlight or development build.
- **The leaderboards must exist in App Store Connect** (Step 2) before the dashboard
  shows anything. With no leaderboards configured it opens empty — which looks
  identical to a broken integration.
