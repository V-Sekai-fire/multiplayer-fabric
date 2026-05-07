# multiplayer-fabric

An open-source social VR platform built on a custom Godot Engine fork. This root repo is a git submodule index; all real work happens inside the submodules.

## Quick start

```sh
git clone --recurse-submodules https://github.com/V-Sekai-fire/multiplayer-fabric
cd multiplayer-fabric
git submodule update --init --recursive
```

### Windows

Develop inside **WSL2** (Ubuntu). Native cmd / PowerShell isn't supported: the local workflow assumes bash scripts (`#!/usr/bin/env bash`), POSIX paths (`/tmp`, `$HOME`), symlinks in the source tree (`references.bib`, `multiplayer-fabric-hosting/generate-secrets.sh`), `lsof`-style port checks, and a UNIX docker socket mount. Godot itself ships a Windows build via `multiplayer-fabric-godot/.github/workflows/windows_builds.yml`, but the rest of the dev tooling is bash-first.

## Repository layout

The full canonical list (with descriptions) is at [v-sekai-fire.github.io/manuals](https://v-sekai-fire.github.io/manuals/) and in [`multiplayer-fabric-manuals/index.md`](multiplayer-fabric-manuals/index.md). The 30 submodules are grouped into:

- **Runtime services** (Fly.io): gateway, zone, zone-backend (uro), zone-console, crdb, baker, observability
- **Engine**: godot, godot-maglev, merge, build, opentelemetry-godot(-project), webtransport, interaction-system(-project)
- **Game systems**: taskweft, predictive-bvh(-research), aria-storage, humanoid-project, llm
- **Infrastructure & tooling**: infra, docker-multiplayer-fabric, hosting, generate-secrets, elf-programs, casync-seed, cockroach
- **Testing & verification**: cycle-tests
- **Skills & docs**: manuals, skills

## Building the engine

The Godot engine is composed from feature branches at build time. The recipe is `multiplayer-fabric-merge/gitassembly`. Build via:

```sh
cd multiplayer-fabric-merge && git checkout multiplayer-fabric-base
gscons     # macOS arm64 editor (cache-warm: ~30 s)
gmscons    # Windows x86_64 cross-compile (mingw + llvm)
gescons    # Web template_release (wasm32, threads)
```

The aliases live in `~/.zshrc`. Output binaries go to `bin/godot.<platform>.editor.dev.double.<arch>`.

## Maglev cycles

End-to-end validation runs as 13 sequenced cycles defined in [`multiplayer-fabric-manuals/decisions/`](https://v-sekai-fire.github.io/manuals/decisions.html). Status (2026-05-07): C0 + C1 pass; C2 producer-side complete; C3+ pending.

Smoke tests for each cycle live in [`multiplayer-fabric-cycle-tests/`](https://github.com/V-Sekai-fire/multiplayer-fabric-cycle-tests):

```sh
godot --headless --script multiplayer-fabric-cycle-tests/cycle-1-gateway-handshake/cycle1.gd
```

Live Fly state can be verified without local flyctl auth via the read-only workflows on `multiplayer-fabric-infra`:

```sh
gh workflow run verify_fly_state.yml --repo V-Sekai-fire/multiplayer-fabric-infra
gh workflow run verify_observability.yml --repo V-Sekai-fire/multiplayer-fabric-infra
```

Both upload `*-state.json` artifacts containing the live `flyctl status / volumes / secrets / ips` and Victoria* HTTP query results.

## Documentation

- [Maglev cycle ADRs](https://v-sekai-fire.github.io/manuals/decisions.html) — design + pass criteria
- [Changelog](https://v-sekai-fire.github.io/manuals/changelog.html) — daily deck logs
- [Repository index](https://v-sekai-fire.github.io/manuals/) — every submodule with purpose
- [`AGENTS.md`](AGENTS.md) — agent workflow rules, commit style, work queue
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — language guides, anti-tropes, doc style
