# TODO

## 1. Unblock CI (do first)

Run 24934241572 in progress — two fixes pushed, static checks ✓.

- [ ] Confirm run 24934241572 green across all platforms
- [ ] Triage and fix any new failures before further feature work

## 2. Security

- [ ] Rotate Cloudflare Turnstile keys — plaintext in `multiplayer-fabric-hosting/.env`

## 3. Demo — Jellyfish Game pass condition

Pass condition (20260425-jellyfish-pass-condition.md): jellyfish appears in VR,
visible to a second player simultaneously, moving under its species domain plan.

- [ ] Verify WebTransport clients reach UDP 443 from public internet
      (`ZONE_HOST=zone-700a.chibifire.com`)
- [ ] Interactive operator camera test: Q/E snap, scroll zoom, WASD pan, F follow, Tab toggle
- [ ] Operator overlay: load bars + dot clustering (20260425-operator-overlay.md)
- [ ] Three.js observer Stage 1: parseInterest() + WebTransport + WebGPU scene

## 4. Headless test matrix

Gate: local Docker → CI headless → VR hardware (20260425-headless-test-matrix.md).

- [ ] `headless_log_observer.gd` — add `--dump-json=<path>` flag
- [ ] Phase 1 GO: Godot observer connects, entity count > 0
- [ ] Phase 1 TO: Three.js observer, `window.__entities.length > 0`
- [ ] Phase 2 GO+TO: same entity IDs from both clients
- [ ] Add `headless_tests.yml` to multiplayer-fabric-godot and wire into `runner.yml`
- [ ] Add 5 branch protection checks (GO, TO, GP, TP, GO+TO)

## 5. taskweft

- [ ] Run PropCheck full suite (NUMTESTS=100) against production CockroachDB

## 6. Branch maintenance

- [ ] Run `elixir update_godot_v_sekai.exs` once all branch CI green
- [ ] `multiplayer-fabric-merge`: add dry-run CI job (`git-assembler --dry-run`)
- [ ] Archive `feat/multiplayer-fabric` once assembled branch is stable

<!-- Completed items in CHANGELOG.md -->
