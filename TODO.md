# TODO

## CI — verify all branches pass

All 13 branches have runs queued as of 2026-04-24. Once results land:

- [ ] `feat/engine-patches` — static checks + build
- [ ] `feat/module-sqlite` — static checks + build
- [ ] `feat/module-http3` — static checks + build
- [ ] `feat/module-sandbox` — static checks + build (Win32 gitignore fix queued)
- [ ] `feat/module-keychain` — static checks + build
- [ ] `feat/module-lasso` — static checks + build
- [ ] `feat/module-openvr` — static checks + build
- [ ] `feat/module-speech` — static checks + build
- [ ] `feat/open-telemetry` — static checks + build
- [ ] `feat/module-multiplayer-fabric` — static checks + build
- [ ] `feat/module-multiplayer-fabric-asset` — static checks + build
- [ ] `feat/module-multiplayer-fabric-mmog` — static checks + build
- [ ] `feat/multiplayer-fabric` (assembled) — full CI matrix

## WebTransport interop test

Add a Python WebTransport client test using `uv` + `aioquic` that connects to
the in-process picoquic echo server on loopback and verifies a datagram echo.
Blocked on understanding the echo server TLS cert hash (self-signed ECDSA) for
`aioquic`'s `verify_mode=ssl.CERT_NONE` or cert-pinning path.

File to add: `modules/http3/tests/wt_python_client.py`
Test to add: `[WebTransportPeer] Python aioquic client echoes datagram` in
`modules/http3/tests/test_web_transport_peer.h`

## Branch maintenance

- [ ] Archive `feat/multiplayer-fabric` once the assembled branch is stable —
      it's now superseded by the split branches + gitassembly composition
- [ ] Add `feat/ci-infra` as a separate branch (currently CI/AGENTS.md changes
      are on `feat/engine-patches`; splitting them would allow engine and CI
      changes to be reviewed independently)

## multiplayer-fabric-merge

- [ ] Run `elixir update_godot_v_sekai.exs` (live push) once all branch CI is
      green to update the canonical `multiplayer-fabric` branch on the remote
- [ ] Add dry-run CI job to `multiplayer-fabric-merge` that runs
      `git-assembler --dry-run` on every push to `main`, so assembly regressions
      are caught automatically

## Zone backend / cluster

- [ ] `ZONE_HOST` is set to `zone-700a.chibifire.com` in `.env` — verify
      WebTransport clients can reach UDP 443 from the public internet
- [ ] Rotate Cloudflare Turnstile keys if they have been exposed
      (`multiplayer-fabric-hosting/.env` has plaintext `TURNSTILE_SECRET_KEY`)

## Submodules

- [ ] `multiplayer-fabric-taskweft` — added `ecto_sql` + `postgrex` to
      `mix.lock`; verify PropCheck suite still passes after dependency bump
