# TODO

## Wire baker → desync

`multiplayer-fabric-baker` bakes Godot zone assets into content-addressed chunks.
`multiplayer-fabric-desync` serves those chunks to clients via the `/chunks/` endpoint.

The two repos have zero cross-references. Baker output goes nowhere; desync serves nothing.

**What needs to happen:** after a successful bake, baker should call desync (or write directly to the versitygw S3 bucket) so that the `.caibx` index and chunk store are populated. Clients can then delta-sync zone assets on zone entry.

The integration point is the versitygw bucket `zone-chunks` (S3-compatible, running at `http://versitygw:7070` inside the stack). Baker writes chunks; desync reads them. No new infrastructure needed — just the wire.
