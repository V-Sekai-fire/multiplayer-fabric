# TODO

Strategy: get it working locally, then CI/CD keeps it from breaking.

- [x] Verify WebTransport reaches UDP 443 (`ZONE_HOST=zone-700a.chibifire.com`)
- [x] `headless_log_observer.gd` — correct port to 7443, add `--dump-json=<path>` flag
- [x] Godot observer and player ADRs — supersede Three.js ADRs

- [ ] Phase 1 GO — run `headless_log_observer.gd` against zone server locally (needs full Docker Compose stack + custom Godot binary)
- [x] Godot player — GDScript CH_PLAYER write path (`send_player_input`), OpenXR presence in `observer.tscn`
- [ ] Wire `headless_tests.yml` — currently broken: zone-fabric needs CockroachDB, Godot binary needs custom modules; headless-tests skipped on PRs anyway

<!-- Completed items in CHANGELOG.md — deferred items in SOMEDAY.md -->
