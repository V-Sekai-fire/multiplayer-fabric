# TODO

## Defect-0 plan — http3 module

**Root cause of all CI failures** (run 24933681802): `SESSION_H3_SETTINGS`
removed from the `SessionState` enum in the WebTransport audit but not from
`quic_picoquic_backend.cpp:589`. Fix pushed in `6feefaf0b9`. Run 24934241572
is the first clean run — static checks pass, builds in progress.

**Known TODOs (feature gaps, not defects):**
- `quic_client.h:157` — poll() stub, no picoquic event loop
- `quic_server.h:51` — listen() stub, no UDP bind
- `http3_client.cpp:82-83` — POST/PUT/DELETE not implemented
- `quic_web_backend.cpp:127-128` — bidi streams not implemented in web backend

**Local headless test**: observer.tscn runs clean — no operator_camera errors,
no SESSION enum errors, no Mutex errors. Only expected: FabricClient can't
connect (no zone server) and 1 ObjectDB leak on exit.

- [ ] Wait for run 24934241572 to complete green
- [ ] If any new failures appear, triage per defect below

## CI — verify all branches pass

All 15 branches re-run on 2026-04-25 (multiplayer-fabric only; others cancelled to save quota).
`feat/godot-cpp-build` local build verified PASS (23 s, warm cache).
GHA runs were all previously cancelled due to rapid push concurrency — no failures.

- [ ] Re-run all 15 branches once CI queue is clear and verify green
- [ ] `feat/engine-patches` — static checks + build
- [ ] `feat/module-sqlite` — static checks + build
- [ ] `feat/module-http3` — static checks + build
- [ ] `feat/module-sandbox` — static checks + build
- [ ] `feat/module-keychain` — static checks + build
- [ ] `feat/module-lasso` — static checks + build
- [ ] `feat/module-openvr` — static checks + build
- [ ] `feat/module-speech` — static checks + build
- [ ] `feat/open-telemetry` — static checks + build
- [ ] `feat/module-multiplayer-fabric` — static checks + build
- [ ] `feat/module-multiplayer-fabric-asset` — static checks + build
- [ ] `feat/module-multiplayer-fabric-mmog` — static checks + build
- [ ] `feat/multiplayer-fabric` (assembled) — full CI matrix

## Operator camera (observer.tscn)

`operator_camera.gd` is written and wired into `observer.tscn`.
Tests needed before further work:

- [x] Headless parse check passes — no script errors in operator_camera.gd
- [ ] Run `observer.tscn` in the Godot editor and confirm Camera3D renders orthographic
- [ ] Q/E rotate: twist snaps to 0.0 / 0.25 / 0.5 / 0.75 (cardinal views)
- [ ] Scroll zoom: spring_length and camera.size both change, zoom stays in [10, 60]
- [ ] WASD pan: CameraRig moves; pan speed scales with zoom
- [ ] F key: enters Follow mode on nearest entity; CameraRig lerps toward it
- [ ] Escape: exits Follow mode, returns to Survey
- [ ] Tab: toggles orthographic ↔ perspective projection
- [ ] Remap input actions in `project.godot`:
      - WASD pan → `cam_pan_left/right/fwd/back` (replace ui_* placeholders)
      - F → `cam_follow` (replace `ui_filedialog_show_hidden`)
- [ ] Add operator overlay CanvasLayer (load bars + dot clustering) per `20260425-operator-overlay.md`

## Client strategy (updated)

**Decision**: two clients replace the previous Godot wasm web export:
- Godot native PCVR client — VR presence, jellyfish creation, entity control (xr-grid + OpenXR)
- Three.js WebGPU client — browser observer, operator overlay, no install (20260425-threejs-webgpu-zone-client.md)

Godot wasm32/wasm64 web export is dropped. COOP/COEP service worker, SharedArrayBuffer
build flags, and gescons wasm targets are no longer needed for the client path.

- [ ] Implement Three.js WebGPU client with CH_INTEREST parser (parseInterest 100-byte format)
- [ ] Wire Three.js client WebTransport connection to zone server
- [ ] Port operator camera twist/swing to Three.js OrthographicCamera (20260425-operator-camera-2-5d.md)
- [ ] Add operator overlay canvas (load bars, dot clustering) over Three.js scene

## Web client PoC (superseded — kept for history)

**Completed:**
- [x] `_server_path_callback`: fix `sctx` derivation — prefer `p_path_app_ctx` over
      `p_stream_ctx->path_callback_ctx` (proved by Lean `fixed_sctx` theorem)
- [x] `wt_server_demo.gd`: mirror received packet's `transfer_mode` before `put_packet`
      (default RELIABLE was routing echo to streams; datagrams need UNRELIABLE)
- [x] `wt_browser.spec.ts`: Playwright Chromium test — session connects, datagram echoes,
      PASS in 1.4 s against local `wt_server_demo.gd` echo server
- [x] `playwright.config.ts`: add chromium project; inline HTTP server for secure context
- [x] `gescons` alias: add `threads=yes arch=wasm64` to match GHA web build matrix

**Remaining:**
- [x] Build web export: `cd multiplayer-fabric-godot && gescons target=template_debug`
- [x] Serve web export with COOP/COEP and load in Playwright — confirm threaded wasm64 build initialises (`godot_web_init.spec.ts` PASS 1.1 s)
- [x] Fix `transport_peer.spec.ts` shape mismatch: `data.zones` → `data.shards`
- [x] Fix zone registration test — `last_put_at` on create + `created_at` fix + PubSub rescue
- [x] Wire `/socket/websocket` — added `handle /socket/*` in Caddyfile; 400 not 404
- [x] Write end-to-end Playwright test: `godot_wt_e2e.spec.ts` PASS 3.3 s —
      wasm32 web export loads, GDScript `WebTransportPeer` connects to echo server,
      sends datagram, receives echo (4 C++ bugs fixed to get here)

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

## Zone backend — API versioning

- [x] Move `/api/v1` prefix into Phoenix router — all routes now under
      `scope "/api/v1"` in `router.ex`; Caddy no longer strips the prefix;
      `uro_client.ex` updated; 6/6 QA green on production.

## Zone backend / cluster

- [ ] `ZONE_HOST` is set to `zone-700a.chibifire.com` in `.env` — verify
      WebTransport clients can reach UDP 443 from the public internet
- [ ] Rotate Cloudflare Turnstile keys if they have been exposed
      (`multiplayer-fabric-hosting/.env` has plaintext `TURNSTILE_SECRET_KEY`)

## Submodules

- [ ] `multiplayer-fabric-taskweft` — added `ecto_sql` + `postgrex` to
      `mix.lock`; verify PropCheck suite still passes after dependency bump
