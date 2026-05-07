# plans

JSON-LD HTN domains feedable to taskweft. Each plan is self-contained — a
domain definition with state variables, primitive actions, and HTN methods —
that can be loaded via:

```sh
cd taskweft
mix run -e '
domain = File.read!("../plans/zone_lifecycle.jsonld")
{:ok, plan} = Taskweft.plan(domain)
Jason.decode!(plan) |> Enum.each(&IO.inspect/1)
'
```

Format mirrors the `taskweft/bench/ipyhop/*.jsonld` benchmark suite: a
`domain:Definition` with `variables` / `actions` / `methods` / `tasks`.
Production domains (`work_queue`, `adventurer`) live inside the taskweft
submodule under `priv/plans/`. This folder is for reusable examples and
multi-submodule scenarios that don't belong to one component.

## Examples

- [`zone_lifecycle.jsonld`](zone_lifecycle.jsonld) — boot sequence of a
  multiplayer-fabric zone server: register with uro, start the WebTransport
  listener, enter the heartbeat loop. Three actions, one method, one task.
