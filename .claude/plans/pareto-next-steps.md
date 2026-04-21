# Multiplayer Fabric — Pareto Next Steps

Strategic context lives in `multiplayer-fabric-strategy.md`. This file owns the execution plan: architecture constraints, task ordering, and actionable checklists.

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

## Tiers

### Tier 1 — Do now, most value already built

| #   | Task                                  | Effort | Impact | Built |
| --- | ------------------------------------- | ------ | ------ | ----- |
| 1   | Publish one-command docker compose    | low    | high   | ~75%  |
| 2   | Wire taskweft ReBAC into zone-backend | low    | high   | ~80%  |

### Tier 2 — Do next, most of the hard work exists

| #   | Task                                      | Effort | Impact | Built |
| --- | ----------------------------------------- | ------ | ------ | ----- |
| 3   | Zone crossing → desync delta-sync trigger | medium | high   | ~50%  |
| 4   | Godot client-side WebTransport module     | high   | high   | ~40%  |

### Tier 3 — Real value, furthest from complete

| #   | Task                            | Effort | Impact | Built |
| --- | ------------------------------- | ------ | ------ | ----- |
| 5   | SQLite-per-zone with WAL replay | high   | medium | ~25%  |

---

## Task details

### 1. Publish a one-command docker compose showing all three layers

Addresses strategy move #5 (target the solo Godot developer first).

`multiplayer-fabric-hosting` already has a compose file and zone-backend is live at hub-700a.chibifire.com. The differentiator is the full spatial layer — zone handoff, content delivery, and permissions that SpacetimeDB does not provide — but a product nobody can run yet is not a product. Ship the wedge first; the differentiated layer lands in v0.2.

- [ ] Update compose so a solo developer gets: zone server + WebTransport listener + desync HTTP chunk server + ReBAC-gated content in one command
- [ ] Add a worked example in the README: `docker compose up`, connect a Godot client, cross a zone boundary
- [ ] Tag v0.1 once a developer outside the project can follow the README cold and reach a running zone server

---

### 2. Wire taskweft ReBAC into zone-backend permissions

Addresses strategy move #3 (ReBAC as zone permission model). Lands as v0.2 immediately after the compose story ships.

`multiplayer-fabric-taskweft` has a fully-proven C++ NIF ReBAC graph (93 PropCheck properties passing). `zone-backend` has `user_relations/` and `user_content/` — the permission call sites exist but still use flat boolean guards (`can_upload_maps`, `is_admin`).

- [ ] Add `{:taskweft, github: "V-Sekai-fire/multiplayer-fabric-taskweft"}` to zone-backend's `mix.exs`
- [ ] Define a relation schema: user `OWNS` content, user `IS_MEMBER_OF` zone
- [ ] Replace `RequireMapUploadPermission`, `RequireAvatarUploadPermission`, `RequirePropUploadPermission` with `Taskweft.ReBAC.check/3` calls
- [ ] Godot module stays unchanged — it checks capabilities in the JWT, not the graph

---

### 3. Zone crossing → desync delta-sync trigger

Addresses strategy move #2 (CAIBX chunk delivery into zone entry).

`multiplayer-fabric-desync` is a complete Go implementation of casync with S3, HTTP, and local backends. Zone-backend has zone metadata and player-position concepts. The content-addressing math is solved. The game-engine integration does not exist anywhere.

- [ ] When the zone server decides a player has crossed a boundary, emit a desync index URL to the client so it fetches only the chunks it lacks
- [ ] One Phoenix channel message + a desync HTTP server pointed at the zone's asset store

---

### 4. Godot client-side WebTransport module

Addresses strategy move #1 (own the Godot-native WebTransport story).

WebTransport datagrams are strictly better than ENet for browser-accessible zones. Proposal #3899 has been open for four years with no merge. No shipped Godot project uses WebTransport as its primary transport. The server side (`multiplayer-fabric-webtransport` Elixir NIF over wtransport Rust) is complete.

- [ ] Implement a `NetworkedMultiplayerPeer` subclass in the `feat/module-multiplayer-fabric` branch backed by wtransport's C API
- [ ] Closes Godot proposal #3899's gap without waiting for upstream merge

---

### 5. SQLite-per-zone with WAL replay

Addresses strategy move #4 (SQLite-per-zone with deterministic replay).

SpacetimeDB keeps everything in memory. KBEngine uses MySQL. `multiplayer-fabric-taskweft` already depends on `exqlite` and has a `store.ex`. A zone server that can replay from a SQLite journal after a crash recovers exact state — something no vendor platform can offer without admitting their infrastructure fails.

- [ ] Design zone server to write all entity mutations as journal entries to SQLite WAL
- [ ] On zone crash/restart, replay the WAL to recover exact state

---

## The pattern

Items 1 and 2 are almost entirely assembly of existing pieces — the code is written, it just is not wired together or framed as a product. Items 3 and 4 each need one new connector on top of a complete library. Item 5 requires a new architectural layer.

Item 1 (compose) ships first because distribution matters as much as product. A running demo that a solo developer can reach in one command gets shared. Item 2 (ReBAC) ships immediately after as v0.2 — it upgrades the running system from "also a backend" to "the only backend with a permission graph." Items 3 and 4 follow to complete the three-layer claim demonstrable in a single `docker compose up` before writing significant new code.

Strategy moves map to tiers: move #5 and #3 → tier 1; move #2 and #1 → tier 2; move #4 → tier 3.
