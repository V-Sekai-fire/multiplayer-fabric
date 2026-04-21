# Shards

A **shard** is a running WebTransport zone server. It registers itself in
CockroachDB on boot, sends heartbeats to stay listed, and is culled automatically
when heartbeats stop.

## Shard vs zone

A shard is the top-level WebTransport server (e.g. `zone-700a.chibifire.com:443`).
A zone is a Hilbert-range spatial partition *inside* a shard. One shard can host
many zones; shards cannot be nested.

## Self-registration flow

```
zone server boots
  → POST /shards  {address, port, map, name, cert_hash}
  → zone-backend writes row to shards table
  → every ~25 s: PUT /shards/:id  (touches updated_at)

ShardJanitor (zone-backend GenServer)
  → runs every 10 s
  → deletes shards where updated_at < now - 30 s
```

## cert_hash

`cert_hash` is the base64-encoded SHA-256 fingerprint of the zone server's
self-signed TLS certificate. Clients pin this value to authenticate the zone
server connection without a CA chain.

The value is supplied by the zone server in its `POST /shards` registration
payload. It is exposed to clients via `GET /shards` so `zone_console` can
pass it to the WebTransport connection.

`ZONE_CERT_HASH_B64` in `multiplayer-fabric-hosting/.env` pre-seeds the cert
hash when the zone server container starts before it has registered itself.

## Database schema

```sql
CREATE TABLE shards (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id      UUID REFERENCES users(id),
  address      TEXT NOT NULL,
  port         INTEGER NOT NULL,
  map          TEXT NOT NULL,
  name         TEXT NOT NULL,
  current_users INTEGER DEFAULT 0,
  max_users     INTEGER DEFAULT 32,
  cert_hash     TEXT,
  inserted_at   TIMESTAMPTZ NOT NULL,
  updated_at    TIMESTAMPTZ NOT NULL
);
```

Migration: `priv/repo/migrations/20250420000000_add_cert_hash_to_shards.exs`

## Freshness query

```elixir
# lib/uro/v_sekai.ex
def list_fresh_shards do
  cutoff = DateTime.utc_now() |> DateTime.add(-30, :second)
  from(s in Shard, where: s.updated_at > ^cutoff)
  |> Repo.all()
  |> Repo.preload([:user])
end
```

## Relevant source files

| File | Role |
|------|------|
| `lib/uro/v_sekai/shard.ex` | Ecto schema + changeset + `to_json_schema/1` |
| `lib/uro/v_sekai/shard_janitor.ex` | GenServer that culls stale shards |
| `lib/uro/v_sekai.ex` | Context: `list_fresh_shards`, CRUD |
| `lib/uro/controllers/shard_controller.ex` | HTTP handlers |

## Cloudflare DNS note

`zone-700a.chibifire.com` must have the **orange cloud disabled** in the
Cloudflare dashboard (DNS-only, not proxied). Cloudflare cannot proxy QUIC
UDP traffic. The DNS A record points directly to the host machine's public IP.

```sh
# Confirm DNS resolves to host machine (not Cloudflare)
dig zone-700a.chibifire.com +short
# expect: 173.180.240.105

# Confirm UDP 443 is reachable
nc -u -w2 zone-700a.chibifire.com 443 && echo "UDP open"
```
