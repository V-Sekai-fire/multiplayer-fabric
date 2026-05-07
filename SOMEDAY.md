# Someday / Maybe

Items that are good ideas but not on the critical path.

## Quest 3 OpenXR on macOS — minimum smoke test with multiplayer-fabric

Pass condition: Godot player scene initialises an OpenXR session via Meta XR
Simulator on macOS, connects to the zone server, and sends one CH_PLAYER
datagram (server ACKs).

Steps:
1. Install Meta XR Simulator on macOS — it registers itself as the active OpenXR runtime
2. In Godot project: enable OpenXR, confirm `XRServer.find_interface("OpenXR")` returns non-null
3. Run `observer.tscn` (with XROrigin3D wired) via `FabricMultiplayerPeer` at `127.0.0.1:7443`
4. Call `send_player_input()` once — confirm server ACKs in console

That is the full minimum bar. Perf, hand tracking, and passthrough are out of scope until the smoke test is green.

See: [20260425-godot-player.md](manuals/decisions/20260425-godot-player.md)

## Three.js WebGPU Observer (browser TO)

Full browser observer without a Godot runtime. Superseded for native CI by
the Godot headless observer; still useful for a zero-install browser view.

- parseInterest() TypeScript — 100-byte CH_INTEREST wire format
- WebTransport connection to zone server (browser API)
- Three.js WebGPU scene — OrthographicCamera at SWING_ELEVATION
- Operator overlay canvas (load bars, dot clustering)
- Playwright Phase 1 TO test: `window.__entities.length > 0`

See: [20260425-godot-observer.md](manuals/decisions/20260425-godot-observer.md)

## Operator overlay

Load bars + dot clustering CanvasLayer over observer.tscn.
See: 20260425-operator-overlay.md

## Headless test matrix — full CI wiring

- Phase 2 GO+TO cross-check (Godot observer + Three.js observer together)
- 5 branch protection checks (GO, TO, GP, TP, GO+TO)
- taskweft PropCheck NUMTESTS=100 against production CockroachDB

## Branch maintenance

- elixir update_godot_v_sekai.exs once CI green
- merge dry-run CI job
- Archive feat/multiplayer-fabric

## Migrated from TODO (infra not on the player path)

- **`GODOT_CPP_BRANCH: 4.5` mismatch in `linux_builds.yml`** — engine is 4.7.dev; mismatch causes every feat-branch CI to fail at `Compilation (godot-cpp)`. Lives at line 16 of both `godot/.github/workflows/linux_builds.yml` and the `build/godot/` mirror.
- **Vendored `thirdparty/predictive_bvh/` snapshots out of sync** — `godot/thirdparty/predictive_bvh/` (632 K) and `build/godot/thirdparty/predictive_bvh/` (582 M) are drift-prone copies of the canonical `predictive-bvh` repo (research-tier modules extracted, library-root reshaped). Refresh or convert to submodules.
- **`headless_tests.yml` wiring detail** — Docker `zone-fabric` Godot process runs but GDScript fails to initialize (no GodotSharp → WebTransport server never starts). Folds into the *Headless test matrix* item above.
