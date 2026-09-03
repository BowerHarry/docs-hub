# Difficulty & balance

**The design goal, in one line: difficult but possible. Losing to bad luck
occasionally is fine; skill must prevail most of the time.**

Every number in `Engine/LevelData.swift` was measured, not guessed. This
document says how, and records the decisions worth remembering.

---

## How it is measured

`Simulator/main.swift` auto-plays every level against the real rules
(`Engine/SliqEngine.swift`, seeded RNG) with four skill tiers:

| bot | models |
|---|---|
| `random` | button-masher — the skill floor |
| `sloppy` | takes an obvious score if one is in front of it, no lookahead, will burn a tile to 0 |
| `greedy` | casual player: takes visible scores, feeds tiles onto matching bottom edges |
| `planner` | skilled player: tracks the rotation, sets up edges a turn ahead, protects tile value |

All four move at the **same pace** — `humanMovesPerSecond = 0.30`, measured from
a real session — so decision quality is the only variable. That matters: the
bots used to be modelled at 1.0–1.5 moves/sec, four to five times faster than
anyone actually plays, which let weak play brute-force a score by sheer volume.

```bash
swiftc -O Sliq/Engine/LevelData.swift Sliq/Engine/SliqEngine.swift Simulator/main.swift -o build/sliqsim
build/sliqsim            # full sweep, 200 seeds/cell -> docs/BALANCE_SWEEP.md
build/sliqsim --quick    # 40 seeds, for a smoke check
build/sliqsim --calibrate  # achievable-score quantiles with the target removed
build/sliqsim --tune       # solve targets for a wanted win rate
build/sliqsim --zeros      # win rate vs number of 0-tile blockers
```

The simulator models the shipped timing exactly — a rotation locks input for its
whole visual batch, and the next countdown starts after it settles. **If you
change that in `GameScene`, change it in the simulator too**, or its numbers stop
meaning anything.

## Where the curve sits today

From `docs/BALANCE_SWEEP.md` (200 seeds per cell). Win rates, low–high across
the twelve levels of each bundle:

| | random | sloppy | greedy (casual) | planner (skilled) |
|---|---|---|---|---|
| Easy | 67–99% | 82–100% | 82–98% | 88–100% |
| Medium | 7–36% | 19–60% | 22–57% | 30–70% |
| Hard | 2–44% | 5–60% | 10–66% | 16–80% |

Easy is an on-ramp that almost anyone clears. Medium is where skill starts to
separate the tiers. Hard leaves the best bot under 20% on its hardest levels.
No level ever stalls — every game ends in a win or a full board.

Pressure matches the intended feel: **keep the bottom row clear; past roughly
half fill you are a turn from losing.**

| | peak fill | bottom row blocked | idle turns |
|---|---|---|---|
| Easy | 25% | 64% | 8% |
| Medium | 40% | 73% | 1% |
| Hard | 40% | 75% | 1% |

Peak fill never crosses the danger line, and the bottom row tightens steadily
from Easy to Hard.

## How targets are set

Targets come from measured achievable-score quantiles (`--calibrate`), keyed to
the tier each bundle is written for, then capped so no level becomes a marathon:

- **Easy** — casual-keyed: greedy P40 (L1–3) → P50 (L4–8) → P60 (L9–12)
- **Medium** — skill-keyed: planner P45 (L1–6) → P55 (L7–12)
- **Hard** — planner P50 (L1–6) → P60 (L7–11) → P75 for the finale

Floor 30, rounded to 5.

Stars are 33 / 65 / 100 % of target. 1–2★ come from strong losses, 3★ from a
win — losing well still earns progress, which is what keeps a hard level from
being a wall.

## Two separate dials

**Spawn rate governs how much there is to do. Target score governs difficulty.**
Changing one moves both, so they are re-derived together or not at all.

This is the single most useful thing learned from playtesting. The symptom that
found it: *"the game is less fun when it feels like there are less meaningful
actions for the player"* — measured, that was ten of twelve Easy levels sitting
at 20–51% idle turns because the boards were nearly empty. Raising Easy's
`spawnBase` from 2 to 3 cut idle turns to 8% average; raising growth and cap as
well over-corrected and dropped win rates hard.

## Playtest telemetry

The bots model difficulty. They cannot model fun, so the app records a local play
log — `Utils/Telemetry.swift`, active in release builds so TestFlight sessions
count.

Play on device → **Settings → PLAYTEST → Export Play Log** → AirDrop the
`.jsonl` to the Mac → `./scripts/analyse-play-log.py <file>`.

Nothing is transmitted anywhere; the file sits in the app's Documents folder
until the player explicitly shares it.

**The log covers one build.** Installing a new one clears it, so an export is
always a clean read on the numbers currently shipping rather than a mix of two
different games. Export before you install an update, or that round is gone.

The report answers two questions:

- **Difficulty** — attempts per level, how close the losses came (a wall of 90%+
  losses means the target is a touch high; sub-55% means something structural is
  wrong), moves wasted burning tiles to 0, and a per-rotation sparkline showing
  whether the player was progressively overwhelmed.
- **Enjoyment** — what the player did after each result. Retry-after-loss is the
  best engagement signal there is: high means losing is motivating, low means it
  is driving them away. Also which level a session died on, since that is where
  the game actually loses people.

---

## Decisions worth remembering

**The board must never rotate unpredictably.** Look-ahead is half the skill in
this game, so there is no random rotation pattern. Every level turns one constant
direction.

**Strict alternation was structurally broken.** A clockwise/counter-clockwise
pair returns the bottom and right edges to where they started, so under strict
alternation those edges never regenerate and the board stagnates. Seventeen
levels were converted to a single direction.

**The player always gets at least 8 seconds a turn.** Hard used to run 1.8–3.2s
intervals, which playtested as "the player can't actually do anything". Intervals
now floor at 8s and Hard gets its difficulty from tile mix, obstacles, spawn
pressure and rotation direction — not from pace.

**Nobody scores before their first input.** The pre-placed board and the opening
spawn wave are both score-neutral, so no game opens with free points.

**A spent 0-tile flushes through the grate instead of blocking forever.** This
was a rules hole, not tuning: a 0-tile could never move and could never match a
border, so once gravity settled it, it dead-cased that cell for the rest of the
level — attacking the core strategy of keeping the bottom row clear, by pure
spawn luck. Measured with `--zeros`, one blocker cost Hard 5 eleven points of win
rate and three cost it forty-two. Two fixes were prototyped: making 0-tiles
movable barely helped (relocating a dead cell does not help; the tile has to
*leave*), and deleting them outright removed the obstacle along with the luck.
Shipped instead: a 0-tile reaching the bottom row drops through for no points.
Still unscoreable, still costs a cell and tempo on the way down, but it always
leaves. The 0-vs-1-blocker gap disappeared and the level curve was unchanged.
It plays no chime and shows no "+0", but it still rides the pipe and gets eaten
— junk goes down the chute.

**The valve is not modelled in the simulator, on purpose.** With no clock in the
game, skipping dead waiting time cannot turn a loss into a win. Its value is
removing empty turns, not adding difficulty.
