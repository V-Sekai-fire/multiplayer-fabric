# AGENTS.md

Guidance for AI coding agents working in this repository.

The submodule list is the canonical source of truth in [README.md](README.md).
Contribution rules, language guides, documentation style, and writing anti-tropes are in [CONTRIBUTING.md](CONTRIBUTING.md). This file covers only what is specific to agent workflows.

## Working with submodules

```sh
# Clone everything
git clone --recurse-submodules <root-url>

# After pulling root updates, sync submodule pointers
git submodule update --init --recursive

# Work inside a submodule as a normal git repo
cd multiplayer-fabric-taskweft
git checkout -b my-branch
# ... make changes, commit, push ...
cd ..

# Record the updated submodule pointer in the root repo
git add multiplayer-fabric-taskweft
git commit -m "Update taskweft submodule pointer"
```

Never commit changes directly to files inside a submodule from the root repo. Always `cd` into the submodule first. Each submodule has its own `CONTRIBUTING.md` — read it before making changes.

## Critical cross-submodule relationships

### multiplayer-fabric-godot — branch rules

`multiplayer-fabric` is a **generated branch** — it is overwritten by the assembly script in `multiplayer-fabric-merge`. Never commit unique work directly to it. All custom module work goes on:

- `feat/module-multiplayer-fabric` — mmog module, Lean proofs
- `feat/module-lasso` — lasso AR module (includes the E2BIG fix)

When `multiplayer-fabric-merge` runs, it fetches these branches from the `v-sekai-fire` remote and reassembles `multiplayer-fabric`. Before triggering a reassembly, verify that any unique commits on `multiplayer-fabric` are already present on a feature branch.

### multiplayer-fabric-sandbox → multiplayer-fabric-godot

`multiplayer-fabric-sandbox` produces RISC-V ELF kernels consumed by the `godot-sandbox` module inside `multiplayer-fabric-godot`. Changing a public kernel symbol requires updating the binding table in the godot module at the same time. Test the integration by building the godot fork with the updated sandbox binary.

### multiplayer-fabric-taskweft → multiplayer-fabric-artifacts-mmog / zone-console

`multiplayer-fabric-taskweft` is a library dependency of both the artifact CLI and the zone console. A breaking API change in taskweft requires updates in both consumers before either is released.

## Running the godot fork assembly

```sh
cd multiplayer-fabric-merge

# Dry run (assemble locally, do not push)
elixir update_godot_v_sekai.exs --dry-run

# Live run (assembles and force-pushes multiplayer-fabric)
elixir update_godot_v_sekai.exs
```

`gitassembly` is the plain-text merge config read by `thirdparty/git-assembler` (Python 3). Do not pass it to `elixir`.

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

## Per-submodule test commands

| Submodule | Test command | Framework |
|-----------|-------------|-----------|
| `multiplayer-fabric-taskweft` | `mix test --include property` | ExUnit + PropCheck |
| `multiplayer-fabric-zone-backend` | `mix test` | ExUnit |
| `multiplayer-fabric-zone-console` | `mix test` | ExUnit + PropCheck |
| `multiplayer-fabric-artifacts-mmog` | `mix test` | ExUnit |
| `multiplayer-fabric-deploy` | `mix test` | ExUnit |
| `multiplayer-fabric-desync` | `go test ./...` | Go testing |
| `multiplayer-fabric-sandbox` | `ctest --test-dir build` | CMake / CTest + ASAN |
| Godot submodules | `godot --headless --quit` | Godot test scenes |
