# Someday / Maybe

Items that are good ideas but not on the critical path.

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
- multiplayer-fabric-merge dry-run CI job
- Archive feat/multiplayer-fabric

## Migrated from TODO (infra not on the player path)

- **`GODOT_CPP_BRANCH: 4.5` mismatch in `linux_builds.yml`** — engine is 4.7.dev; mismatch causes every feat-branch CI to fail at `Compilation (godot-cpp)`. Lives at line 16 of both `multiplayer-fabric-godot/.github/workflows/linux_builds.yml` and the `multiplayer-fabric-build/godot/` mirror.
- **Zone-server cert renewal — `notAfter=2026-05-09`** — `generate-secrets.sh` must be re-run before then; add a cron reminder. Affects `multiplayer-fabric-zone-server/priv/cert.pem`.
- **Vendored `thirdparty/predictive_bvh/` snapshots out of sync** — `multiplayer-fabric-godot/thirdparty/predictive_bvh/` (632 K) and `multiplayer-fabric-build/godot/thirdparty/predictive_bvh/` (582 M) are drift-prone copies of the canonical `multiplayer-fabric-predictive-bvh` repo (research-tier modules extracted, library-root reshaped). Refresh or convert to submodules.
- **`headless_tests.yml` wiring detail** — Docker `zone-fabric` Godot process runs but GDScript fails to initialize (no GodotSharp → WebTransport server never starts). Use the Elixir zone server (`just zone-server-local` from `multiplayer-fabric-hosting/`) as the CI target instead of the Docker image. Folds into the *Headless test matrix* item above.
