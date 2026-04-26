# Objective: MMOG Infrastructure — Done Condition

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

| Layer | Component |
|---|---|
| Engine assembly | `multiplayer-fabric-merge` — gitassembly → `multiplayer-fabric` branch |
| Godot engine | `multiplayer-fabric-godot` — branch `multiplayer-fabric` |
| Build system | `multiplayer-fabric-build` — SCons, Justfile |
| Test project | `multiplayer-fabric-humanoid-project` — `humanoid/scenes/mire.tscn` |
| Character | VRM + MToon + humanoid skeleton via `addons/vrm` and `addons/humanoid` |
| Zone server | `zone-backend` — Elixir, WebTransport/HTTP3 |
| Interest management | `multiplayer-fabric-predictive-bvh` — Lean-verified BVH |

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
