# Multiplayer Fabric — Pareto Next Steps

## The secret (from the gist)

The truth almost nobody is acting on: the Improbable failure was an architectural mistake, not a vision mistake. SpatialOS tried to sit above the game engine and abstract the spatial problem away from it entirely. The abstraction leaked under real load because latency, interest management, and zone state are not infrastructure problems separable from the engine — they are game problems that have to be solved inside it.

The spatial OS problem is real. Solving it inside the engine rather than above it is the correct approach, and four years after Improbable's collapse, nobody has built that. multiplayer-fabric is positioned to fill that gap as Godot modules rather than a cloud layer the engine reports to.

---

## 2026 competitive landscape

| Project                | Status                            | What it does                                   | Gap                                                                  |
| ---------------------- | --------------------------------- | ---------------------------------------------- | -------------------------------------------------------------------- |
| SpacetimeDB v2.0       | Shipped, free tier, growing       | DB-as-server, WASM logic, BitCraft ships on it | No zone topology, no content delivery, no WebTransport, BSL licensed |
| Nakama                 | Dominant, 1T req/month            | Full-stack self-hostable backend               | No zone streaming, no spatial partitioning                           |
| Rivet                  | YC-backed, pivoting to Actors     | Self-hostable server orchestration (Rust)      | Orchestration only, no zone stitching, no game logic                 |
| W4 Cloud               | Running, W4 Build killed Jan 2026 | Godot-specific hosted backend                  | Hosted only, no zone architecture, uncertain future                  |
| Talo                   | Small, MIT licensed               | Auth, leaderboards, saves for indie/Godot      | No multiplayer, no zones                                             |
| Agones                 | Stable                            | Kubernetes server pod management               | Orchestration only                                                   |
| KBEngine               | Unmaintained                      | C++/Python MMOG zones                          | No WebTransport, stagnant                                            |
| Colyseus               | Stable                            | Node.js room server                            | No zone continuity                                                   |
| PlayFab / EOS / Photon | Managed cloud                     | Everything, hosted                             | Vendor lock-in                                                       |
| casync / desync        | Stable, unmoved                   | Content-addressed sync for OS images           | Never applied to game assets                                         |
| OpenFGA                | CNCF Incubating, v1.13            | Zanzibar ReBAC                                 | Zero adoption in any game backend                                    |

SpacetimeDB is the most significant new entrant. BitCraft Online open-sourced its server code in January 2026. SpacetimeDB handles game logic well but does not handle zone topology, geographic partitioning, content delivery, or transport. That problem is unsolved.

---

## Three gaps confirmed unfilled in 2026

**Zone stitching outside Unreal.** Open World Server exists for Unreal. For Godot, Unity, or any other engine, there is no production-ready zone-border handoff layer. Not stalled — simply absent.

**Content-addressed game asset delivery.** casync solved delta-sync for OS images. Nobody has applied that to game world data. No shipped solution exists for a client entering a zone and delta-syncing only the asset chunks it doesn't already have. The math is solved; the integration is not built.

**ReBAC as a game permission primitive.** OpenFGA and SpiceDB both matured in 2025-2026. Both remain general-purpose auth systems with zero adoption in published game backends. "Who can open this door inside this guild-controlled zone" is a graph traversal, and every existing backend still answers it with role checks or custom ACL tables.

---

## Architecture boundary (read before touching code)

The three "backends" are different things serving different purposes:

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

## Tier 1 — Do now, most value already built

### 1. Wire taskweft ReBAC into zone-backend permissions

**Effort: low — Impact: high — ~80% built**

Addresses gist move #3 (ReBAC as zone permission model) and move #5 (solo developer wedge).

`multiplayer-fabric-taskweft` has a fully-proven C++ NIF ReBAC graph (93 PropCheck properties passing). `zone-backend` has `user_relations/` and `user_content/` — the permission call sites exist but still use flat boolean guards (`can_upload_maps`, `is_admin`).

**What to do:**

- Add `{:taskweft, github: "V-Sekai-fire/multiplayer-fabric-taskweft"}` to zone-backend's `mix.exs`
- Define a relation schema: user `OWNS` content, user `IS_MEMBER_OF` zone
- Replace the three permission plugs (`RequireMapUploadPermission`, `RequireAvatarUploadPermission`, `RequirePropUploadPermission`) with `Taskweft.ReBAC.check/3` calls
- Godot module stays unchanged — it checks capabilities in the JWT, not the graph

Zero new infrastructure. Zero other project anywhere has this.

---

### 2. Publish a one-command docker compose showing all three layers

**Effort: low — Impact: high — ~75% built**

Addresses gist move #5 (target the solo Godot developer first).

`multiplayer-fabric-hosting` already has a compose file and zone-backend is live at hub-700a.chibifire.com. The differentiator is not "also a backend" — SpacetimeDB already owns that. The differentiator is the full spatial layer: zone handoff, content delivery, and permissions that SpacetimeDB does not provide.

**What to do:**

- After #1 above, update compose so a solo developer gets: zone server + WebTransport listener + desync HTTP chunk server + ReBAC-gated content
- Add a worked example in the README: `docker compose up`, connect a Godot client, cross a zone boundary
- The wedge product the gist describes is essentially already running — it just isn't packaged as a single `docker compose up` story

---

## Tier 2 — Do next, most of the hard work exists

### 3. Zone crossing → desync delta-sync trigger

**Effort: medium — Impact: high — ~50% built**

Addresses gist move #2 (CAIBX chunk delivery into zone entry).

`multiplayer-fabric-desync` is a complete Go implementation of casync with S3, HTTP, and local backends. Zone-backend has zone metadata and player-position concepts. The content-addressing math is solved. The game-engine integration does not exist anywhere.

**What to do:**

- When the zone server decides a player has crossed a boundary, emit a desync index URL to the client so it fetches only the chunks it lacks
- One Phoenix channel message + a desync HTTP server pointed at the zone's asset store
- Nobody has built this integration for any engine

---

### 4. Godot client-side WebTransport module

**Effort: high — Impact: high — ~40% built**

Addresses gist move #1 (own the Godot-native WebTransport story).

WebTransport datagrams are strictly better than ENet for browser-accessible zones. Proposal #3899 has been open for four years with no merge. No shipped Godot project uses WebTransport as its primary transport. The server side (`multiplayer-fabric-webtransport` Elixir NIF over wtransport Rust) is complete. That is a four-year head start on the only production implementation.

**What to do:**

- Implement a `NetworkedMultiplayerPeer` subclass in the `feat/module-multiplayer-fabric` branch backed by wtransport's C API
- Closes Godot proposal #3899's gap without waiting for upstream merge
- Completes the "only shipped Godot WebTransport title" claim

---

## Tier 3 — Real value, but furthest from complete

### 5. SQLite-per-zone with WAL replay

**Effort: high — Impact: medium — ~25% built**

Addresses gist move #4 (SQLite-per-zone with deterministic replay).

SpacetimeDB keeps everything in memory. KBEngine uses MySQL. `multiplayer-fabric-taskweft` already depends on `exqlite` and has a `store.ex`. A zone server that can replay from a SQLite journal after a crash recovers exact state — something no vendor platform can offer without admitting their infrastructure fails.

**What to do:**

- Design zone server to write all entity mutations as journal entries to SQLite WAL
- On zone crash/restart, replay the WAL to recover exact state
- This is the reliability story for self-hosters; build after the top four

---

## The pattern

Items 1 and 2 are almost entirely assembly of existing pieces — the code is written, it just isn't wired together or framed as a product. Items 3 and 4 each need one new connector on top of a complete library. Item 5 requires a new architectural layer. This ordering delivers the full three-layer claim (WebTransport + CAIBX + ReBAC) demonstrable in a single `docker compose up` before writing significant new code.

The gist's five moves map to tiers as follows: move #3 and #5 → tier 1; move #2 and #1 → tier 2; move #4 → tier 3.
