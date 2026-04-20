# TODO

## Deploy to production (Fly.io)

All configs live in `multiplayer-fabric-fly/`. Images are built externally and pushed to GHCR before deploying.

### Cycles (dependency order)

| Cycle | What you get | Effort | Status |
| ----- | ------------ | ------ | ------ |
| 1 | Build + push `multiplayer-fabric-zone-backend` Docker image via `multiplayer-fabric-zone-backend` CI | Medium | [ ] |
| 2 | Build + push `multiplayer-fabric-godot-server` Docker image via `multiplayer-fabric-deploy` CI | High | [ ] |
| 3 | Build + push `ghcr.io/v-sekai/cockroach` Docker image via `v-sekai/cockroach` CI | Medium | [ ] |
| 4 | Create Fly apps and provision Tigris bucket | Low | [ ] |
| 5 | Deploy CockroachDB (`multiplayer-fabric-crdb`) with persistent volume | Low | [ ] |
| 6 | Deploy zone-backend (`multiplayer-fabric-uro`) and wire secrets | Low | [ ] |
| 7 | Deploy zone servers (`multiplayer-fabric-zones`) and smoke test | Low | [ ] |

### Cycle 1 — zone-backend Docker image

Add a `Dockerfile` to `multiplayer-fabric-zone-backend` and a GitHub Actions workflow that builds and pushes `ghcr.io/v-sekai-fire/multiplayer-fabric-zone-backend:latest` on every push to `main`.

### Cycle 2 — Godot server Docker image

Add a `Dockerfile` to `multiplayer-fabric-deploy` that cross-compiles the headless Godot server binary (aarch64) and packages it. Push `ghcr.io/v-sekai-fire/multiplayer-fabric-godot-server:latest` on every release tag.

### Cycle 3 — CockroachDB fork image

Add a GitHub Actions workflow to `v-sekai/cockroach` that builds from the `release-22.1-oxide` branch and pushes `ghcr.io/v-sekai/cockroach:latest`.

### Cycle 4 — Fly app provisioning

```bash
fly apps create multiplayer-fabric-uro
fly apps create multiplayer-fabric-zones
fly apps create multiplayer-fabric-crdb
fly storage create                          # creates Tigris bucket, outputs credentials
```

### Cycle 5 — CockroachDB

```bash
fly volumes create crdb_data --app multiplayer-fabric-crdb --region yyz --size 80
fly deploy --config multiplayer-fabric-fly/crdb/fly.toml
```

### Cycle 6 — Zone backend

```bash
fly secrets set --app multiplayer-fabric-uro \
  DATABASE_URL="postgresql://root@<crdb-private-host>:26257/production" \
  AWS_S3_BUCKET="<tigris-bucket>" \
  AWS_S3_ENDPOINT="https://fly.storage.tigris.dev" \
  AWS_ACCESS_KEY_ID="<key>" \
  AWS_SECRET_ACCESS_KEY="<secret>"
fly deploy --config multiplayer-fabric-fly/uro/fly.toml
```

### Cycle 7 — Zone servers

```bash
fly deploy --config multiplayer-fabric-fly/zones/fly.toml
# Smoke test: join a zone from the console, upload a minimal scene, instance it
```

## WebTransport Platform Support (modules/http3)

Two backends — `web` uses the browser JS API; `linux` uses picoquic + picotls + mbedtls.

| Platform | Backend                 | Role                     |
| -------- | ----------------------- | ------------------------ |
| `web`    | JS (`quic_web_glue.js`) | Primary — browser client + WebXR |
| `linux`  | picoquic native         | Server                   |

## Zone Console Asset Streaming

Enable `zone_console` to upload a Godot scene to uro, then trigger the
running zone process to stream and instance that scene near the current
player — closing the loop from authoring tool to live world.

### Cycles (Pareto order — highest value/effort ratio first)

| Cycle | What you get                                                      | Effort | Status |
| ----- | ----------------------------------------------------------------- | ------ | ------ |
| 1     | `UroClient.upload_asset/3` — casync chunk → S3 → uro manifest     | Medium | [ ]    |
| 2     | `upload <path>` command — user can store a scene                  | Low    | [ ]    |
| 3     | `CMD_INSTANCE_ASSET` wire encoding — protocol ready               | Low    | [ ]    |
| 4     | `instance <id> <x> <y> <z>` command — user can trigger instancing | Low    | [ ]    |
| 5     | `UroClient.get_manifest/2` — chunk manifest fetch                 | Low    | [ ]    |
| 6     | Godot zone handler — zone actually instances the scene            | High   | [ ]    |
| 7     | Round-trip integration smoke test                                 | High   | [ ]    |

### Cycle 1 — UroClient.upload_asset/3

Add `{:aria_storage, github: "V-Sekai-fire/aria-storage"}` to
`modules/multiplayer_fabric_mmog/tools/zone_console/mix.exs`.

Implement `UroClient.upload_asset/3`:

1. `AriaStorage.process_file(path, backend: :s3)` → `{:ok, %{chunks, store_url}}`
2. POST `/storage` with `{name, chunks, store_url}` + Bearer token
3. Return `{:ok, id}`

Configure S3 in `config/runtime.exs`:

```elixir
config :aria_storage,
  storage_backend: :s3,
  s3_bucket: System.get_env("AWS_S3_BUCKET", "uro-uploads"),
  s3_endpoint: System.get_env("AWS_S3_ENDPOINT", "http://localhost:7070"),
  aws_access_key_id: System.get_env("AWS_ACCESS_KEY_ID"),
  aws_secret_access_key: System.get_env("AWS_SECRET_ACCESS_KEY")
```

### Cycle 2 — `upload <path>` command

Add `"upload"` clause to `handle_line/2` in `app.ex`; call
`UroClient.upload_asset`, display the returned ID.

### Cycle 3 — CMD_INSTANCE_ASSET wire protocol

Add to `fabric_mmog_peer.h`:

```cpp
CMD_INSTANCE_ASSET = 4,
// payload[1] = shared_file_uuid_hi (u32)
// payload[2] = shared_file_uuid_lo (u32)
// payload[3] = pos_x as bit-cast f32 (u32)
// payload[4] = pos_y as bit-cast f32 (u32)
// payload[5] = pos_z as bit-cast f32 (u32)
```

Add `ZoneClient.send_instance/4` — 100-byte packet, 6 payload slots used.

### Cycle 4 — `instance <asset_id> <x> <y> <z>` command

Add `"instance"` clause to `handle_line/2`; parse asset_id and float
coords; call `ZoneClient.send_instance`.

### Cycle 5 — UroClient.get_manifest/2

Add `get_manifest/2` to `UroClient` — POST `/storage/:id/manifest`,
return `{:ok, %{store_url: _, chunks: [_|_]}}`.

### Cycle 6 — Godot zone: handle CMD_INSTANCE_ASSET

In `FabricMMOGPeer::_process_peer_packet`:

- Add `case CMD_INSTANCE_ASSET:` dispatch
- Extract `asset_id` (two u32 slots → UUID) and `pos` (three f32 slots)
- Call `FabricMMOGAsset::fetch_asset` with the uro manifest URL
- On completion: `ResourceLoader::load()` + `Node::instantiate()` at `pos`

`FabricMMOGAsset::fetch_asset` already handles the caibx index + chunk
download + SHA-512/256 verification pipeline.

### Cycle 7 — Round-trip integration smoke test

Requires CockroachDB + VersityGW + uro + zone all running locally.
Upload a minimal `.tscn`, `instance` it, assert the zone entity list
shows a new entry near `pos`.
