# JMPR — Pesticide Safety Evaluations (FAO/WHO)

Acceptable Daily Intakes (ADI) and Acute Reference Doses (ARfD) set by the Joint FAO/WHO Meeting on Pesticide Residues — the toxicological assessment that every Codex pesticide limit is derived from.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1573+ live data sources.

## Tools

| Tool | What it answers |
|---|---|
| `jmpr_search` | Find a pesticide by name or fragment |
| `jmpr_chemical` | Full evaluation: ADI, ARfD, CAS, chemical class, use, every meeting's conclusions and report links |
| `jmpr_list_pesticides` | All 382 pesticides JMPR has evaluated |

## The three-pack picture

| Pack | Question |
|---|---|
| **jmpr** (this) | Is this pesticide safe, and at what daily intake? |
| [`codex-mrl`](../codex-mrl) | What residue level is legally permitted in each food? |
| [`jecfa`](../jecfa) | The same two questions for additives and veterinary drugs |

**The gaps between them carry meaning.** Chlorpyrifos is the worked example: Codex *revoked* its residue limits, so `codex-mrl` has no record of it at all — while JMPR still holds the evaluation, and the 2024 entry says why: *the toxicological database was insufficient to establish health-based guidance values*. Neither pack alone tells that story; `codex-mrl` would report a bare absence.

## Coverage

382 pesticides. Evaluations run from the 1960s to the present, newest first.

## Auth

None. Keyless, no registration, no quota.

## Data source

<https://apps.who.int/pesticide-residues-jmpr-database/> — server-rendered HTML, no API. Two shapes: `/Home/Range/All` lists every pesticide, `/pesticide?name=NAME` is one record. **Names are the identifier** — there are no numeric ids, so a name that doesn't match exactly renders an empty shell rather than a 404, which is why both detail lookups resolve against the catalogue first.

## Gotchas

1. **A blank ADI is a finding, not a gap.** The evaluation table has an ADI column that is often empty, and dropping those rows or treating them as "no data" inverts the meaning. Chlorpyrifos 2024 has no ADI *because* the meeting judged the toxicological data insufficient — and the reason sits in the Comments cell of that same row. When a pesticide has been evaluated but carries no ADI anywhere, `jmpr_chemical` says so in `note`.

2. **The newest evaluation is not always the newest ADI.** A re-evaluation can decline to set a value while the previous one still stands, so `current_adi` / `current_adi_year` are tracked separately from `latest_evaluation_year`. For chlorpyrifos those are 2004 and 2024 respectively.

3. **Absence from this database means never evaluated.** That is different from "evaluated and found unsafe" — the `no_match` hints say which, because an agent reading a bare empty result will assume the stronger claim.

4. **Parent compounds and variants are separate records with different ADIs** (CHLORPYRIFOS vs CHLORPYRIFOS-METHYL). A fragment that matches several returns all of them flagged `ambiguous` rather than picking one; an exact name still resolves outright.

5. **Document links are mixed relative and absolute** — `./Document/297` on the WHO host, or a full FAO URL. Both are resolved to absolute.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "jmpr": {
      "url": "https://gateway.pipeworx.io/jmpr/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/jmpr/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1573+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "jmpr": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-jmpr"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-jmpr
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Jmpr data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
