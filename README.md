# @pipeworx/dchub

Data-center site selection, power and grid intelligence from DC Hub: facility registry, per-metro market intelligence, live grid fuel mix and demand, leasable capacity, and a one-call planner for multi-part siting questions.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1882+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `dchub_execute_plan(intent, ...)` — answers a multi-part data-center question in one call (e.g. "rank markets for a 200 MW AI campus", "power headroom in PJM") and returns the combined result with citations.
- `dchub_search_facilities(city, state, country, operator, min_capacity_mw, ...)` — which data centers exist and who operates them.
- `dchub_get_market_intel(market)` — vacancy, pricing, pipeline and operators for one metro, by market slug (e.g. `northern-virginia`).
- `dchub_get_grid_data(iso)` — live fuel mix and demand for a grid (PJM, ERCOT, CAISO and others).
- `dchub_source_capacity(min_mw, region, ...)` — powered land, shells, turnkey capacity and colocation available to buy or lease.

## Auth

Keyless. Without a DC Hub key, DC Hub returns a trimmed PREVIEW for some tools (a few rows per list, no MW detail); every response says so in `preview` / `preview_notice`. A caller may pass their own DC Hub key as `_apiKey` for fuller answers. A Pipeworx-wide partner key is not configured yet.

## Data sources

- DC Hub (dchub.cloud), data-center intelligence platform: facility registry, Data Center Power Index and market intelligence, grid feeds, capacity listings. Cite as "DC Hub, dchub.cloud". Live per-request; nothing is stored by Pipeworx.

DC Hub asked that partner traffic be identifiable; every call carries `X-DCHub-Via: pipeworx` (changes nothing in access). Results are JSON wrapped in `{source, preview, result}`; `result.provenance` carries DC Hub's own completeness and as-of fields, and `as_of` may be null: do not cite `retrieved_at` as a data date. `dchub_execute_plan` responses are large (tens of KB).

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "dchub": {
      "url": "https://gateway.pipeworx.io/dchub/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/dchub/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1882+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/dchub_execute_plan \
  -H 'Content-Type: application/json' \
  -d '{"intent":"rank markets for a 200 MW AI campus"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/dchub_execute_plan`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "dchub": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-dchub"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-dchub
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Dchub data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
