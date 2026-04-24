# TODO

## Replace desync with aria-storage ✅ COMPLETE

`multiplayer-fabric-desync` (Go) has been replaced by `aria-storage` (Elixir).

### All phases done

- Lean4 proofs: buzhash termination, chunk partition invariant, SHA-512/256 identity
- Path format fixed: `storage_dir` uses the 4-char desync prefix (`<abcd>/<id>.cacnk`)
- SHA-512/256 bug fixed in aria-storage (was truncated SHA-512, now FIPS 180-4 §6.7)
- `AriaStorage.ChunkServerPlug`: GET / HEAD / PUT wire protocol, PropCheck tested
- Wired into zone-backend router at `/chunks/*`
- Caddy rerouted: `/chunks/*` → `uro:4000` (was `desync:9090`)
- `desync:` service removed from docker-compose
- Godot upload path: `put_chunk` + `upload_asset` + PUT/HEAD in `http_request_blocking`
- `FabricMMOGAsset` extracted into standalone `multiplayer_fabric_asset` Godot module
- Stack tested: GET/HEAD `/chunks/...` returns 404 (chunk not in store — correct)

### Remaining (Phase 6 — decommission submodule)

When staging is verified stable, remove the submodule and archive the repo:

```sh
git submodule deinit multiplayer-fabric-desync
git rm multiplayer-fabric-desync
git commit -m "Remove multiplayer-fabric-desync submodule — replaced by aria-storage"
git push
gh repo archive V-Sekai-fire/multiplayer-fabric-desync --yes
```

---

## Wire baker → aria-storage

`multiplayer-fabric-baker` bakes Godot zone assets into content-addressed chunks.

**What needs to happen:** after a successful bake, baker should:

1. PUT each `.cacnk` chunk to `/chunks/<prefix>/<id>.cacnk` on zone-backend
   (the `AriaStorage.ChunkServerPlug` PUT endpoint — now live)
2. POST the `.caibx` index to `/storage/:id/bake` on zone-backend so
   `SharedFile.baked_url` is set and Godot clients can discover the asset

Integration point: the versitygw bucket `zone-chunks` (S3-compatible, at
`http://versitygw:7070` inside the stack). No new infrastructure needed.
