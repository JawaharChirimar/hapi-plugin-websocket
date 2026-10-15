# Fork notice

This is a fork of [rse/hapi-plugin-websocket](https://github.com/rse/hapi-plugin-websocket)
for AgenticTMP. It exists for one reason and should be deleted once that
reason goes away.

## Why

Upstream pins `ws` to exactly `8.18.1`, which carries two advisories:

- [GHSA-58qx-3vcg-4xpx](https://github.com/advisories/GHSA-58qx-3vcg-4xpx) —
  uninitialized memory disclosure (`>=8.0.0 <8.20.1`)
- [GHSA-96hv-2xvq-fx4p](https://github.com/advisories/GHSA-96hv-2xvq-fx4p) —
  memory exhaustion DoS from tiny fragments (`>=8.0.0 <8.21.0`)

AgenticTMP reaches this code from a live route (`locks/connect`, model
locking), so the advisories are exposed in production.

An npm `overrides` entry does not work: npm reports the override as applied and
then installs 8.18.1 anyway, because the parent pins it exactly. OWASP
documents that overriding an exact-pinned parent dependency is unreliable.

Upstream has been asked and has not acted:

- [issue #27](https://github.com/rse/hapi-plugin-websocket/issues/27) — "Use ^
  for dependencies", open since 2024-09-11, no reply
- [PR #26](https://github.com/rse/hapi-plugin-websocket/pull/26) — "Bump ws
  from 8.13.0 to 8.17.1", open since 2024-06-18, not merged

## What changed

One line in `package.json`. No source changes.

    ws: 8.18.1 -> ^8.22.0

`@types/ws` is left at upstream's 8.18.1 — its latest release is 8.18.2, the
types lag the library, and type definitions carry no security weight.

The plugin uses only `new WS.Server()`, `ws.on`, `ws.send`, `ws.close`,
`ws.terminate`, `ws.ping`, `ws.readyState` and `WebSocket.OPEN` — core API that
is unchanged across all of 8.x, and 8.18 to 8.22 is the same major.

## How to retire this fork

Watch issue #27 and PR #26. Once a published `hapi-plugin-websocket` depends on
`ws >= 8.21.0`:

1. In `admin/server/package.json`, point `hapi-plugin-websocket` back at the
   npm registry version.
2. `npm install`, then `npm audit` to confirm the `ws` advisories are gone.
3. Delete this fork.
