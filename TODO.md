# TODO

## Replace desync with aria-storage ✅ COMPLETE

`multiplayer-fabric-desync` (Go) has been replaced by `aria-storage` (Elixir).
Submodule removed from monorepo. GitHub repo archived.

## Wire baker → aria-storage ✅ COMPLETE

The baker now chunks, uploads, and registers assets end-to-end:

1. `baker/run.gd` — after export, calls `FabricMMOGAsset.upload_asset_gd(chunk_url, file_bytes)`
   which chunks the file with buzhash, PUTs each `.cacnk` to `/chunks/*` on zone-backend,
   and returns the `.caibx` index bytes.

2. `baker/run.gd` — base64-encodes the caibx and POSTs `{caibx_data: "..."}` to
   `/storage/:id/bake` on zone-backend.

3. `StorageController.bake/2` — accepts `caibx_data` (base64), PUTs the `.caibx` to
   `zone-chunks/<id>.caibx` in versitygw via ExAws, sets `SharedFile.baked_url`.

4. Godot client — polls `/storage/:id/manifest` until `baked_url` is set, then
   downloads missing chunks via `FabricMMOGAsset.fetch_asset`.

---

No open items.
