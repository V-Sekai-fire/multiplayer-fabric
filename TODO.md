# TODO

Strategy: get it working locally, then CI/CD keeps it from breaking.

## Blocking (must do first)

- [ ] **Phase 1 GO** — run `just zone-server-local` then `just go-test`; Elixir zone server compiles and cert symlinks exist but connection not yet verified
- [ ] **GODOT_CPP_BRANCH: 4.5** in `linux_builds.yml` — engine is 4.7.dev; godot-cpp branch mismatch causes every feat branch CI to fail at `Compilation (godot-cpp)`

## Cleanup

- [ ] `multiplayer-fabric-abyssal/sandbox/` — duplicate of `multiplayer-fabric-elf-programs`; remove and add elf-programs as submodule instead
- [ ] Submodule pointers stale: `multiplayer-fabric-hosting` and `multiplayer-fabric-zone-server` (run `/sync-submodule-pointers`)
- [ ] Zone-server cert expires in 14 days — `generate-secrets.sh` must be re-run; add cron reminder
- [ ] Repair `multiplayer-fabric-predictive-bvh-research` modules — Phase 1c (`sorted_is_ascending_after_build`, `aabbQueryB_agrees_with_aabbQueryN`), Phase 2b' (`rayQueryN` / `convexQueryN` soundness), incremental `tick_agrees_with_build`. All 8 modules currently broken under Lean 4.26; not in production codegen import closure

## CI

- [ ] Wire `headless_tests.yml` — ports 7443-7542 are forwarded; Docker zone-fabric Godot process runs but GDScript fails to initialize (no GodotSharp → WebTransport server never starts); use Elixir zone server (`just zone-server-local`) as the CI target instead of the Docker image
- [ ] `elixir update_godot_v_sekai.exs` — reassemble multiplayer-fabric once CI is green

<!-- Completed items in CHANGELOG.md — deferred items in SOMEDAY.md -->
