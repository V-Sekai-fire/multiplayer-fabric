# multiplayer-fabric strategy

Own the smallest viable market completely, then grow.

## The prior failure

A well-funded company once pitched a distributed simulation platform as an OS for game worlds, arguing that existing servers could never make a world feel real. Two flagship games launched and failed. The platform shut down.

The architecture was wrong. The platform sat above the engine and tried to abstract the spatial problem away. Latency, interest management, and zone state cannot be separated from the engine — they are game problems that must be solved inside it. The spatial simulation problem is real and nobody has solved it this way.

multiplayer-fabric fills that gap: the same ambition, built as Godot modules instead of a cloud layer.

## Three uncharted gaps

No production-ready zone-border handoff exists outside one proprietary engine, and the absence is structural rather than a matter of timing. The delta-sync approach from OS image distribution solves game asset delivery mathematically, but the game-engine integration has never been built. Relationship-based access control tooling is mature; adoption in game backends is zero.

## Two moves

No shipped Godot project uses WebTransport as its primary transport. multiplayer-fabric has it wired, which is a concrete head start on the only production implementation.

`docker compose up` and a working zone server with WebTransport, content-addressed delivery, and relationship-based permissions is the wedge for solo Godot developers. Design documentation for each built component is in `manuals/decisions/`.
