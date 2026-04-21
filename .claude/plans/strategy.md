# multiplayer-fabric strategy

The goal is to find a combination of truths nobody else is building toward, own the smallest viable market completely, then grow from there. The last mover wins, not the first. Distribution matters as much as product.

## A prior attempt and its failure

Around 2015, a well-funded company launched a distributed simulation platform pitched as an operating system for game worlds — the claim being that existing server architectures could never make a world feel real, because a real world requires millions of entities interacting simultaneously with persistent consequences and spatial awareness.

The pitch was concrete: offload all distributed-systems complexity to the platform, and a small team could simulate a very large world. MMO scale without an MMO-sized infrastructure team.

It didn't work. Two flagship games launched and failed commercially. The platform lost hundreds of millions per year and the vision quietly ended.

The failure had a specific cause. The platform tried to sit above the game engine and abstract the spatial problem away from it entirely. The abstraction leaked under real load because latency, interest management, and zone state are not infrastructure problems separable from the engine — they are game problems that have to be solved inside it.

That is the gap multiplayer-fabric is positioned to fill: the same spatial simulation ambition, built as Godot modules rather than a cloud layer the engine reports to.

## The 2026 landscape

Categories of existing solutions and their gaps:

| Category | What it does | Gap |
|---|---|---|
| DB-as-server (in-memory, WASM logic) | Game logic co-located with data | No zone topology, no content delivery, no WebTransport |
| Full-stack self-hostable backend | Auth, matchmaking, presence, leaderboards | No zone streaming, no spatial partitioning |
| Server orchestration | Spins up and routes to game server pods | Orchestration only, no zone stitching, no game logic |
| Godot-specific hosted backend | Godot-native auth and storage | Hosted only, no zone architecture |
| Indie auth/saves libraries | Auth, leaderboards, saves | No multiplayer, no zones |
| Kubernetes pod management | Manages dedicated server lifecycle | Orchestration only |
| Legacy MMOG zone frameworks | C++/scripted zone servers | No WebTransport, unmaintained |
| Node.js room servers | Lightweight rooms, no spatial state | No zone continuity |
| Managed cloud backends | Everything, hosted | Vendor lock-in |
| Content-addressed sync tools | Delta sync for OS images | Never applied to game assets |
| General-purpose ReBAC engines | Fine-grained auth graph | Zero adoption in any game backend |

WebTransport adoption has grown significantly in real-time sectors but has zero shipped Godot titles using it natively. The Godot engine proposal for first-class WebTransport support has been open for several years and is still unmerged.

## What remains genuinely uncharted in 2026

Three gaps are confirmed unfilled after surveying the full landscape:

**Zone stitching outside one major engine.** One proprietary game engine has a production-ready zone-border handoff layer. For Godot, or any other engine, there is no equivalent. Not stalled — simply absent.

**Content-addressed game asset delivery.** The delta-sync approach from OS image distribution solves the problem mathematically. Nobody has applied it to game world data. No shipped solution exists for a client entering a zone and syncing only the asset chunks it does not already have. The math is solved; the game-engine integration is not built.

**ReBAC as a game permission primitive.** General-purpose relationship-based access control tools matured in 2025–2026. Both remain general-purpose auth systems with zero adoption in published game backends. "Who can open this door inside this guild-controlled zone" is a graph traversal, and every existing backend still answers it with role checks or custom ACL tables.

## The two moves

### 1. Own the Godot-native WebTransport story

WebTransport datagrams are better suited than ENet for browser-accessible zones. No shipped Godot project uses WebTransport as its primary transport. multiplayer-fabric has it wired today.

### 2. Target the solo Godot developer first, then expand

`docker compose up` and a working zone server with WebTransport, CAIBX, and ReBAC built in is the wedge. The full spatial layer — zone handoff, content delivery, and permissions — is the differentiator. See `manuals/decisions/` for design documentation on each built component.

## The secret

The prior failure was an architectural mistake, not a vision mistake. The spatial simulation problem is real. Solving it inside the engine rather than above it is the correct approach, and years after that collapse, nobody has built it.
