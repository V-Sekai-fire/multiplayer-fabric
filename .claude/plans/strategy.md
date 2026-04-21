# multiplayer-fabric strategy

Peter Thiel's core argument in Zero to One: competition is for losers. The goal is to find a secret — something true that almost no one believes yet — build a monopoly around it in a small market, then expand. The last mover wins, not the first. First movers get disrupted; last movers define the category permanently. Distribution matters as much as product. And the power law means one correct bet beats a portfolio of hedged ones.

Applied here: find the specific combination of truths nobody else is building toward, own the smallest viable market completely, then grow from there.

## What Improbable wanted to build in 2015

Herman Narula launched SpatialOS at Slush Helsinki in November 2015 as a distributed operating system for simulations — the same relationship to games that Windows has to applications. His claim: existing server architectures could never make a world feel real, because a real world requires millions of entities interacting simultaneously with persistent consequences and spatial awareness. He called this "strong simulation" and pointed at worlds like the Matrix as the destination.

For developers the pitch was concrete: offload all distributed-systems complexity to SpatialOS, and a two-person team could simulate a world the size of Israel with four million entities. MMO scale without an MMO-sized infrastructure team.

It didn't work. The SDK was nearly unusable. The two flagship games — Worlds Adrift and Mavericks: Proving Grounds — both launched and failed commercially. After a $500M SoftBank raise, Improbable was losing £149M per year. The company sold its defence division in 2023 and the spatial OS vision quietly ended.

The failure had a specific cause. SpatialOS tried to sit above the game engine and abstract the spatial problem away from it entirely. The abstraction leaked under real load because latency, interest management, and zone state are not infrastructure problems separable from the engine — they are game problems that have to be solved inside it.

That is the gap multiplayer-fabric is positioned to fill: the same spatial OS ambition, built as Godot modules rather than a cloud layer the engine reports to.

## The 2026 landscape

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

**What changed since 2024:** SpacetimeDB is the most significant new entrant. BitCraft Online open-sourced its server code in January 2026 — the most complete public reference for a SpacetimeDB-powered MMOG. WebTransport adoption reached 27% across real-time sectors by early 2025 (up from 8% in 2024), but remains Chromium-only and has zero shipped Godot titles using it natively. Godot proposal #3899 (WebTransport support) has been open since 2022 and is still unmerged.

## What remains genuinely uncharted in 2026

Three gaps are confirmed unfilled after surveying the full landscape:

**Zone stitching outside Unreal.** Open World Server exists for Unreal. For Godot, Unity, or any other engine, there is no production-ready zone-border handoff layer. Not stalled — simply absent.

**Content-addressed game asset delivery.** casync solved delta-sync for OS images. Nobody has applied that to game world data. No shipped solution exists for a client entering a zone and delta-syncing only the asset chunks it doesn't already have. The math is solved; the integration is not built.

**ReBAC as a game permission primitive.** OpenFGA and SpiceDB both matured in 2025-2026. Both remain general-purpose auth systems with zero adoption in published game backends. "Who can open this door inside this guild-controlled zone" is a graph traversal, and every existing backend still answers it with role checks or custom ACL tables.

The SpacetimeDB gap is the most important one to understand precisely. SpacetimeDB handles game logic well — it is a real competitor for the "solo developer wants an MMOG backend" segment. But it does not handle zone topology, geographic partitioning, content delivery, or transport. A BitCraft-scale world on SpacetimeDB still requires a separate solution for where players are in space, how they cross boundaries, and what assets they need. That problem is unsolved.

## The five moves

### 1. Own the Godot-native WebTransport story

WebTransport datagrams are strictly better than ENet for browser-accessible zones. Proposal #3899 has been open for four years with no merge. No shipped Godot project uses WebTransport as its primary transport. multiplayer-fabric has it wired today. That is a four-year head start on the only production implementation.

### 2. Wire CAIBX chunk delivery into zone entry

When a player crosses a zone boundary, the client should delta-sync only the world data chunks it does not already have. The content-addressing math is solved. The game-engine integration does not exist anywhere. Building it here means owning the only implementation when zone-scale worlds become the default expectation.

### 3. Make ReBAC the zone permission model from day one

OpenFGA hit CNCF Incubating in October 2025. The tooling is mature. The adoption in game backends is zero. Coarse RBAC produces permission bugs at MMOG scale that are expensive to fix retroactively. Building the relationship graph in from the start means this cannot be bolt-on.

### 4. SQLite-per-zone with deterministic replay

SpacetimeDB keeps everything in memory. KBEngine uses MySQL. A zone server that can replay from a SQLite journal after a crash recovers exact state — something no vendor platform can offer without admitting their infrastructure fails. For self-hosters, this is the reliability story that justifies running their own stack.

### 5. Target the solo Godot developer first, then expand

SpacetimeDB's BitCraft reference is now public, which raises the bar. The differentiator is not "also a backend" — SpacetimeDB already owns that. The differentiator is the full spatial layer: zone handoff, content delivery, and permissions that SpacetimeDB does not provide. `docker compose up` and a working zone server with WebTransport, CAIBX, and ReBAC built in is the wedge. That combination does not exist anywhere else.

## The secret

The Improbable failure was an architectural mistake, not a vision mistake. The spatial OS problem is real. Solving it inside the engine rather than above it is the correct approach, and four years after Improbable's collapse, nobody has built that.

Execution plan and task checklists live in `architecture.md`.
