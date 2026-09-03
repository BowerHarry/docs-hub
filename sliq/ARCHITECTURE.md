# Architecture

Sliq is a SpriteKit game with one hard rule: **the rules of the game live in one
file that knows nothing about drawing.** Everything else follows from that.

```
Sliq/
  Engine/      SliqEngine.swift    the rules — no SpriteKit, no UIKit
               LevelData.swift     the 36 levels, as one table
  Game/        BoardNode           draws the board and animates engine events
               TileNode            one tile
               BorderNode          the rotating colour ring
               FactoryNode         pipes, valve, belt and bear
               PipeRoutes          hand-traced plumbing geometry
  Scenes/      one file per screen
  UI/          reusable components — buttons, cards, sheets, the progress bar
  Models/      GameState, Progress — what survives between scenes and launches
  Utils/       Theme, Constants, Textures, Telemetry, GameCenter
Simulator/     a headless bot that plays the game to measure difficulty
SliqTests/     unit tests
```

## The engine is the truth

`SliqEngine` holds the board, the border and the score. It is deterministic:
every random choice goes through a seeded `SeededRNG`, so a seed replays a game
exactly. It has no dependency on SpriteKit, which is what lets the balancing
simulator compile it on its own and play millions of games headlessly.

Every action returns a list of `EngineEvent`s describing what happened, in
order:

```swift
let events = engine.moveTile(x: 4, y: 1, direction: .right)
board.apply(events)
```

The scene layer never decides anything about the rules. It reads events and
animates them. `BoardNode.apply` turns a batch into a timed schedule and returns
how long the whole batch takes to play out, which is what the scene uses to lock
input during a rotation.

**Drift is expected and healed, not prevented.** Animations are asynchronous and
can be interrupted, so `BoardNode.reconcile()` runs a few times a second,
compares its sprites against the engine's grid, and fixes any difference. That
is why a stuck or vanished tile is a bug that repairs itself within a second
rather than one that ruins a game.

## Levels are data

`LevelData.swift` is a table, not code. Each row states only what that level
changes on top of its bundle's baseline:

```swift
medium(7, every: 8.5, turn: .counterClockwise, target: 130, fill: 0.12,
       mix: (5, 200, 295, 300, 200), anim: 1.47, move: 0.20,
       is: [.speed, .complexity, .rotation], plus: [.target])
```

`mix` is the per-mille chance of a new tile carrying 0, 1, 2, 3 or 4. The level
number in each row is load-bearing: rows are sorted by it and the bundle is
checked for gaps at startup, so a mis-numbered row fails loudly instead of
silently shifting twelve levels along.

Every number in that table was calibrated with the simulator. Change one and
re-run the sweep — see [BALANCE.md](BALANCE.md).

## What the scene layer owns

`GameScene` owns the engine, the board, the factory, the rotation timer and the
HUD, and nothing else. Two things were deliberately pulled out of it:

- **`FactoryNode`** — the pipes, valve wheel, funnel, belt and bear. It is pure
  feedback: the scene hands it `deliver(fromSceneX:value:)` when a tile scores
  and `setFullness(_:)` as the score climbs, and it reports a valve tap back
  through `onValveTurned`. It owns no rules and reads no game state.
- **`GameOverlays`** — the pause, result and celebration sheets. They are built
  from plain values and callbacks, so they hold no reference to the scene and
  the scene keeps its state private.

## An interrupted game is kept

Losing focus writes the board to `Library/Application Support` as a `SavedGame`
— the tiles, the border, the score, and the generator's state. The generator
matters: without it a resumed game would deal different tiles from the one that
was saved, which is a different game wearing the same score.
`SavedGameTests` plays both on and demands the same event stream.

The menu offers it back once, in a `ResumeBanner`. Anything the player does next
clears it — resuming, starting any other game, finishing one, quitting to the
levels, or tapping the banner's ×. Only bundle levels look their rules up in the
table; free play carries its own config, because nothing else describes it.

An attempt picked up this way is marked `resumed` in the play log so it does not
count as another try at the level.

## Persistence

Everything the game remembers is a `UserDefaults` key, and every key is named in
`Models/Progress.swift`. Nothing else in the app writes a key string by hand.

`Progress` is the single source of truth for stars and unlocks; `GameState` is
the handful of values that outlive a scene (which level is being played, the
free-play config, the high score). `Defaults.store` is the backing store, which
tests point at a scratch suite so they never touch real progress.

## The timing contract

This is the fiddliest part of the game and the easiest thing to break.

A rotation locks all input except pause for its **entire** visual batch — the
border turn, then the falls, then the spawns — and the countdown to the next
rotation starts only once that batch has settled:

```
deadline = batch end + the level's interval
```

So the player always gets a full interval of play, whatever the animation speed.
`Simulator/main.swift` models the same thing, which is what makes its difficulty
numbers mean anything. **If you change the timing in one, change it in both.**

"Batch end" is an estimate that `BoardNode.apply` builds while it schedules, and
an estimate that runs short breaks both halves at once: the player gets control
back mid-animation *and* their clock starts early. It used to run short. A score
exit reserved a flat 0.6 s, but the exit waits out any fall the tile is still
finishing, so on a cascade it could outlast its reservation — measured on Hard
12 at up to **317 ms** past the lock lifting. Exits are now reserved from the
moment they can actually start (`stopsMoving`), which brings the same run down
to 17–34 ms.

That residue is a few frames and is not arithmetic: every hop in a batch is an
`SKAction` that resolves on a frame boundary, so a schedule built from exact
durations still lands late — 17–50 ms across ten rotations.
`GameScene.checkUnlockContract` is the DEBUG tripwire on this, set just above
that band at 80 ms, since the fault it exists to catch was 317 ms.

## The board owns its own size

`BoardNode(engine:width:)` is told how much room the scene is giving it and
derives the cell and tile size from that; `TileNode`, `BorderNode` and
`FactoryNode` are handed the numbers they need. Nothing reads a shared global,
so two boards of different sizes can exist at once — a scene mid-transition, or
two previews side by side.

## Losing focus pauses the game

`GameScene` watches `UIApplication.willResignActiveNotification` and shows the
pause sheet. Every deadline in the scene is a wall-clock instant, and SpriteKit
freezes the scene when the app stops being active, so without this a phone call
would fast-forward the board on return — measurably: 25 seconds away used to
cost a full rotation. `shiftClocks(by:)` adds the whole interruption back to the
rotation deadline, the input lock and the board's animation schedule together;
miss one and the board unlocks while tiles are still in the air.

## Three things that look odd on purpose

**`reconcile()` stays, and it caught a real bug.** It repairs the board when
the animation scheduler leaves the sprites out of step with the engine. It was
on probation — it counts what it fixed and ships that with every attempt in the
play log — and the counting paid for itself.

Device play reported **5 repairs across 19 measured attempts (264 rotations)**,
every one of them `outOfPlace` and every one on Hard. One kind, one bundle, is
not what noise looks like, and the cause was this:

A swipe into an open column emits `.tileMoved` then `.tileFell` for the *same
tile* (`SliqEngine.moveTile`; pinned by `SwipeEventShapeTests`). `BoardNode`
schedules them back to back, so the slide's completion fires at the same instant
the fall begins. `TileNode.isAnimatingVisuals` was a single `Bool` written by
both, so whichever finished first cleared it — and when that was the slide, the
tile advertised itself as settled while the fall was still carrying it across
the board. `reconcile()` skips animating tiles and repairs settled ones, so it
did exactly what it was told: dragged the tile back to the cell it was leaving.

It is a count now, not a flag, so an animation can only clear its own bracket.
The race needed a second condition to become visible, which is why it showed up
on Hard and roughly once per 50 rotations rather than on every swipe: the batch
duration is an estimate, and a `tileScored` reserves 0.6 s while its exit can
take longer if it has a fall to wait out. Hard's cascades are the longest, so
`timeUntilSettled` reaching zero early is likeliest there. The per-tile check is
the one that matters, and it is honest now.

`reconcile()` is staying regardless — it is cheap, and it is what stands between
a scheduling slip and a board that lies to the player about where their tiles
are. The next play log says whether the fix held: `boardRepairs` should read
empty across a round of Hard.

**The splash frames are cropped.** The Dream animation is 28 frames, and holding all of
them decoded costs ~90 MB during launch — most of it identical flat teal, stored
twenty-eight times. They ship cropped to the band that actually moves, drawn over the
scene's own teal. `scripts/crop-splash-frames.py` regenerates them from the artwork and
prints the constants; `SplashLayoutTests` fails if the band stops lining up with the
square the launch storyboard draws.

**Nothing in the game runs on a `Timer`.** The auto-rotation deadline is a wall-clock
instant checked in `update(_:)`. A timer fires at an arbitrary point in the frame, so a
rotation could rearrange the scene graph mid-render, and it was a second source of truth
for "is the game running" that had to be invalidated in three places.

## Telemetry

`Telemetry` appends newline-delimited JSON to `Documents/sliq-play-log.jsonl`.
It runs in release builds too — TestFlight playtesting is the point — and
nothing ever leaves the device unless the player exports it from Settings.
`scripts/analyse-play-log.py` turns an exported log into a report.

## Conventions

- **No literal colours, fonts or spacing in a scene.** Use `Theme`.
- **No `SKTexture(imageNamed:)` outside `Textures`.** It caches, and it keeps
  the `.png` suffix in one place.
- **Components don't restyle each other.** If a button needs to look different
  on a cream panel, that belongs on `PillButton`, not at the call site.
- **Every view has a `#Preview`.** See `UI/PreviewSupport.swift`.
