# TODO

Strategy: get it working locally, then CI/CD keeps it from breaking.

- [ ] Phase 1 GO — `just zone-server-local` then `just go-test`; Elixir zone server compiles, cert generated, observer not yet verified against it
- [ ] Merge zone-backend and zone-console generate-secrets PRs — blocked on required CI: https://github.com/V-Sekai-fire/multiplayer-fabric-zone-backend/pull/1 and https://github.com/V-Sekai-fire/multiplayer-fabric-zone-console/pull/1
- [ ] Wire `headless_tests.yml` — zone-fabric container needs CockroachDB; Godot binary in CI needs custom modules

<!-- Completed items in CHANGELOG.md — deferred items in SOMEDAY.md -->
