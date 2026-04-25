# TODO

Strategy: get it working locally, then CI/CD keeps it from breaking.

- [x] Verify WebTransport reaches UDP 443 (`ZONE_HOST=zone-700a.chibifire.com`)
- [x] `headless_log_observer.gd` — correct port to 7443, add `--dump-json=<path>` flag
- [x] Godot observer and player ADRs — supersede Three.js ADRs

- [ ] Phase 1 GO — run `headless_log_observer.gd` against zone server (`127.0.0.1:7443`), assert `entities > 0`, write Playwright spec
- [ ] Godot player — GDScript CH_PLAYER write path (`send_player_input`), OpenXR presence in `observer.tscn`
- [ ] Wire `headless_tests.yml` — GO branch protection check in `multiplayer-fabric-godot`

<!-- Completed items in CHANGELOG.md — deferred items in SOMEDAY.md -->
