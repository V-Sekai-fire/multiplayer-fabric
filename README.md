# multiplayer-fabric

An open-source social VR platform built on a custom Godot Engine fork. This root repo is a git submodule index; all real work happens inside the submodules.

## Quick start

```sh
git clone --recurse-submodules <root-url>
git submodule update --init --recursive
```

### Windows

Develop inside **WSL2** (Ubuntu). Native cmd / PowerShell isn't supported: the
local workflow assumes bash scripts (`#!/usr/bin/env bash`), POSIX paths
(`/tmp`, `$HOME`), symlinks in the source tree (`references.bib`,
`multiplayer-fabric-hosting/generate-secrets.sh`), `lsof`-style port checks,
and a UNIX docker socket mount. Godot itself ships a Windows build via
`multiplayer-fabric-godot/.github/workflows/windows_builds.yml`, but the rest
of the dev tooling is bash-first.
