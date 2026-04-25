# Someday / Maybe

Items that are good ideas but not on the critical path.

## Three.js WebGPU Observer (Stage 1 — TO)

Full browser observer without a Godot runtime.

- parseInterest() TypeScript — 100-byte CH_INTEREST wire format
- WebTransport connection to zone server (browser API)
- Three.js WebGPU scene — OrthographicCamera at twist/swing SWING_ELEVATION
- Operator overlay canvas (load bars, dot clustering)
- Playwright Phase 1 TO test: `window.__entities.length > 0`

Blocked on: time. CRIS +4 but lower urgency than GO which reuses existing Godot tooling.
See: 20260425-threejs-observer.md, 20260425-headless-test-matrix.md

## Operator overlay

Load bars + dot clustering CanvasLayer over observer.tscn.
See: 20260425-operator-overlay.md

## Headless test matrix — CI wiring

- headless_tests.yml workflow in multiplayer-fabric-godot
- 5 branch protection checks (GO, TO, GP, TP, GO+TO)
- Phase 2 GO+TO cross-check

## CI/CD

- Confirm run 24934241572 green
- Wire headless_tests.yml into runner.yml
- taskweft PropCheck NUMTESTS=100 against production CockroachDB

## Branch maintenance

- elixir update_godot_v_sekai.exs once CI green
- multiplayer-fabric-merge dry-run CI job
- Archive feat/multiplayer-fabric
