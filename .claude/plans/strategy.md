# multiplayer-fabric strategy

Own the smallest viable market completely, then grow.

## The prior failure

A well-funded company once pitched a distributed simulation platform as an OS for game worlds. The claim: existing servers could never make a world feel real. Two flagship games launched and failed. The platform shut down.

The architecture was wrong. The platform sat above the engine and tried to abstract the spatial problem away. Latency, interest management, and zone state cannot be separated from the engine — they are game problems that must be solved inside it. The spatial simulation problem is real and nobody has solved it this way.

That is the gap multiplayer-fabric fills: the same ambition, built as Godot modules instead of a cloud layer.

## Three uncharted gaps

Outside one proprietary engine, no production-ready zone-border handoff exists anywhere. The absence is structural.

The delta-sync approach from OS image distribution solves content delivery mathematically. The game-engine integration has never been built.

General-purpose relationship-based access control tools are mature. Adoption in game backends is zero.

## Two moves

No shipped Godot project uses WebTransport as its primary transport. multiplayer-fabric has it wired. That is a concrete head start on the only production implementation.

`docker compose up` and a working zone server with WebTransport, CAIBX, and ReBAC is the wedge for solo Godot developers. The full spatial layer — zone handoff, content delivery, and permissions — is what distinguishes it. Design documentation for each built component is in `manuals/decisions/`.
