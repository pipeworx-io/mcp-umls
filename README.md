# mcp-umls

UMLS MCP — wraps the NLM UMLS Terminology Services REST API (uts-ws.nlm.nih.gov/rest)

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1576+ live data sources.

## Tools

| Tool | Description |
|------|-------------|
| `umls_search` | Search clinical terms/codes → UMLS concepts (CUIs). Maps free text or a source code to UMLS concept unique identifiers spanning SNOMED CT, ICD-10, RxNorm, LOINC, MeSH, etc. Example: umls_search({ query: "myocardial infarction", _apiKey: "your-key" }) |
| `umls_concept` | Concept detail for a CUI. Returns the preferred name, semantic types, atom count, and relation count for a UMLS Concept Unique Identifier. Example: umls_concept({ cui: "C0027051", _apiKey: "your-key" }) |
| `umls_atoms` | Source codes (ICD/SNOMED/RxNorm/LOINC) for a concept. Lists the source-asserted atoms of a CUI — the actual codes and names each vocabulary uses for that concept (the crosswalk table). Example: umls_atoms({ cui: "C0027051", _apiKey: "your-key" }) |
| `umls_crosswalk` | Crosswalk a code from one vocabulary to others. Given a code in a source vocabulary (e.g. an ICD-10-CM code), returns the concepts that share the same UMLS CUI in other vocabularies (SNOMED CT, RxNorm, LOINC, etc.). Example: umls_crosswalk({ source: "ICD10CM", code: "I21.9", _apiKey: "your-key" }) |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "umls": {
      "url": "https://gateway.pipeworx.io/umls/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/umls/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1576+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/umls_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "umls": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-umls"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-umls
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Umls data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
