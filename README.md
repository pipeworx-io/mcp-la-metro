# @pipeworx/la-metro

Routes, stops and live service cancellations for Los Angeles County Metro
(LACMTA) bus and rail, from Metro's own open API at `https://api.metro.net`.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

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

Both URLs reach the same gateway and the same 1679+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/lametro_routes \
  -H 'Content-Type: application/json' \
  -d '{"agency":"LACMTA","search":"Wilshire"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/lametro_routes`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

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

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-la-metro
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

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
