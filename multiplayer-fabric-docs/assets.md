# Asset pipeline

Raw assets (GLB, .tscn) are uploaded to `zone-backend`, baked asynchronously
into casync format by a one-shot Docker container, then streamed to zone servers
as a delta-sync chunk set.

## casync format

Assets use the casync content-addressable format implemented by the `AriaStorage`
Elixir library (`aria-storage/` submodule):

- `.cacnk` — a single content-addressed chunk (SHA512/256 hash, compressed)
- `.caidx` — a directory-tree index that references chunks by hash and offset
- `.caibx` — a flat-blob index (alternative to `.caidx`)

Chunks are stored in VersityGW at path `chunks/<ab>/<cd>/<hash>.cacnk` where
`ab` and `cd` are the first two byte-pairs of the hash (similar to Git's object
store layout). This allows zone clients to download only the chunks they are
missing when an asset updates.

## Full pipeline

```
1. zone_console uploads raw asset
   POST /storage  multipart  →  zone-backend:4000

2. zone-backend stores raw file in VersityGW
   PUT versitygw:7070/uro-uploads/<id>
   writes shared_files record  (store_url set, baked_url null)

3. zone-backend spawns baker container (one-shot, exits when done)
   docker run --rm \
     --network multiplayer-fabric-hosting_default \
     -v /tmp/scene-<id>:/scene \
     -e ASSET_ID=<id> \
     -e URO_URL=http://zone-backend:4000 \
     -e VERSITYGW_URL=http://versitygw:7070 \
     multiplayer-fabric-godot-baker:latest

4. baker container
   copies /vsk-project → temp workspace
   godot --editor --headless --quit --path <workspace>
     (loads vsk_importer_exporter EditorPlugin, validates + imports asset)
   AriaStorage.create_chunks(".godot/imported/", compression: :zstd)
     → uploads .cacnk files to versitygw:7070/uro-uploads/chunks/
   AriaStorage.create_index_from_chunks(chunks, format: :caidx)
     → writes <id>.caidx
   PUT <id>.caidx → versitygw:7070/uro-uploads/<id>.caidx
   POST http://zone-backend:4000/storage/<id>/bake
        {baked_url: "http://versitygw:7070/uro-uploads/<id>.caidx"}
   exit 0

5. zone-backend writes baked_url to shared_files record in CockroachDB

6. zone client fetches manifest
   POST /storage/<id>/manifest
   → {store_url, chunks, baked_url}

7. zone server reconstructs .godot/imported/
   fetch .caidx from baked_url
   for each chunk not in local cache:
     GET versitygw:7070/uro-uploads/chunks/<ab>/<cd>/<hash>.cacnk
   write chunks to local import cache
```

## Baker image

The baker image uses a pre-built Godot editor binary (`editor=yes`) so
`vsk_importer_exporter` and its EditorPlugin system can load. Zone servers
use a separate `editor=no` binary (smaller).

The build context requires `vsk-project/` — a minimal copy of
`multiplayer-fabric-baker` (15 MB, stripped of XR/UI/animation addons).

```sh
cd multiplayer-fabric-godot

# Populate build context from multiplayer-fabric-baker
bash baker/populate-vsk-project.sh [../multiplayer-fabric-baker]

docker build -t multiplayer-fabric-godot-baker:latest -f Dockerfile.baker .
```

To regenerate `multiplayer-fabric-baker` when `multiplayer-fabric-rx` changes:

```sh
bash baker/regenerate-baker-project.sh [../multiplayer-fabric-rx] [../multiplayer-fabric-baker]
# commit + push multiplayer-fabric-baker, then update its submodule pointer
```

## Database schema

```sql
ALTER TABLE shared_files ADD COLUMN store_url  TEXT;
ALTER TABLE shared_files ADD COLUMN chunks     JSONB;
ALTER TABLE shared_files ADD COLUMN baked_url  TEXT;
```

Migration: `priv/repo/migrations/20250420000001_add_bake_fields_to_shared_files.exs`

## Relevant source files

| File | Role |
|------|------|
| `lib/uro/controllers/storage.ex` | HTTP handlers: create, manifest, bake |
| `lib/uro/shared_content.ex` | Context: DB queries, set_baked_url |
| `lib/uro/shared_content/shared_file.ex` | Ecto schema + changesets |
| `test/zone_console/uro_client_bake_test.exs` | Integration test (`:prod` tag) |

## Monitoring the baker

```sh
# List baker containers (includes exited)
docker ps -a --filter ancestor=multiplayer-fabric-godot-baker:latest

# View baker logs
docker logs <baker-container-id>

# Confirm .caidx index in VersityGW
AWS_ACCESS_KEY_ID=minioadmin AWS_SECRET_ACCESS_KEY=minioadmin \
  aws --endpoint-url http://localhost:7070 \
  s3 ls s3://uro-uploads/ | grep ".caidx"

# Confirm chunks
AWS_ACCESS_KEY_ID=minioadmin AWS_SECRET_ACCESS_KEY=minioadmin \
  aws --endpoint-url http://localhost:7070 \
  s3 ls s3://uro-uploads/chunks/ --recursive | head -10

# Check baked_url in CockroachDB
docker exec multiplayer-fabric-hosting-crdb-1 \
  /cockroach/cockroach sql --insecure \
  -e "SELECT id, name, baked_url IS NOT NULL FROM vsekai.shared_files \
      ORDER BY inserted_at DESC LIMIT 5;"
```

## Security

The baker container runs on the Docker-internal network
(`multiplayer-fabric-hosting_default`) and cannot reach the public internet.
It authenticates to `zone-backend` using a `BAKER_TOKEN` env var (internal
service token), not a user OAuth token.
