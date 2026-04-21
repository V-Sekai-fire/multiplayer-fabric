# Tier 3 — Real value, furthest from complete

Requires a new architectural layer. Build after tiers 1 and 2 are done.

Strategic context: `strategy.md`
Architecture constraints: `architecture.md`

---

## 5. SQLite-per-zone with WAL replay

**Effort: high — Impact: medium — ~25% built**

Addresses strategy move #4 (SQLite-per-zone with deterministic replay).

SpacetimeDB keeps everything in memory. KBEngine uses MySQL. `multiplayer-fabric-taskweft` already depends on `exqlite` and has a `store.ex`. A zone server that can replay from a SQLite journal after a crash recovers exact state — something no vendor platform can offer without admitting their infrastructure fails. For self-hosters, this is the reliability story that justifies running their own stack.

- [ ] Design zone server to write all entity mutations as journal entries to SQLite WAL
- [ ] On zone crash/restart, replay the WAL to recover exact state
