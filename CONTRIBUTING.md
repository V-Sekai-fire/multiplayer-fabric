# Contributing to Multiplayer Fabric

Contribution rules for the whole multiplayer-fabric ecosystem. The submodule list is in [README.md](README.md). Agent-specific workflow (git submodule ops, assembly, test commands) is in [AGENTS.md](AGENTS.md). Each sub-project also has its own `CONTRIBUTING.md` with component-specific details.

## Table of contents

- [Philosophy](#philosophy)
- [Universal rules](#universal-rules)
- [Language-specific guidance](#language-specific-guidance)
- [Documentation style](#documentation-style)
- [Writing anti-tropes](#writing-anti-tropes)
- [Lean-proved invariants](#lean-proved-invariants)
- [Distribution](#distribution)

---

## Philosophy

**Simplify, Then Add Lightness.**

Development speed comes from reducing the mass of the learning loop — ruthlessly deleting unnecessary requirements and pushing complexity into software. When something feels slow, don't ask how to go faster; ask what unnecessary burdens can be dropped.

- Question and subtract requirements. Examine which parts of a spec are not absolutely necessary before implementing.
- Sequence your risks. Early prototypes are scientific experiments designed to retire specific risks in order, not prove everything at once.
- Insource the uncertain. Mature components can be outsourced; core uncertainties stay in-house.
- Shift complexity into software. Replace physical or architectural complexity with computation.
- Compress learning loops. Distance between engineers and the product is a direct tax on speed.
- Maintain organizational lightness. Stay small enough to naturally share context.

---

## Universal rules

### Red-green-refactor

Every feature and every fix is driven by a failing test committed before any implementation code. Validate that the failure message proves the assertion is load-bearing — mutation-test it by briefly breaking the implementation if the message is ambiguous. Commit when green; clean up with tests still green.

One commit per cycle. The TDD arc must be legible in `git log`.

### Error handling

Functions return structured results at every boundary. `raise` / `panic` / `throw` are reserved for programmer errors (wrong argument type, missing config at boot) — never for runtime conditions.

- Elixir: `{:ok, value}` / `{:error, reason}` tuples everywhere. NIF boundary `rescue` blocks must return a typed fallback (`nil`, `[]`, `:ok`) so callers can distinguish.
- Go: return `(value, error)` and propagate `context.Context` through every I/O call.
- C++: return result types or output parameters; no exceptions across module boundaries.

### Credentials never in source

All secrets come from environment variables. Never commit API keys, database passwords, S3 credentials, or tokens. Environment variable names are documented in each sub-project's README.

### Dry-run by default

Every destructive action (upload, tag push, branch reset, remote exec) must support a `--dry-run` flag that prints what would happen without doing it.

### Idempotent operations

Running a pipeline step twice against the same inputs must produce the same result. Upload steps check for an existing artifact before uploading. Never overwrite a published release.

### Commit messages

Sentence case. No `type(scope):` conventional-commit prefixes. Start with a verb in imperative form: `Add`, `Fix`, `Update`, `Remove`. Keep the first line under 72 characters.

```
Add MToon parameter export for hair materials
Fix chunk boundary detection for small files
Update multiplayer_fabric_mmog to 20 Hz default simulation rate
```

---

## Language-specific guidance

### Elixir

- Use PropCheck generators rather than mocks. Generators produce inputs; properties express invariants. If a generator is hard to write, the API surface is too wide.
- Ecto is optional. Behaviour registrations are guarded by `Code.ensure_loaded?`. Libraries must compile and pass NIF tests in projects without Ecto.
- Escript binaries must run on machines with no Elixir installed. Verify with `mix escript.build` on a clean environment before releasing.
- TUI state is pure. Ratatui rendering functions take a model and return a new model; they must not perform I/O. Side effects run in `Task` calls that send messages back to the event loop.
- Migrations are forward-only. Once merged to main, never alter a migration. Fixes require a new migration. Every migration must include a `down/0` that reverts it cleanly.
- `multiplayer-fabric-zone-backend` uses CockroachDB via `Ecto.Adapters.Postgres`. Its Repo must set `migration_lock: false`. The `DATABASE_URL` environment variable is the canonical connection source; never hardcode hostnames or credentials. TLS cert paths come from `CRDB_CA_CERT`, `CRDB_CLIENT_CERT`, `CRDB_CLIENT_KEY`. `aria-storage` has no database — it is a pure chunk-storage library backed by the filesystem and S3.
- NIF boundaries (`multiplayer-fabric-llm`) must schedule all blocking C calls on dirty schedulers (`ERL_NIF_DIRTY_JOB_CPU_BOUND`). Token streaming uses `enif_send` from the dirty scheduler thread — never block the regular scheduler. Resource destructors must be idempotent (null-check before freeing).

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
- `multiplayer-fabric-llm` Makefile mirrors the GPU backend matrix from `turboquant-godot/modules/llm/SCsub` exactly. When turboquant-godot updates its SCsub source lists or backend conditionals, update the Makefile to match. Metal embed generation runs via `mix gen_metal_embed`, not Python.

### Lean 4

- Lean 4 + Batteries only; no Mathlib. Proofs use `omega`, `grind`, `decide`, and hand induction.
- No `sorry` may remain in any proof under `PredictiveBVH/`.
- No `axiom` under `PredictiveBVH/`. All axioms must be declared at the top level only.
- If you find yourself writing `r128_add` / `r128_mul` inline in a string literal in `TreeC.lean`, stop. Extract the polynomial as `Expr Int` and emit it via `genC`.
- Build gate: 313 Lean jobs green from `lake build`; 32 regression test cases / 4347+ assertions green.

---

## Documentation style

All human-facing prose must use Hz, seconds, and metres as public units.

- Durations in seconds or ms: "4 s migration hysteresis", "100 ms latency floor"
- Rates in Hz: "20 Hz default simulation rate"
- Distances in metres: "5 m interest radius"
- Velocities in m/s: "10 m/s velocity cap"
- Accelerations in m/s²

The word "tick" is forbidden in human-facing prose. It survives only in wire-format field names literally named `server_tick` / `player_tick`, in code identifiers when the text refers to the symbol itself, and in one sentence per module's Units section that explains the μm/tick internal encoding.

Tick-rate-dependent quantities must be written parametrically (e.g. `pbvh_latency_ticks(hz)`) with the physical meaning alongside (e.g. "100 ms"). The simulation tick rate is a retargetable implementation choice; public docs should read the same at 20 Hz, 64 Hz, or 120 Hz.

---

## Writing anti-tropes

These patterns weaken documentation regardless of language. Avoid them in all prose, commit messages, and code comments.

Do not start list items with a bolded phrase. That pattern is a formatting tell that makes lists harder to scan. Write the rule as a plain sentence.

Do not restate the same point at every structural level. Each section earns its place by adding information, not by echoing what appeared in the section above.

Drop hollow transitions: "it's worth noting," "importantly," "notably," "it bears mentioning." If information matters, state it. The sentence structure should carry the signal.

Do not inflate the stakes of routine decisions. A commit message convention is not a paradigm shift. A test framework choice is not civilization-defining.

Avoid appending hollow -ing phrases: "highlighting the importance of X," "reflecting broader trends in Y." If there is a point, make it directly.

Do not use "delve," "leverage," "robust," "streamline," "utilize," "harness," "synergy," or "paradigm" anywhere in this repository's documentation.

Do not pose rhetorical questions and immediately answer them. State the point directly.

---

## Lean-proved invariants

The following properties are formally verified in `multiplayer-fabric-godot/modules/multiplayer_fabric_mmog/predictive_bvh/`. They must not be violated by any change to the module.

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
cd multiplayer-fabric-godot/modules/multiplayer_fabric_mmog/predictive_bvh
lake build
lake exe bvh-codegen
```

Never hand-edit `predictive_bvh.h` or `predictive_bvh.rs`.

---

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

---

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
