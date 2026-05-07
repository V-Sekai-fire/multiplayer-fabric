# AGENTS.md

Guidance for everyone — human contributors and AI coding agents — working in this repository. Each submodule has its own `AGENTS.md` with component-specific details that this file does not duplicate.

The full submodule list lives at <https://v-sekai-fire.github.io/manuals/> and in [`manuals/index.md`](manuals/index.md).

## Philosophy

**Simplify, then add lightness.**

Development speed comes from reducing the mass of the learning loop — ruthlessly deleting unnecessary requirements and pushing complexity into software. When something feels slow, don't ask how to go faster; ask what unnecessary burdens can be dropped.

- Question and subtract requirements. Examine which parts of a spec are not absolutely necessary before implementing.
- Sequence your risks. Early prototypes are scientific experiments designed to retire specific risks in order, not prove everything at once.
- Insource the uncertain. Mature components can be outsourced; core uncertainties stay in-house.
- Shift complexity into software. Replace physical or architectural complexity with computation.
- Compress learning loops. Distance between engineers and the product is a direct tax on speed.
- Maintain organizational lightness. Stay small enough to naturally share context.

## Operating system

Develop on macOS or Linux. **On Windows, use WSL2 (Ubuntu)** — the local workflow is bash-first (POSIX shebangs, `/tmp` paths, symlinks, `lsof`, UNIX docker socket). cmd / PowerShell are not supported.

## Working with submodules

```sh
# Clone everything
git clone --recurse-submodules https://github.com/V-Sekai-fire/multiplayer-fabric
cd multiplayer-fabric

# After pulling root updates, sync submodule pointers
git submodule update --init --recursive

# Work inside a submodule as a normal git repo
cd taskweft
git checkout -b my-branch
# ... make changes, commit, push ...
cd ..

# Record the updated submodule pointer in the root repo
git add taskweft
git commit -m "Sync submodules: taskweft add streaming API"
```

Never commit changes directly to files inside a submodule from the root repo. Always `cd` into the submodule first.

## Universal rules

### Red-green-refactor

Every feature and every fix is driven by a failing test committed before any implementation code. Validate that the failure message proves the assertion is load-bearing — mutation-test it by briefly breaking the implementation if the message is ambiguous. Commit when green; clean up with tests still green. One commit per cycle. The TDD arc must be legible in `git log`.

### Error handling

Functions return structured results at every boundary. `raise` / `panic` / `throw` are reserved for programmer errors (wrong argument type, missing config at boot) — never for runtime conditions.

- Elixir: `{:ok, value}` / `{:error, reason}` tuples everywhere. NIF boundary `rescue` blocks must return a typed fallback (`nil`, `[]`, `:ok`) so callers can distinguish.
- Go: return `(value, error)` and propagate `context.Context` through every I/O call.
- C++: return result types or output parameters; no exceptions across module boundaries.

### Credentials never in source

All secrets come from environment variables. Never commit API keys, database passwords, S3 credentials, or tokens. Environment variable names are documented in each sub-project's `AGENTS.md`.

### Dry-run by default

Every destructive action (upload, tag push, branch reset, remote exec) must support a `--dry-run` flag that prints what would happen without doing it.

### Idempotent operations

Running a pipeline step twice against the same inputs must produce the same result. Upload steps check for an existing artifact before uploading. Never overwrite a published release.

## Commit message style

MUST NOT use Conventional Commits (`feat:`, `fix:`, `chore:`, etc.). Write commit messages in sentence case: one short imperative sentence, no type prefix, no scope, no colon. Keep the first line under 72 characters.

```
Add CMD_INSTANCE_ASSET to peer command enum
Fix mix.exs missing closing bracket in deps list
Update zone-console submodule pointer
Add MToon parameter export for hair materials
```

## Documentation style

All human-facing prose must use Hz, seconds, and metres as public units.

- Durations in seconds or ms: "4 s migration hysteresis", "100 ms latency floor"
- Rates in Hz: "20 Hz default simulation rate"
- Distances in metres: "5 m interest radius"
- Velocities in m/s: "10 m/s velocity cap"
- Accelerations in m/s²

The word "tick" is forbidden in human-facing prose. It survives only in wire-format field names literally named `server_tick` / `player_tick`, in code identifiers when the text refers to the symbol itself, and in one sentence per module's Units section that explains the μm/tick internal encoding.

Tick-rate-dependent quantities must be written parametrically (e.g. `pbvh_latency_ticks(hz)`) with the physical meaning alongside (e.g. "100 ms"). The simulation tick rate is a retargetable implementation choice; public docs should read the same at 20 Hz, 64 Hz, or 120 Hz.

## Writing anti-tropes

These patterns weaken documentation regardless of language. Avoid them in all prose, commit messages, and code comments.

Do not start list items with a bolded phrase. That pattern is a formatting tell that makes lists harder to scan. Write the rule as a plain sentence.

Do not restate the same point at every structural level. Each section earns its place by adding information, not by echoing what appeared in the section above.

Drop hollow transitions: "it's worth noting," "importantly," "notably," "it bears mentioning." If information matters, state it. The sentence structure should carry the signal.

Do not inflate the stakes of routine decisions. A commit message convention is not a paradigm shift. A test framework choice is not civilization-defining.

Avoid appending hollow -ing phrases: "highlighting the importance of X," "reflecting broader trends in Y." If there is a point, make it directly.

Do not use "delve," "leverage," "robust," "streamline," "utilize," "harness," "synergy," or "paradigm" anywhere in this repository's documentation.

Do not pose rhetorical questions and immediately answer them. State the point directly.

## Language-specific guidance

### Elixir

- Use PropCheck generators rather than mocks. Generators produce inputs; properties express invariants. If a generator is hard to write, the API surface is too wide.
- Ecto is optional. Behaviour registrations are guarded by `Code.ensure_loaded?`. Libraries must compile and pass NIF tests in projects without Ecto.
- Escript binaries must run on machines with no Elixir installed. Verify with `mix escript.build` on a clean environment before releasing.
- TUI state is pure. Ratatui rendering functions take a model and return a new model; they must not perform I/O. Side effects run in `Task` calls that send messages back to the event loop.
- Migrations are forward-only. Once merged to main, never alter a migration. Fixes require a new migration. Every migration must include a `down/0` that reverts it cleanly.
- `zone-backend` (uro) uses CockroachDB via `Ecto.Adapters.Postgres`. Its Repo must set `migration_lock: false`. The `DATABASE_URL` environment variable is the canonical connection source; never hardcode hostnames or credentials. TLS cert paths come from `CRDB_CA_CERT`, `CRDB_CLIENT_CERT`, `CRDB_CLIENT_KEY`. `aria-storage` has no database — it is a pure chunk-storage library backed by the filesystem and S3.
- NIF boundaries (`llm`) must schedule all blocking C calls on dirty schedulers (`ERL_NIF_DIRTY_JOB_CPU_BOUND`). Token streaming uses `enif_send` from the dirty scheduler thread — never block the regular scheduler. Resource destructors must be idempotent (null-check before freeing).

### Go

- New store backends implement a minimal interface. Do not add methods to an interface unless every existing implementation needs them.
- Options are passed explicitly. `init()` blocks that modify global state are not permitted outside `cmd/`.
- Every function performing I/O accepts `context.Context` as its first argument.
- `gofmt -l .` must produce no output before committing.

### Godot / GDScript

- Validate changes by opening the project in the matching Godot editor build. Headless checks (`godot --headless --quit`) are preferred for CI.
- Each addon under `addons/` must function independently. No cross-addon `preload` calls; use signals or autoloads for cross-cutting concerns.
- Addons must not register global autoloads. Consumers instantiate what they need and wire signals themselves.
- Interaction state changes emit signals. Do not poll state in `_process`.
- Client-side prediction is permitted for visual smoothing only. Gameplay state must always be confirmed by the server before being treated as canonical.
- Commit only `.gd`, `.tres`, `.tscn`, `.gdshader`, and `.import` metadata. Large binary assets are distributed via the artifact system.
- Microphone capture must never be activated without explicit user action.

### C++ (native modules and sandbox kernels)

- `-fsanitize=address,undefined` runs on every Debug build. An ASAN or UBSAN finding is a RED, not a warning.
- Use `std::array`, `std::span`, or a bump allocator in inner loops. No dynamic allocation in hot paths.
- All SIMD optimizations must be guarded by a compile-time feature flag with a scalar reference implementation tested independently.
- Never hand-edit `predictive_bvh.h` or `predictive_bvh.rs` — regenerate with `lake exe bvh-codegen`.
- `llm` Makefile mirrors the GPU backend matrix from `turboquant-godot/modules/llm/SCsub` exactly. When turboquant-godot updates its SCsub source lists or backend conditionals, update the Makefile to match. Metal embed generation runs via `mix gen_metal_embed`, not Python.

### Lean 4

- Lean 4 + Batteries only; no Mathlib. Proofs use `omega`, `grind`, `decide`, and hand induction.
- No `sorry` may remain in any proof under `PredictiveBVH/`.
- No `axiom` under `PredictiveBVH/`. All axioms must be declared at the top level only.
- If you find yourself writing `r128_add` / `r128_mul` inline in a string literal in `TreeC.lean`, stop. Extract the polynomial as `Expr Int` and emit it via `genC`.
- Build gate: 313 Lean jobs green from `lake build`; 32 regression test cases / 4347+ assertions green.

## Mathematics and proof authority

`predictive-bvh` is the canonical mathematical authority for all physics, geometry, and algorithmic proofs across the multiplayer fabric. It contains:

- Formal Lean 4 proofs of core algorithms (Hilbert curves, BVH construction, interest management)
- O(1) per-entity complexity theorems for MMO scaling
- Geometric stability proofs for adversarial physics scenarios
- Rate-distortion optimization bounds for bandwidth / latency tradeoffs

Integration rules:

- When porting algorithms to Elixir, C++, or other languages, the Lean formalization in `predictive-bvh` is the source of truth.
- Port implementations from proof-verified code, not from implementations in other languages.
- If an algorithm differs from the Lean proof, trust the proof; fix the implementation.
- Add new proofs to `predictive-bvh` before implementing in production code.

Examples:

- 3D Hilbert curve: formalized in `predictive-bvh/PredictiveBVH.lean`; Elixir port in `multiplayer-fabric-deploy/lib/multiplayer_fabric_deploy/hilbert_curve.ex`
- Interest management: formalized via ReBAC + Hilbert bounds; consumed by zone-console and zone servers
- BVH structure: Lean proof of O(1) query time; C++ implementation in godot-mmog module

## Lean-proved invariants

The following properties are formally verified in `godot/modules/multiplayer_fabric_mmog/predictive_bvh/`. They must not be violated by any change to the module.

| Invariant | Location |
|-----------|----------|
| `owned_to_staging` — entity transitions from OWNED to STAGING | `Protocol/Fabric.lean` |
| `staging_resolves_to_single_owner` — exactly one owner after commit | `Protocol/Fabric.lean` |
| `staging_plus_aborted` — timeout rollback restores original owner | `Protocol/Fabric.lean` |
| `expansion_covers_k_ticks` — ghost AABB covers all positions within δ seconds | `Spatial/Tree.lean` |
| `surfaceArea_nonneg` — SAH formula soundness | `Formulas/Formula.lean` |
| Bucket bound — `bmax ≤ 2 · PBVH_BUCKET_K_TARGET` at every built N | `Spatial/BucketBound.lean` |
| Hilbert forward/inverse bijection | `Spatial/HilbertRoundtrip.lean` |

Regenerate the emitted C after any proof change:

```bash
cd godot/modules/multiplayer_fabric_mmog/predictive_bvh
lake build
lake exe bvh-codegen
```

Never hand-edit `predictive_bvh.h` or `predictive_bvh.rs`.

## Maglev cycle workflow

The Maglev cycles are a 13-step end-to-end validation of the multiplayer fabric stack. ADRs and pass criteria live in [`manuals/decisions/20260506-maglev-cycle-*.md`](https://v-sekai-fire.github.io/manuals/decisions.html). Status (as of 2026-05-07): C0 ✅ pass, C1 ✅ pass, C2 ◐ producer-side complete, infra verified; C3+ pending.

### Verifying live Fly state without local auth

`flyctl auth login` is not required locally — verification runs as a workflow inside CI where `FLY_API_TOKEN` is the org-level secret. Two read-only workflows on `infra`:

```sh
gh workflow run verify_fly_state.yml --repo V-Sekai-fire/multiplayer-fabric-infra
gh run list --repo V-Sekai-fire/multiplayer-fabric-infra --workflow verify_fly_state.yml --limit 1
gh run download <id> --repo V-Sekai-fire/multiplayer-fabric-infra --name fly-state -D /tmp/fly-state
```

The artifact (`fly-state.json`) holds `flyctl status / volumes list / secrets list / ips list` for the four cycle apps (gateway, crdb, zone-backend, observability).

If apps come back `suspended`, wake them with `start_fly_apps.yml`; if they have 0 machines, trigger a fresh deploy from each app's own repo.

For OTel verification: `verify_observability.yml` opens a `flyctl proxy` per Victoria* port and curls the queries — see [`.github/workflows/verify_observability.yml`](https://github.com/V-Sekai-fire/multiplayer-fabric-infra/blob/main/.github/workflows/verify_observability.yml).

### Running cycle smoke tests

Cycle test scripts live in [`cycle-tests`](https://github.com/V-Sekai-fire/multiplayer-fabric-cycle-tests). Each is a minimal headless Godot `SceneTree` that exits 0 on PASS, non-zero on FAIL.

```sh
# Build the engine (cache-warm: ~30 s on M-series Mac)
cd merge && git checkout multiplayer-fabric-base
gscons   # alias for the canonical macOS editor build

# Run cycle 1 against live infrastructure
godot --headless --script ../cycle-tests/cycle-1-gateway-handshake/cycle1.gd
```

Currently `gscons` produces `bin/godot.macos.editor.dev.double.arm64`. `gmscons` does the Windows mingw cross-compile; `gescons` does web/wasm32. All three were verified end-to-end against the assembled engine on 2026-05-07.

### Engine assembly (merge)

The engine is composed from feature branches at build time. The recipe is `merge/gitassembly`; the driver is `merge/update_godot_v_sekai.exs` which:

1. Adds remotes (`v-sekai-fire/multiplayer-fabric-godot` + `opentelemetry-godot/feat/open-telemetry-base`).
2. Runs `git-assembler --recreate` to merge each feature branch into a fresh `multiplayer-fabric-base`.
3. Force-pushes the result to upstream.

```sh
cd merge
elixir update_godot_v_sekai.exs --dry-run   # validate the recipe assembles cleanly
elixir update_godot_v_sekai.exs             # real run; force-pushes assembled branches
```

Hard rules:

- Never use `git rerere` in `merge`. Conflict resolutions there are local-only and won't reproduce on another machine. Fix the underlying overlap on the source feature branches instead — orthogonality across branches is the contract.
- Each feature branch owns its files outright. No two branches in the recipe should add the same file with different content. Today's split of `feat/engine-patches` into 10 single-topic branches enforces this.
- The build repo (`build/godot/`) tracks `multiplayer-fabric-godot-maglev/multiplayer-fabric-base` via `git subrepo`. The maglev fork is a publish target for the assembled engine; never push feature branches there.

## Per-submodule test commands

| Submodule | Test command | Framework |
|-----------|-------------|-----------|
| `aria-storage` | `mix test` | ExUnit + PropCheck |
| `llm` | `mix test` | ExUnit (elixir_make builds NIF automatically) |
| `taskweft` | `mix test --include property` | ExUnit + PropCheck |
| `zone-backend` (uro) | `mix test` | ExUnit |
| `zone-console` | `mix test` | ExUnit + PropCheck |

## Critical cross-submodule relationships

`taskweft` is a library dependency of the artifact CLI and the zone console. A breaking API change in taskweft requires updates in both consumers before either is released.

## Verbalized Sampling (VS)

When facing an open-ended decision — which branch to create, how to resolve a conflict, what name to give a new submodule — use Verbalized Sampling (Zhang et al. 2025, `references.bib`) instead of committing to the first option that comes to mind.

Technique: generate N candidate responses and assign an explicit probability to each. Pick the highest-probability response, or surface the distribution to the user when probabilities are close. This counters typicality bias: the first response an LLM produces is the most familiar, not necessarily the best.

Example — choosing a name for a new skills submodule:

| Response | P |
|---|---|
| `skills` | 0.35 |
| `playbook` | 0.20 |
| `sop` | 0.15 |
| `runbook` | 0.12 |
| `recipes` | 0.08 |
| `tactics` | 0.05 |
| `procedures` | 0.03 |
| `ops` | 0.02 |

Chosen: `skills` (0.35) — highest probability, domain-neutral, consistent with the Elixir convention of naming behaviour units "skills".

How we use it in this project: before taking any irreversible action (creating a GitHub repo, naming a branch, choosing a conflict resolution strategy), produce a VS table in your reasoning. Do not show the table to the user unless probabilities are close (top two within 0.10 of each other) — in that case, surface the top two and ask. Otherwise silently pick the highest and proceed.

The table must have exactly 8 rows. Probabilities must sum to 1.00. Assign probability 0.00 to options that violate project rules (e.g. any name starting with `archived/`, any option that requires force-pushing a protected branch).

When to use VS:

- Naming a new repo, branch, or file
- Resolving an assembly conflict with multiple valid fixes
- Deciding which SOP to write next
- Choosing between two architectural approaches

Reference: `references.bib` → `zhang2025verbalized`.

## Work queue discipline

Primary definition: `taskweft/priv/plans/domains/work_queue.jsonld`. Schedule proof: `lean4-predictive-bvh-proofs/WorkQueue.lean` (`workQueue_scheduleValid`).

One Saturday per week, single entity (ifire). Each item advances stub → green → refactor. Green phase includes writing the Lean proof for that item. Demo: 2026-08-29.

```
 1. openxr_simulator     May 02 → May 16   XR   critical path
 2. vr_interaction       May 23 → Jun 06   XR   critical path
 3. pcvr_player          Jun 13 → Jun 27   MMOG critical path
 4. zone_manifest        Jul 04 → Jul 18   MMOG demo gate
 5. sandbox_rebac        Jul 25 → Aug 08   MMOG demo gate
 6. jellyfish_demo       Aug 15 → Aug 29   MMOG ← demo
 7. isolated_submodule   Sep 05 → Sep 19   MMOG CI
 8. headless_observer    Sep 26 → Oct 10   MMOG CI
 9. headless_matrix      Oct 17 → Oct 31   MMOG CI
10. turboquant_llm       Nov 07 → Nov 21   RPG
11. artifactsmmog_bot    Nov 28 → Dec 12   RPG
12. gepa_cycle           Dec 19 → Jan 02   RPG
13. concert_archetype    Jan 09 → Jan 23   MMOG archetype
14. ragdoll_archetype    Jan 30 → Feb 13   MMOG archetype
15. convoy_chokepoint    Feb 20 → Mar 06   MMOG archetype
16. otel_tracing         Mar 13 → Mar 27   XR   independent
```

To regenerate the plan:

```sh
cd taskweft && mix run -e '
domain = File.read!("priv/plans/domains/work_queue.jsonld")
{:ok, plan} = Taskweft.plan(domain)
Jason.decode!(plan) |> Enum.each(&IO.inspect/1)
'
```

## Distribution

### casync / desync asset distribution (aria-storage)

Game assets are distributed as casync chunk stores. The canonical chunk store for V-Sekai game builds is `https://raw.githubusercontent.com/V-Sekai/casync-v-sekai-game/main/store`. Index files (`.caibx` / `.caidx`) live alongside.

- Chunks are content-addressed (SHA-512/256). A chunk never changes after upload — only new chunks are added for new content.
- The local chunk cache lives at `~/.cache/casync/chunks` (XDG on Linux, `~/Library/Caches/casync/chunks` on macOS, `%LOCALAPPDATA%\casync\chunks` on Windows) and is shared between `aria-storage`, `desync`, and `casync`. Do not change this path without updating all three.
- `mix aria_storage.fetch --index <url> --store <url>` reassembles an asset. Add `--cache <path>` to override the default cache location.
- Chunk store layout: `{store}/{hex[0:4]}/{hex}.cacnk`. Flat layouts are not interoperable.

### Homebrew (macOS)

Formulae live in `homebrew-multiplayer-fabric/Formula/`. For each version bump:

1. Download the release archive and compute `shasum -a 256 <tarball>`.
2. Update `url` and `sha256` in the formula. Remove any `revision` line unless a formula-level patch was reapplied.
3. Run `brew audit --strict Formula/<name>.rb` and `brew install --build-from-source Formula/<name>.rb`.
4. Commit as `Bump <name> to <version>`.

Do not add or modify `bottle do` blocks manually — bottles are generated by CI.

### Scoop (Windows)

Manifests live in `scoop-multiplayer-fabric/bucket/`. For each version bump:

1. Download the Windows archive and compute `(Get-FileHash <file> -Algorithm SHA256).Hash.ToLower()`.
2. Update `version`, `url`, and `hash` in the manifest.
3. Run `scoop install bucket/<name>.json` to confirm install and uninstall cleanly.
4. Commit as `Bump <name> to <version>`.

Every manifest must include `checkver` and `autoupdate` pointed at the GitHub releases API.

## Not building

These were explicitly superseded or rejected. Do not reopen without an ADR.

| Feature | Reason |
|---------|--------|
| Three.js WebGPU zone client | Superseded by Godot native client |
| Three.js observer (Stage 1) | Superseded by Godot headless observer |
| Three.js player (Stage 2) | Superseded by Godot PCVR player |
| Dual-client Playwright test | Superseded by headless test matrix (GO+GP) |
| Godot wasm32/wasm64 web export | Dropped |
| SQL-based feature flagging | Rejected |
| Nutanix/Harvester HCI evaluation | Deferred |

## multiplayer-fabric-artifacts-mmog

An Elixir HTN-planning bot for the ArtifactsMMO game. It calls the game API via `Req`, builds a Taskweft domain JSON from live character state, plans one episode, and executes the resulting action sequence.

### Running the bot

```sh
cd multiplayer-fabric-artifacts-mmog

# List available goals
mix artifacts_mmog.goals

# Run a goal loop (Ctrl-C to stop)
ARTIFACTS_MMOG_KEY=<token> mix artifacts_mmog.run <CharName> <goal>

# Run a fixed number of iterations
ARTIFACTS_MMOG_KEY=<token> mix artifacts_mmog.run <CharName> fight_chickens 10
```

### Available goals

| Goal | Action |
|---|---|
| `farm_copper` / `farm_iron` / `farm_coal` | Gather from ore nodes |
| `farm_ash` / `farm_birch` / `farm_spruce` / `farm_sunflowers` | Gather from resource nodes |
| `fight_chickens` / `fight_pigs` / `fight_goblins` / `fight_wolverines` | Fight monsters |
| `fish_gudgeon` | Fish at gudgeon spot |
| `task_cycle` | Accept or complete the active task |
| `rest_at_bank` | Go to bank and rest to full HP |

### Formatting

A `.formatter.exs` is present. Always run `mix format` before committing and ensure `mix format --check-formatted` passes (the CI gate checks this).

### adventurer.jsonld — baseline persona plan

`priv/plans/personas/adventurer.jsonld` is a standalone JSON-LD domain that mirrors the actions and methods produced by `ArtifactsMmog.Domain.build/2`. When `domain.ex` changes, this file must be kept in sync:

- Action op syntax must match: `domain.ex` emits `"op": "add"` / `"op": "get"` — not the old `"type": "math/add"` / `"type": "pointer/get"`.
- Method set must be complete: every method in `domain.ex` (`go_to_bank`, `ensure_rested`, `bank_if_full`, `farm_resources`, `fight_monsters`, `rest_at_bank`, `task_cycle`) must appear in `adventurer.jsonld`.
- Zone enum and IDs must match the `@zones` list order in `domain.ex` (bank=0 … task_master=13).
