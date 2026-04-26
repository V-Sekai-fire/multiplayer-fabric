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

### Win condition

**16 mires visible in the `zone_console` ASCII map, moving under HTN control,
at 20 Hz, with 896 entity slots occupied — all in a terminal, no Godot window.**

### Pass conditions

| Check | Target |
|---|---|
| All 16 mires appear on the map | Zone server logs 16 registered players |
| Entity slots | 896 occupied (16 × 56); zone never exceeds 1,800 |
| Zone tick rate | Holds at 20 Hz; `tick=50ms` shown in TUI status bar |
| Mires move | Each bot completes ≥ 1 HTN plan cycle (move → arrive → replan) |
| Clean shutdown | `FabricSnapshot` written; positions recoverable on restart |

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
