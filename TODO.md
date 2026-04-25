# TODO

## CI — multiplayer-fabric-godot (assembled branch)

Run 24934241572 in progress — static checks ✓, all builds running.
Two fixes pushed today:
- `6feefaf0b9` — remove `SESSION_H3_SETTINGS` from picoquic backend (missed in audit)
- `99024b02ab` — spelling modelled → modeled (static checks)

- [ ] Confirm run 24934241572 passes all platforms green
- [ ] If any new failures: triage and fix before further feature work

Known feature gaps in http3 (not blocking CI):
- `quic_client.h:157` — poll() stub, picoquic event loop not driven
- `quic_server.h:51` — listen() stub, no UDP bind
- `http3_client.cpp:82` — POST/PUT/DELETE not implemented
- `quic_web_backend.cpp:127` — bidi streams not implemented in web backend

## Demo — Jellyfish Game (Infinite Aquarium)

**Pass condition** (20260425-jellyfish-pass-condition.md): jellyfish appears in VR,
visible to a second player simultaneously, moving under its species domain plan.

**Client strategy**: Godot native PCVR + Three.js WebGPU (no wasm export).
See: 20260425-threejs-observer.md, 20260425-threejs-player.md.

### Operator camera (observer.tscn)

- [x] `operator_camera.gd` written — twist/swing [0,1], Survey/Follow modes
- [x] Input actions added to `project.godot` (Q/E, WASD, F, Tab, scroll)
- [x] `window.__camera_state` JS export via JavaScriptBridge
- [x] Playwright Layer 1 (JS simulation): 7/7 pass
- [x] Headless parse: observer.tscn loads with no script errors
- [ ] Interactive editor test: Q/E snap, scroll zoom, WASD pan, F follow, Tab toggle
- [ ] Operator overlay CanvasLayer: load bars + dot clustering (20260425-operator-overlay.md)

### Three.js WebGPU observer (Stage 1)

- [ ] parseInterest() TypeScript — 100-byte CH_INTEREST wire format
- [ ] WebTransport connection to zone server (browser API)
- [ ] Three.js WebGPU scene — OrthographicCamera at twist/swing SWING_ELEVATION
- [ ] Operator overlay canvas (load bars, dot clustering)

### Headless test matrix (20260425-headless-test-matrix.md)

Gate: local Docker → CI headless → VR hardware.

- [ ] `headless_log_observer.gd` — add `--dump-json=<path>` flag
- [ ] Phase 1 GO: Godot observer connects, entity count > 0
- [ ] Phase 1 TO: Three.js observer, `window.__entities.length > 0`
- [ ] Phase 2 GO+TO: same entity IDs from both clients

### Build pipeline (20260425-build-cache-test-pipeline.md)

- [ ] Add `headless_tests.yml` workflow to multiplayer-fabric-godot
- [ ] Wire as 4th stage in `runner.yml` after `docker-images`
- [ ] Add 5 branch protection status checks (GO, TO, GP, TP, GO+TO)

## taskweft — PropCheck tests

All 67 property tests green (confirmed 2026-04-25, NUMTESTS=5).
Fixes landed: query_all nil-rows, module-level pool, cert path, Postgrex.rollback.

- [ ] Run full suite (NUMTESTS=100) against production CockroachDB to confirm

## Zone backend / cluster

- [ ] Verify WebTransport clients reach UDP 443 from public internet
      (`ZONE_HOST=zone-700a.chibifire.com` in `.env`)
- [ ] Rotate Cloudflare Turnstile keys
      (`multiplayer-fabric-hosting/.env` has plaintext `TURNSTILE_SECRET_KEY`)

## Branch maintenance

- [ ] Archive `feat/multiplayer-fabric` once assembled branch is stable
- [ ] `multiplayer-fabric-merge`: add dry-run CI job (`git-assembler --dry-run`)
- [ ] Run `elixir update_godot_v_sekai.exs` once all branch CI green

## Web client PoC (superseded — history)

- [x] `wt_browser.spec.ts` PASS 1.4 s
- [x] `godot_web_init.spec.ts` PASS 1.1 s
- [x] `godot_wt_e2e.spec.ts` PASS 3.3 s (4 C++ bugs fixed)
- [x] Zone backend `/api/v1` migration — 6/6 QA green
- [x] WebTransport audit: datagram reader lock, incoming mutex, dead state, Lean proofs
- [x] taskweft: query_all nil-rows fix, TLS test helpers, module-level pool, rollback fix
