# Tier 1 — Do now

Most value already built. Both items are assembly of existing pieces — the code is written, it just is not wired together or framed as a product.

Strategic context: `strategy.md`
Architecture constraints: `architecture.md`

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

## 1. Publish a one-command docker compose showing all three layers

**Effort: low — Impact: high — ~75% built**

Addresses strategy move #5 (target the solo Godot developer first).

`multiplayer-fabric-hosting` already has a compose file and zone-backend is live at hub-700a.chibifire.com. The differentiator is the full spatial layer — zone handoff, content delivery, and permissions that SpacetimeDB does not provide — but a product nobody can run yet is not a product. Ship the wedge first; the differentiated layer lands in v0.2.

- [x] Update compose so a solo developer gets: zone server + WebTransport listener + desync HTTP chunk server + ReBAC-gated content in one command
- [x] Add a worked example in the README: `docker compose up`, connect a Godot client, cross a zone boundary
- [x] Tag v0.1 once a developer outside the project can follow the README cold and reach a running zone server

---

## 2. Wire taskweft ReBAC into zone-backend permissions

**Effort: low — Impact: high — ~80% built**

Addresses strategy move #3 (ReBAC as zone permission model). Lands as v0.2 immediately after the compose story ships.

`multiplayer-fabric-taskweft` has a fully-proven C++ NIF ReBAC graph (93 PropCheck properties passing). `zone-backend` has `user_relations/` and `user_content/` — the permission call sites exist but still use flat boolean guards (`can_upload_maps`, `is_admin`).

- [ ] Add `{:taskweft, github: "V-Sekai-fire/multiplayer-fabric-taskweft"}` to zone-backend's `mix.exs`
- [ ] Define a relation schema: user `OWNS` content, user `IS_MEMBER_OF` zone
- [ ] Replace `RequireMapUploadPermission`, `RequireAvatarUploadPermission`, `RequirePropUploadPermission` with `Taskweft.ReBAC.check/3` calls
- [ ] Godot module stays unchanged — it checks capabilities in the JWT, not the graph
