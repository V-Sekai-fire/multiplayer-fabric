# Architecture

## Service topology

```
Internet
  │
  ▼
Cloudflare edge  (TLS termination, HTTP/1.1 + HTTP/3)
  │  hub-700a.chibifire.com → Cloudflare Tunnel
  ▼
cloudflared  (Docker, tunnel token in .env)
  │  routes hub-700a.chibifire.com → http://zone-backend:4000
  ▼
zone-backend:4000  (Phoenix/Uro, Docker)
  ├── crdb:26257        CockroachDB single-node, ghcr.io/v-sekai/cockroach
  └── versitygw:7070    S3-compatible object store (local POSIX backend)

zone-700a.chibifire.com
  │  DNS A record → host public IP 173.180.240.105
  │  Router forwards UDP 443 → host machine
  ▼
zone-server:443/udp  (Godot headless, Docker, editor=no build)
  └── WebTransport / QUIC / picoquic — NOT proxied by Cloudflare
```

All services run in the same Docker Compose project on one host machine.

## Key design decisions

### Cloudflare Tunnel for HTTP, direct UDP for WebTransport

Cloudflare Tunnel carries all `hub-700a.chibifire.com` HTTP traffic. TLS
terminates at the Cloudflare edge; the tunnel delivers plain HTTP/1.1 to
`zone-backend:4000` on the Docker-internal network.

WebTransport uses QUIC over UDP. Cloudflare does not proxy UDP, so
`zone-700a.chibifire.com` is a plain DNS A record (orange cloud off) pointing
directly to the host machine. The router forwards UDP 443 to the Docker zone
server port. Clients pin the zone server's self-signed certificate using
`ZONE_CERT_HASH_B64`.

### Shard vs zone distinction

A **shard** is a running WebTransport game server (e.g. `zone-700a.chibifire.com:443`).
A **zone** is a Hilbert-range spatial partition *within* a shard. One shard can contain
many zones; a shard cannot be nested inside another shard.

Zone servers register themselves in CockroachDB via `POST /shards` on boot, then
send `PUT /shards/:id` heartbeats every ~25 s. `ShardJanitor` culls entries with no
heartbeat in 30 s.

### Authority and interest

The zone whose Hilbert range contains `hilbert3D(pos)` is the authority for any
entity at that position. It is the only zone that executes `CMD_INSTANCE_ASSET`.
Neighbouring zones within `AOI_CELLS` receive a `CH_INTEREST` ghost — they do
not re-fetch or re-instance.

### ReBAC access control

The authority zone evaluates `rebacCheck` before instancing. `observe` permission
is public; `modify` requires `owner`. Access policies are stored in CockroachDB
and evaluated by the zone server C++ module.

### Headless asset baking

Zone servers carry no editor code (`editor=no` build). Asset baking runs as a
one-shot Docker container using a Godot `editor=yes` binary. See [Asset pipeline](assets.md).

### casync object storage

Assets are stored in casync format in VersityGW:
- Content-addressed `.cacnk` chunk files (SHA512/256 hash, path `chunks/ab/cd/<hash>.cacnk`)
- `.caidx` directory-tree index that references the chunks

Zone clients reconstruct `.godot/imported/` by fetching the `.caidx` index then
downloading only the missing `.cacnk` chunks. The `AriaStorage` Elixir library
implements both the uploader (baker side) and reader (zone-console side).

### Lean 4 proof authority

All physics, geometry, and algorithmic invariants (Hilbert curve, BVH, interest
management) are formally proved in `multiplayer-fabric-predictive-bvh`. C++ and
Elixir ports must follow the proof, not the other way around. Never hand-edit
`predictive_bvh.h` — regenerate with `lake exe bvh-codegen`.

## Data model

```
users
  id (binary_id)
  email, username, display_name
  ↑ belongs_to
user_identities
  provider ("pow_assent" | "clerk" | …)
  uid (provider-assigned subject)
  ↑ joins to users

shared_files
  id (binary_id)
  name, tags[], is_public
  store_url     raw upload location (VersityGW)
  chunks        [{hash, offset, length}] jsonb
  baked_url     .caidx index URL (set after baker completes)
  uploader_id → users

shards
  id (binary_id)
  address, port
  map, name
  current_users, max_users
  cert_hash     SHA-256 fingerprint of zone server TLS cert
  inserted_at, updated_at  (updated_at used for heartbeat freshness)
```
