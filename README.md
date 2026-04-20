# multiplayer-fabric

An open-source social VR platform built on a custom Godot Engine fork. This root repo is a git submodule index; all real work happens inside the submodules.

- **[CONTRIBUTING.md](CONTRIBUTING.md)** — contribution philosophy, language guides, doc style rules, Lean-proved invariants, and distribution workflows.
- **[AGENTS.md](AGENTS.md)** — AI agent guidance: submodule git workflow, cross-repo relationships, assembly instructions, and per-submodule test commands.

## Submodule map

| Path | Language | Role |
|------|----------|------|
| `multiplayer-fabric-godot` | C++ / GDScript | Godot engine fork; hosts `multiplayer_fabric`, `multiplayer_fabric_mmog`, `keychain`, and `godot-sandbox` modules |
| `multiplayer-fabric-merge` | Elixir / Python 3 | Assembles the godot fork from feature branches via `git-assembler` |
| `multiplayer-fabric-sandbox` | C++20 / CMake | RISC-V simulation kernels (jellygrid, Taskweft planner) consumed by godot-sandbox |
| `multiplayer-fabric-taskweft` | Elixir / C++ NIF | HRR-backed Ecto adapter with HTN planner, persisted to SQLite |
| `multiplayer-fabric-rx` | GDScript | V-Sekai social VR Godot project (playable client) |
| `multiplayer-fabric-humanoid-project` | GDScript | VRM avatar addon and MToon shader Godot project |
| `multiplayer-fabric-interaction-system` | GDScript | Raycast / VR interaction Godot addon |
| `multiplayer-fabric-zone-backend` | Elixir / Phoenix | Uro — zone management server (PostgreSQL, Redis, S3, WebTransport) |
| `multiplayer-fabric-zone-console` | Elixir escript | CLI console for zone inspection over HTTP + WebTransport |
| `multiplayer-fabric-artifacts-mmog` | Elixir escript | MMOG artifact upload and lifecycle TUI |
| `multiplayer-fabric-deploy` | Elixir | Release builder and multi-target deployment tooling |
| `multiplayer-fabric-desync` | Go | Content-addressed chunk synchronization (`casync`-compatible) |
| `homebrew-multiplayer-fabric` | Ruby | macOS Homebrew tap |
| `scoop-multiplayer-fabric` | JSON | Windows Scoop bucket |

## Quick start

```sh
git clone --recurse-submodules <root-url>
git submodule update --init --recursive
```
