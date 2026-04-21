# multiplayer-fabric strategy

Own the smallest viable market completely, then grow. The last mover wins.

## The prior failure

A well-funded company once pitched a distributed simulation platform as an OS for game worlds. The claim: existing servers could never make a world feel real. Two flagship games failed. The platform shut down.

The cause was architectural. The platform sat above the engine and tried to abstract the spatial problem away. Latency, interest management, and zone state cannot be separated from the engine — they are game problems that must be solved inside it.

That is the gap multiplayer-fabric fills: the same spatial simulation ambition, built as Godot modules instead of a cloud layer.

## Three uncharted gaps

**Zone stitching** — outside one proprietary engine, no production-ready zone-border handoff exists. Not stalled. Absent.

**Content-addressed game asset delivery** — the delta-sync math is solved. The game-engine integration has never been built.

**ReBAC as a game permission primitive** — general-purpose relationship-based access control tools are mature. Adoption in game backends is zero.

## Two moves

**Own the Godot-native WebTransport story.** No shipped Godot project uses WebTransport as its primary transport. multiplayer-fabric has it wired.

**Target the solo Godot developer first.** `docker compose up` and a working zone server with WebTransport, CAIBX, and ReBAC is the wedge. See `manuals/decisions/` for design documentation on each built component.

## The secret

The prior failure was an architectural mistake, not a vision mistake. The spatial simulation problem is real. Solving it inside the engine is the correct approach. Nobody has built it.
