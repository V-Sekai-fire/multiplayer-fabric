# Tier 2 — Do next

Most of the hard work exists. Each item needs one new connector on top of a complete library.

Strategic context: `strategy.md`
Architecture constraints: `architecture.md`

---

## Cleanup: delete stale zone_console copy in godot repo

- [x] Delete `modules/multiplayer_fabric_mmog/tools/zone_console` from `multiplayer-fabric-godot` — directory never existed in any branch; already clean

---

## 3. Zone crossing → desync delta-sync trigger

**Effort: medium — Impact: high — ~50% built**

Addresses strategy move #2 (CAIBX chunk delivery into zone entry).

`multiplayer-fabric-desync` is a complete Go implementation of casync with S3, HTTP, and local backends. Zone-backend has zone metadata and player-position concepts. The content-addressing math is solved. The game-engine integration does not exist anywhere.

- [x] When the zone server decides a player has crossed a boundary, emit a desync index URL to the client so it fetches only the chunks it lacks
- [x] One Phoenix channel message + a desync HTTP server pointed at the zone's asset store

---

## 4. Godot client-side WebTransport module

**Effort: high — Impact: high — ~40% built**

Addresses strategy move #1 (own the Godot-native WebTransport story).

WebTransport datagrams are strictly better than ENet for browser-accessible zones. Proposal #3899 has been open for four years with no merge. No shipped Godot project uses WebTransport as its primary transport. The server side (`multiplayer-fabric-webtransport` Elixir NIF over wtransport Rust) is complete.

- [ ] Implement a `NetworkedMultiplayerPeer` subclass in the `feat/module-multiplayer-fabric` branch backed by wtransport's C API
- [ ] Closes Godot proposal #3899's gap without waiting for upstream merge
