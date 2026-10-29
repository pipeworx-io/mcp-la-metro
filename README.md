# @pipeworx/la-metro

Routes, stops and live service cancellations for Los Angeles County Metro
(LACMTA) bus and rail, from Metro's own open API at `https://api.metro.net`.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

**Keyless.** No registration, no key, no quota published.

## Tools

| Tool | What it answers |
|---|---|
| `lametro_routes` | The bus or rail route roster, with route codes |
| `lametro_route_stops` | Every stop on a route, in running order |
| `lametro_stop` | One stop's location and attributes |
| `lametro_service_cancellations` | Trips canceled today, per route (**live**) |

## LA Metro has no public GTFS-Realtime feed — read this before adding a tool

The commonly cited `api.metro.net/LACMTA/vehicle_positions`, `/trip_updates`
and `/alerts` paths are **gone**; all three return HTTP 404 (verified
2026-09-17). The Mobility Database lists LACMTA with static GTFS only and no
realtime row at all.

The v2 API still advertises a "Real-Time data" tag over
`/{agency}/trip_detail/...`, and that is the trap worth knowing about: those
endpoints answer **HTTP 200 with `[]` for every route**, on both agencies, in
the middle of a weekday afternoon. Fourteen bus and rail routes were swept at
15:00 PDT and every one came back empty. A tool built on them would pass every
CI gate and quietly tell callers that LA Metro is not running any buses — the
silent-zero failure class in `docs/silent-zero-policy.md`.

`canceled_service_summary` **is** live operational data and is wired up. It is
the one real-time signal LACMTA publishes publicly, and it answers a question
no schedule or GTFS feed can: *is my line dropping trips today?*

## Agencies

`LACMTA` is the bus agency (~124 routes); `LACMTA_Rail` is rail and busway
(A/B/C/D/E/K lines). The API returns HTTP 422 on any other value.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "la-metro": {
      "url": "https://gateway.pipeworx.io/la-metro/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/la-metro/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/lametro_routes \
  -H 'Content-Type: application/json' \
  -d '{"agency":"LACMTA","search":"Wilshire"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/lametro_routes`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-la-metro` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/la-metro/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "la-metro": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-la-metro"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about La Metro data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
