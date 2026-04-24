# TODO

## Replace desync with aria-storage (in progress)

`multiplayer-fabric-desync` (Go) is being replaced by `aria-storage` (Elixir).
The Elixir library is wire-compatible with desync clients (Godot, baker) and is
already built into zone-backend as a dependency.

### Done

- Lean4 proofs: buzhash termination, chunk partition invariant, SHA-512/256 identity
- Path format fixed: `storage_dir` now uses the 4-char desync prefix (`<abcd>/<id>.cacnk`)
- `AriaStorage.ChunkServerPlug`: Plug implementing GET / HEAD / PUT wire protocol
- SHA-512/256 bug fixed in aria-storage (was truncated SHA-512, now FIPS 180-4 §6.7)
- Godot upload path: `put_chunk` + `upload_asset` + PUT/HEAD in `http_request_blocking`
- `FabricMMOGAsset` extracted into standalone `multiplayer_fabric_asset` Godot module

### Remaining — three steps to decommission desync

**1. Wire ChunkServerPlug into zone-backend router**

`multiplayer-fabric-zone-backend/lib/uro/router.ex` — add:

```elixir
scope "/chunks" do
  forward "/", AriaStorage.ChunkServerPlug, writeable: true
end
```

Also add `{:aria_storage, github: "V-Sekai-fire/aria-storage"}` to
`multiplayer-fabric-zone-backend/mix.exs` deps.

**2. Reroute Caddy from desync to zone-backend**

`multiplayer-fabric-hosting/Caddyfile` — change:

```
# before
handle /chunks/* {
    reverse_proxy desync:9090
}

# after
handle /chunks/* {
    reverse_proxy uro:4000
}
```

**3. Remove desync from docker-compose**

`multiplayer-fabric-hosting/docker-compose.yml` — delete the `desync:` service
block and any `depends_on: desync` references.

After all three are verified in staging, remove the `multiplayer-fabric-desync`
submodule and archive the GitHub repo.

---

## Wire baker → aria-storage

`multiplayer-fabric-baker` bakes Godot zone assets into content-addressed chunks.

**What needs to happen:** after a successful bake, baker should:

1. PUT each `.cacnk` chunk to `POST /chunks/<prefix>/<id>.cacnk` on zone-backend
   (the `ChunkServerPlug` PUT endpoint, once step 1 above is done)
2. POST the `.caibx` index to `/storage/:id/bake` on zone-backend so
   `SharedFile.baked_url` is set and Godot clients can discover the asset

Integration point: the versitygw bucket `zone-chunks` (S3-compatible, at
`http://versitygw:7070` inside the stack). No new infrastructure needed once
the ChunkServerPlug endpoint is live.
