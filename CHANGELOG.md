# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

## [0.4.0] - 2026-04-25

### Added

- `operator_camera.gd` — 2.5D top-down operator camera with twist/swing
  [0, 1] per axis (Survey mode: Q/E snap, scroll zoom, WASD pan; Follow mode:
  Blue Archive smooth entity tracking). Input actions added to `project.godot`.
  See `20260425-operator-camera-2-5d.md`.
- `window.__camera_state` exported via `JavaScriptBridge.eval()` for Playwright
  inspection without a full Godot web build.
- Playwright operator camera test (`operator_camera.spec.ts`) — Layer 1 JS
  simulation 7/7 pass; Layer 2 Godot web export test scaffolded (skipped until
  export built).
- `WasmEquiv.lean` — `vm_deterministic` proves compile-once libriscv VM
  correctness across native and Emscripten WASM hosts.
- ADRs: Three.js WebGPU observer (Stage 1), Three.js player (Stage 2), dual-
  client test, headless test matrix, build cache pipeline, CRIS scoring,
  jellyfish game, jellyfish pass condition, operator overlay, operator camera.

### Changed

- Client strategy: Godot wasm32/wasm64 web export dropped. Two clients replace
  it — Godot native PCVR (VR + entity control) and Three.js WebGPU (browser
  observer + WebXR). See `20260425-threejs-observer.md`.
- `observer.tscn` — `SpectatorRig` replaced by `OperatorRig` with
  `operator_camera.gd`; `StatusHUD` re-parented to new camera path.

### Fixed

- `quic_picoquic_backend.cpp:589` — `SESSION_H3_SETTINGS` reference after the
  enum value was removed in the WebTransport audit (`6feefaf0b9`).
- WebTransport audit fixes: datagram reader exclusive-lock bug (new reader per
  iteration → single reader reuse); `incoming` queue mutex for picoquic cross-
  thread safety; `SESSION_H3_SETTINGS` dead state removed from `SessionState`.
- `quic_web_glue.js` — Lean 4 proofs (`WebTransport.lean`) for state machine
  acyclicity, queue discipline, and reader exclusivity invariant.

## [0.3.0] - 2026-04-25

### Added

- Zone backend `/api/v1` prefix moved into Phoenix router (`scope "/api/v1"`);
  Caddy no longer strips the prefix. 6/6 QA tests green on production.
- Playwright tests:
  - `wt_browser.spec.ts` — browser WebTransport datagram echo PASS 1.4 s
  - `godot_web_init.spec.ts` — threaded wasm64 engine loads PASS 1.1 s
  - `godot_wt_e2e.spec.ts` — Godot web export WebTransportPeer end-to-end
    datagram echo PASS 3.3 s (4 C++ bugs fixed to reach green)

### Fixed

- `_server_path_callback` — prefer `p_path_app_ctx` over
  `p_stream_ctx->path_callback_ctx` (proved by Lean `fixed_sctx` theorem).
- `wt_server_demo.gd` — mirror received packet's `transfer_mode` before
  `put_packet` (default RELIABLE routed echo to streams; datagrams need
  UNRELIABLE).
- `transport_peer.spec.ts` shape mismatch: `data.zones` → `data.shards`.
- Zone registration: `last_put_at` on create, `created_at` field fix, PubSub
  rescue in zone controller.
- Caddyfile: added `handle /socket/*` reverse proxy; socket path was 404.
- `gescons` alias updated: `threads=yes arch=wasm64` to match GHA web build.

## [0.2.0] - 2026-04-25

### Added

- taskweft HRR property tests (67 properties) — all green after fixes.
- `Taskweft.Test.DBHelpers` — auto-detects `multiplayer-fabric-hosting/certs/crdb/`
  TLS certs; replaces hardcoded `sslmode=disable` URL in all three prop test files.
- Module-level Postgrex pool in each prop test module (`setup_all`) — eliminates
  per-`forall`-iteration TLS connection overhead that caused 60 s ExUnit timeouts
  during PropCheck shrinking.

### Fixed

- `Storage.query_all/3` — `Enum.map(nil, ...)` crash on DML statements
  (INSERT/UPDATE/DELETE return `rows: nil` in Postgrex). Fixed with `rows || []`.
- Four transaction rollback tests — `raise "rollback"` inside `rescue _ -> :ok`
  committed instead of rolling back. Fixed with `Postgrex.rollback(conn, :rollback)`.
- `references.bib` — non-ASCII bytes (en dashes, smart quotes, invalid Win-1252
  sequences) replaced with ASCII equivalents.

## [0.1.0] - 2026-04-21

### Added

- Infinite Aquarium (Jellyfish Game) proof-of-concept: zone networking,
  RECTGTN jellyfish behaviour planning, UGC asset pipeline, ReBAC permissions.
- `quic_web_glue.js` — browser WebTransport glue for Godot web export.
- `web_transport_peer.cpp` — `poll_incoming_func` + `_web_poll_incoming()`.
- `SCsub` — `env.Append(JS_LIBS=[...])` fix so JS library reaches the linker.
- manuals ADR corpus: 517 ADRs tagged with filename slug; `tag_adrs.exs`,
  `linkify_adrs.exs`, `audit_tropes.exs`, `fix_bold_bullets.exs` scripts;
  bold-first bullet trope fixed across 104 files (578 labels stripped);
  verbatim template heading replaced with "Design" in 309 files.
