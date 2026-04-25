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
