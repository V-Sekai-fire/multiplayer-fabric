# AGENTS.md

Guidance for AI coding agents working in this repository.

The submodule list is the canonical source of truth in [README.md](README.md).
Contribution rules, language guides, documentation style, and writing anti-tropes are in [CONTRIBUTING.md](CONTRIBUTING.md). This file covers only what is specific to agent workflows.

## Commit message style

MUST NOT use Conventional Commits (`feat:`, `fix:`, `chore:`, etc.). Write commit messages in sentence case: one short imperative sentence, no type prefix, no scope, no colon. Examples:

- `Add CMD_INSTANCE_ASSET to peer command enum`
- `Fix mix.exs missing closing bracket in deps list`
- `Update zone-console submodule pointer`

## Operating system

Develop on macOS or Linux. **On Windows, use WSL2 (Ubuntu)** — the local
workflow is bash-first (POSIX shebangs, `/tmp` paths, symlinks, `lsof`,
UNIX docker socket). cmd / PowerShell are not supported.

## Working with submodules

```sh
# Clone everything
git clone --recurse-submodules https://github.com/V-Sekai-fire/multiplayer-fabric
cd multiplayer-fabric

# After pulling root updates, sync submodule pointers
git submodule update --init --recursive

# Work inside a submodule as a normal git repo
cd multiplayer-fabric-taskweft
git checkout -b my-branch
# ... make changes, commit, push ...
cd ..

# Record the updated submodule pointer in the root repo
git add multiplayer-fabric-taskweft
git commit -m "Sync submodules: taskweft add streaming API"
```

Never commit changes directly to files inside a submodule from the root repo. Always `cd` into the submodule first. Each submodule has its own `CONTRIBUTING.md` — read it before making changes.
## Work Queue Discipline

Primary definition: `multiplayer-fabric-taskweft/priv/plans/domains/work_queue.jsonld`
Schedule proof: `lean4-predictive-bvh-proofs/WorkQueue.lean` (`workQueue_scheduleValid`)

One Saturday per week, single entity (ifire). Each item advances stub → green → refactor.
Green phase includes writing the Lean proof for that item. Demo: **2026-08-29**.

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

To regenerate the plan: `cd multiplayer-fabric-taskweft && mix run -e '
domain = File.read!("priv/plans/domains/work_queue.jsonld")
{:ok, plan} = Taskweft.plan(domain)
Jason.decode!(plan) |> Enum.each(&IO.inspect/1)
'`

## Mathematics and Proof Authority

### multiplayer-fabric-predictive-bvh — primary proof source

`multiplayer-fabric-predictive-bvh` is the **canonical mathematical authority** for all physics, geometry, and algorithmic proofs across the multiplayer fabric. It contains:

- **Formal Lean 4 proofs** of core algorithms (Hilbert curves, BVH construction, interest management)
- **O(1) per-entity complexity theorems** for MMO scaling
- **Geometric stability proofs** for adversarial physics scenarios
- **Rate-distortion optimization** bounds for bandwidth / latency tradeoffs

**Integration rules:**
- When porting algorithms to Elixir, C++, or other languages, the Lean formalization in `multiplayer-fabric-predictive-bvh` is the source of truth
- Port implementations from **proof-verified code**, not from implementations in other languages
- If an algorithm differs from the Lean proof, trust the proof; fix the implementation
- Add new proofs to `multiplayer-fabric-predictive-bvh` before implementing in production code

**Examples:**
- 3D Hilbert curve: formalized in `multiplayer-fabric-predictive-bvh/PredictiveBVH.lean`; Elixir port in `multiplayer-fabric-deploy/lib/multiplayer_fabric_deploy/hilbert_curve.ex`
- Interest management: formalized via ReBAC + Hilbert bounds; consumed by zone-console and zone servers
- BVH structure: Lean proof of O(1) query time; C++ implementation in godot-mmog module

Location: `multiplayer-fabric-predictive-bvh/` (submodule in this repo)

## Critical cross-submodule relationships

### multiplayer-fabric-taskweft → multiplayer-fabric-artifacts-mmog / zone-console

`multiplayer-fabric-taskweft` is a library dependency of both the artifact CLI and the zone console. A breaking API change in taskweft requires updates in both consumers before either is released.

## Verbalized Sampling (VS)

When facing an open-ended decision — which branch to create, how to resolve a
conflict, what name to give a new submodule — use Verbalized Sampling
(Zhang et al. 2025, `references.bib`) instead of committing to the first
option that comes to mind.

**Technique:** Generate N candidate responses and assign an explicit probability
to each. Pick the highest-probability response, or surface the distribution to
the user when probabilities are close.

This counters typicality bias: the first response an LLM produces is the most
familiar, not necessarily the best.

**Example — choosing a name for a new skills submodule:**

| Response | P |
|---|---|
| `multiplayer-fabric-skills` | 0.35 |
| `multiplayer-fabric-playbook` | 0.20 |
| `multiplayer-fabric-sop` | 0.15 |
| `multiplayer-fabric-runbook` | 0.12 |
| `multiplayer-fabric-recipes` | 0.08 |
| `multiplayer-fabric-tactics` | 0.05 |
| `multiplayer-fabric-procedures` | 0.03 |
| `multiplayer-fabric-ops` | 0.02 |

Chosen: `multiplayer-fabric-skills` (0.35) — highest probability, domain-neutral,
consistent with the Elixir convention of naming behaviour units "skills".

**How we use it in this project:**

Before taking any irreversible action (creating a GitHub repo, naming a branch,
choosing a conflict resolution strategy), produce a VS table in your reasoning.
Do not show the table to the user unless probabilities are close (top two within
0.10 of each other) — in that case, surface the top two and ask. Otherwise
silently pick the highest and proceed.

The table must have exactly 8 rows. Probabilities must sum to 1.00.
Assign probability 0.00 to options that violate project rules (e.g. any name
starting with `archived/`, any option that requires force-pushing a protected
branch).

**When to use VS:**

- Naming a new repo, branch, or file
- Resolving an assembly conflict with multiple valid fixes
- Deciding which SOP to write next
- Choosing between two architectural approaches

**Reference:** `references.bib` → `zhang2025verbalized`

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

- **Action op syntax** must match: `domain.ex` emits `"op": "add"` / `"op": "get"` — not the old `"type": "math/add"` / `"type": "pointer/get"`.
- **Method set** must be complete: every method in `domain.ex` (`go_to_bank`, `ensure_rested`, `bank_if_full`, `farm_resources`, `fight_monsters`, `rest_at_bank`, `task_cycle`) must appear in `adventurer.jsonld`.
- **Zone enum and IDs** must match the `@zones` list order in `domain.ex` (bank=0 … task_master=13).

## Maglev cycle workflow

The Maglev cycles are a 13-step end-to-end validation of the multiplayer fabric stack. ADRs and pass criteria live in [`multiplayer-fabric-manuals/decisions/20260506-maglev-cycle-*.md`](https://v-sekai-fire.github.io/manuals/decisions.html). Status (as of 2026-05-07): C0 ✅ pass, C1 ✅ pass, C2 ◐ producer-side complete, infra verified; C3+ pending.

### Verifying live Fly state without local auth

`flyctl auth login` is not required locally — verification runs as a workflow inside CI where `FLY_API_TOKEN` is the org-level secret. Two read-only workflows on `multiplayer-fabric-infra`:

```sh
gh workflow run verify_fly_state.yml --repo V-Sekai-fire/multiplayer-fabric-infra
gh run list --repo V-Sekai-fire/multiplayer-fabric-infra --workflow verify_fly_state.yml --limit 1
gh run download <id> --repo V-Sekai-fire/multiplayer-fabric-infra --name fly-state -D /tmp/fly-state
```

The artifact (`fly-state.json`) holds `flyctl status / volumes list / secrets list / ips list` for the four cycle apps (gateway, crdb, zone-backend, observability).

If apps come back `suspended`, wake them with `start_fly_apps.yml`; if they have 0 machines, trigger a fresh deploy from each app's own repo.

For OTel verification: `verify_observability.yml` opens a `flyctl proxy` per Victoria* port and curls the queries — see [`.github/workflows/verify_observability.yml`](https://github.com/V-Sekai-fire/multiplayer-fabric-infra/blob/main/.github/workflows/verify_observability.yml).

### Running cycle smoke tests

Cycle test scripts live in [`multiplayer-fabric-cycle-tests`](https://github.com/V-Sekai-fire/multiplayer-fabric-cycle-tests). Each is a minimal headless Godot `SceneTree` that exits 0 on PASS, non-zero on FAIL.

```sh
# Build the engine (cache-warm: ~30 s on M-series Mac)
cd multiplayer-fabric-merge && git checkout multiplayer-fabric-base
gscons   # alias for the canonical macOS editor build

# Run cycle 1 against live infrastructure
godot --headless --script ../multiplayer-fabric-cycle-tests/cycle-1-gateway-handshake/cycle1.gd
```

Currently `gscons` produces `bin/godot.macos.editor.dev.double.arm64`. `gmscons` does the Windows mingw cross-compile; `gescons` does web/wasm32. All three were verified end-to-end against the assembled engine on 2026-05-07.

### Engine assembly (multiplayer-fabric-merge)

The engine is composed from feature branches at build time. The recipe is `multiplayer-fabric-merge/gitassembly`; the driver is `multiplayer-fabric-merge/update_godot_v_sekai.exs` which:

1. Adds remotes (`v-sekai-fire/multiplayer-fabric-godot` + `opentelemetry-godot/feat/open-telemetry-base`).
2. Runs `git-assembler --recreate` to merge each feature branch into a fresh `multiplayer-fabric-base`.
3. Force-pushes the result to upstream.

```sh
cd multiplayer-fabric-merge
elixir update_godot_v_sekai.exs --dry-run   # validate the recipe assembles cleanly
elixir update_godot_v_sekai.exs             # real run; force-pushes assembled branches
```

Hard rules:

- **Never use `git rerere`** in `multiplayer-fabric-merge`. Conflict resolutions there are local-only and won't reproduce on another machine. Fix the underlying overlap on the source feature branches instead (orthogonality across branches is the contract).
- **Each feature branch owns its files outright.** No two branches in the recipe should add the same file with different content. Today's split of `feat/engine-patches` into 10 single-topic branches enforces this.
- The build repo (`multiplayer-fabric-build/godot/`) tracks `multiplayer-fabric-godot-maglev/multiplayer-fabric-base` via `git subrepo`. The maglev fork is a publish target for the assembled engine; never push feature branches there.

## Per-submodule test commands

| Submodule | Test command | Framework |
|-----------|-------------|-----------|
| `aria-storage` | `mix test` | ExUnit + PropCheck |
| `multiplayer-fabric-artifacts-mmog` | `mix test` | ExUnit |
| `multiplayer-fabric-deploy` | `mix test` | ExUnit |
| `multiplayer-fabric-llm` | `mix test` | ExUnit (elixir_make builds NIF automatically) |
| `multiplayer-fabric-taskweft` | `mix test --include property` | ExUnit + PropCheck |
| `multiplayer-fabric-zone-backend` | `mix test` | ExUnit |
| `multiplayer-fabric-zone-console` | `mix test` | ExUnit + PropCheck |
| `multiplayer-fabric-sandbox` | `ctest --test-dir build` | CMake / CTest + ASAN |
