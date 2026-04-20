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
