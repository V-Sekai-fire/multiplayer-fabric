# Multiplayer Fabric — Pareto Next Steps

Strategic context lives in `multiplayer-fabric-strategy.md`. This file owns the architecture constraints and task index. Task checklists live in the tier files.

---

## Architecture boundary (read before touching code)

| Component                          | Language            | Role                                                 |
| ---------------------------------- | ------------------- | ---------------------------------------------------- |
| `zone-backend` (Uro)               | Elixir/Phoenix      | Hub API: accounts, auth, asset uploads, zone listing |
| `taskweft`                         | Elixir + C++20 NIFs | Library: ReBAC engine, HTN planner, SQLite store     |
| `multiplayer-fabric-godot` modules | C++ (inside Godot)  | Game server: entity simulation, physics, zone state  |

Taskweft's C++ is compiled into a BEAM NIF `.so`. Godot's C++ is compiled into the engine binary. They never share a process.

**Uro owns pre-entry permissions.** "Can this user enter zone X, upload this map, own this content?" These are currently flat boolean fields in `UserPrivilegeRuleset`. Uro mints a JWT capability token on success.

**Godot zone server trusts the token.** The Godot C++ module does not run ReBAC. It receives the JWT from Uro, verifies the signature, and enforces the capabilities encoded in it. In-game checks ("can this character open this door") are simple capability presence checks against the token — not graph traversals.

**Taskweft is a library dep of Uro only.** It does not go near the Godot C++ build.

---

## Task index

| #   | Task                                      | Tier | Effort | Impact | Built | File                  |
| --- | ----------------------------------------- | ---- | ------ | ------ | ----- | --------------------- |
| 1   | Publish one-command docker compose        | 1    | low    | high   | ~75%  | `tier-1-do-now.md`    |
| 2   | Wire taskweft ReBAC into zone-backend     | 1    | low    | high   | ~80%  | `tier-1-do-now.md`    |
| 3   | Zone crossing → desync delta-sync trigger | 2    | medium | high   | ~50%  | `tier-2-do-next.md`   |
| 4   | Godot client-side WebTransport module     | 2    | high   | high   | ~40%  | `tier-2-do-next.md`   |
| 5   | SQLite-per-zone with WAL replay           | 3    | high   | medium | ~25%  | `tier-3-real-value.md`|

Items 1 and 2 are assembly of existing pieces. Items 3 and 4 each need one new connector on top of a complete library. Item 5 requires a new architectural layer.

Strategy moves map to tiers: move #5 and #3 → tier 1; move #2 and #1 → tier 2; move #4 → tier 3.
