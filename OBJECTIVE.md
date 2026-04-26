# Objective: MMOG Infrastructure — Done Condition

## Scope

VR headset integration lives in `multiplayer-fabric-xr-dev`; RPG game-logic
lives in `multiplayer-fabric-rpg`. This segment is the shared fabric beneath
both: engine assembly, zone networking, and interest management.

## Goal

Get the assembled `multiplayer-fabric` Godot engine running
`multiplayer-fabric-humanoid-project` with a humanoid character walking on the
`mire` plane, connected to a local zone server.

When this is done, the segment is complete.

## What "fully working" means

The gitassembly in `multiplayer-fabric-merge` produces a clean `multiplayer-fabric`
branch. That engine boots the humanoid project headlessly with no errors, loads a
VRM character into the `mire.tscn` flat terrain, and the character can move forward
on the plane. The zone server accepts a WebTransport connection from the client.

### Pass condition

| Check | What it verifies |
|---|---|
| `godot --headless --quit-after 300` exits 0 | Engine boots, scene loads, no SCRIPT ERROR |
| VRM character visible in `mire.tscn` | MToon shader, humanoid skeleton, and VRM importer all work against the assembled engine |
| Character steps forward on plane | Basic rigid-body plane collision and input routing functional |
| Zone server accepts connection | WebTransport (HTTP/3) handshake succeeds between client and `zone-backend` |

## Current blockers

1. **gitassembly must run** — `multiplayer-fabric-merge/update_godot_v_sekai.exs`
   needs to be executed inside a checkout of `multiplayer-fabric-godot` on `master`
   to produce the assembled `multiplayer-fabric` branch. The branch is then
   force-pushed to `V-Sekai-fire/multiplayer-fabric-godot`.

2. **Engine not yet built** — `multiplayer-fabric-build` must compile the
   assembled engine for the target platform before the headless test can run.

3. **Zone connection untested** — `zone-backend` WebTransport endpoint has not
   been smoke-tested against the assembled engine's HTTP/3 module.

## Stack

| Layer | Component | Still Godot? |
|---|---|---|
| Engine assembly | `multiplayer-fabric-merge` — gitassembly → `multiplayer-fabric` branch | Yes |
| Godot engine fork | `multiplayer-fabric-godot` — branch `multiplayer-fabric` | Yes |
| Build system | `multiplayer-fabric-build` — SCons, Justfile | Yes |
| Zone server | `multiplayer-fabric-humanoid-project` — headless Godot running `mire.tscn` | Yes — scene the server simulates |
| Zone registry | `multiplayer-fabric-zone-backend` — Elixir/Phoenix, WebTransport/HTTP3 | No |
| Interest management | `multiplayer-fabric-predictive-bvh` — Lean-verified BVH | No |
| Clients / players | taskweft/picoquic bots — C++20, HTN planner, ~15 MB each | No — replaces Godot client |
| Observer / monitor | `multiplayer-fabric-zone-console` — ratatui TUI, WebTransport observer | No — replaces Godot window |

## TUI demo / load test

The load test is also a watchable demo. The zone server runs headless; taskweft
bots provide the population; `zone_console` renders a live top-down ASCII map of
the mire plane in the terminal. No VR headset, no GPU, no Godot window required.

```
┌─ mire.tscn ── zone 0 ── 20 Hz ─────────────────┐
│  · · · · · · · · · · · · · · · · · · · · · · ·  │
│  · · M · · · · · · M · · · · · · · · M · · · ·  │
│  · · · · · · M · · · · · · · · · · · · · · · ·  │
│  · · · · · · · · · · · M · · · · · · · · M · ·  │
│  · M · · · · · · · · · · · · · M · · · · · · ·  │
│  · · · · · M · · · · · · · · · · · · · · · · ·  │
│  players=16  entities=896  tick=50ms  hlc=…      │
└─────────────────────────────────────────────────┘
```

### How it works

| Component | Role |
|---|---|
| Headless Godot zone server | Owns the simulation; constant-work 20 Hz loop; broadcasts 100-byte entity snapshots |
| Taskweft/picoquic bots (×16) | C++20 processes; HTN planner walks each mire; picoquic sends position datagrams |
| `zone_console` TUI | Connects as observer via WebTransport; decodes snapshots; renders ASCII map via `ex_ratatui` |

Bots are **not** full Godot processes. Each is ~15 MB. On this M2 Pro (12 cores, 32 GB):

| Resource | Allocation |
|---|---|
| zone-backend | 1 core |
| Zone server | 1 core |
| OS headroom | 1 core |
| 16 bot processes | ~240 MB RAM, share remaining 9 cores |

### Scenarios

All four run on the flat mire plane with 16 taskweft bots. Each is a distinct
HTN domain passed to `tw_bot --scenario <name>`. Each maps to a formal Lean
construct in `multiplayer-fabric-predictive-bvh`.

#### Concert
**Lean:** `separatedConcertFits` / `naiveConcertFits` — `Interest/AuthorityInterest.lean`

All 16 bots converge to the same zone area. Local players (authority on this
zone) fill the authority budget; remote players (authority on a neighbouring
zone) fill the interest/replica budget. The separation theorem proves the total
visible count `lo + re ≤ (cap − headroom) + InterestCapacity` exceeds what the
naive model allows.

Pass: `separatedConcertFits 16 0 1800 400` holds; `concert_coexistence` theorem
fires; interest-set snapshot for the observer equals full 896 slots.

```
· · · · · · · · ·
· · · MMMMMM · ·   ← all mires converging
· · · MMMMMM · ·
· · · · · · · · ·
```

#### Chokepoint
**Lean:** C7/G221 `currentFunnelPeakVUmTick` = 60 m/s — `Spatial/ScaleContradictions.lean`

Bots rush through a narrow corridor simultaneously, reaching speeds at or above
`vMaxPhysical` (10 m/s) due to crowd pressure. The theorem
`c7_current_funnel_exceeds_cap : vMaxPhysical < currentFunnelPeakVUmTick`
describes the adversarial case. The zone server must register the funnel segment
with its own per-segment velocity rather than the global cap; mitigation is the
C7 theorem `c7_funnel_mitigation_ge`.

Pass: no entity exits the BVH ghost bound; zone server logs the funnel segment
velocity override; no SCRIPT ERROR.

```
· · · · | · · · ·
M M M M → · · · ·   ← group A
· · · · ← M M M M   ← group B
· · · · | · · · ·
```

#### Convoy
**Lean:** `wpPeriodMin` / `migration_completes_before_phase_flip` — `Protocol/WaypointBound.lean`

A mover entity leads a column of cabin entities (bots) on a periodic route that
crosses the zone boundary and returns — one full cycle = 2 × `wpPeriodMin`.
The theorem `migration_completes_before_phase_flip` proves each STAGING
migration finishes before the phase reversal, preventing hysteresis resets.
The convoy half-cycle must satisfy `wpPeriodValid(WP_PERIOD) = true`.

Pass: all cabin bots complete ≥ 2 zone crossings; `wpPeriodValid` holds for the
chosen period; no cabin disappears during STAGING.

```
· · · · · · · · ·
· M M M M M M M ·   ← mover + cabins →
· · · · · · · · ·
```

#### Ragdoll
**Lean:** C1/G13 `g13_vTrue` = 15 m/s + C2/G29 `aHalfMinForearm` — `Spatial/ScaleContradictions.lean`

Bots collide head-on, momentarily reaching `g13_vTrue` = 15 m/s (above the
10 m/s `vMaxPhysical` cap). The C1 mitigation clamps to `vMaxPhysical`; C2
accounts for forearm half-acceleration `aHalfMinForearm` in ghost bound
expansion. Tests that the constant-work tick stays stable under rapid velocity
and acceleration spikes.

Pass: server clamps impulse velocities to `vMaxPhysical` (C1); ghost bound
expansion uses `aHalfMinForearm` (C2); tick p99 ≤ 2× baseline.

```
 M→  ←M  M→  ←M
   ↘ ↙    ↘ ↙
    ✕       ✕      ← collision impulses
```

### Win condition

**All four scenarios complete with 16 mires visible in `zone_console`, 20 Hz
tick maintained throughout, and zero entity loss — all in a terminal.**

### Pass conditions

| Check | Target |
|---|---|
| All 16 mires on the map | Zone server logs 16 registered players in each scenario |
| Entity slots | 896 occupied (16 × 56); zone never exceeds 1,800 |
| Zone tick rate | Holds at 20 Hz; `tick=50ms` in TUI status bar for all 4 scenarios |
| Concert | All mires reach centre; interest set = full 896 slots for observer |
| Chokepoint | STAGING migrations fire and complete; no mire disappears during crossing |
| Convoy | Tick-time variance ≤ Concert (correlated movement ≤ random scatter cost) |
| Ragdoll | No tick spike > 2× baseline during collision burst |
| Clean shutdown | `FabricSnapshot` written after each scenario; positions recoverable |

### How to run

```sh
# 1. Build the taskweft bot
cd multiplayer-fabric-taskweft/standalone
cmake -B build && cmake --build build --target tw_bot

# 2. Start zone stack
cd multiplayer-fabric-zone-backend && mix phx.server &
ZONE_IDX=0 ZONE_TOTAL=1 godot --headless --path ../multiplayer-fabric-humanoid-project &

# 3. Spawn 16 bots
for i in $(seq 1 16); do
  ./build/tw_bot --host localhost --port 4433 --seed $i &
done

# 4. Watch in zone_console
cd multiplayer-fabric-zone-console
./zone_console http://localhost:8888
# → ASCII map populates with 16 moving M glyphs
# → status bar: players=16  entities=896  tick=50ms
```

## Definition of done

```sh
# 1. Assemble engine
cd multiplayer-fabric-merge/multiplayer-fabric-godot
elixir ../update_godot_v_sekai.exs

# 2. Build
cd multiplayer-fabric-build
just build-platform-target macos editor arm64 double no

# 3. Smoke test
godot --path ../multiplayer-fabric-humanoid-project --headless --quit-after 300
# → exit 0, no SCRIPT ERROR
# → "Humanoid loaded" in stdout
# → character steps forward on mire plane

# 4. Zone connection
mix phx.server &   # zone-backend
# → WebTransport handshake logged
```
