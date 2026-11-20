# @pipeworx/norway-law

The text of the consolidated current Norwegian acts (about 750 lover), by act and section, with the in-force facts the publication carries. Find an act by title, short title, abbreviation or the number Norwegians cite it by, then read the act or one section (`§ 275`, `5-3`, or a constitutional article).

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2015+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `no_law_search(query, limit?)` finds acts by title, short title, abbreviation or law number ("straffeloven", "mrl", "lov 20 mai 2005 nr 28", "2005-05-20-28"). Returns the act id, title, short title, entry-into-force date, date of the last change in force and a citation. Titles only, not act text.
- `no_law_get(id, section?, max_chars?)` reads one act or one section. A section comes back with its text, where it sits in the act, its amendment history with entry-into-force dates, and a citation ("Lov 20. mai 2005 nr. 28 (Straffeloven) § 275"). Without a section it returns the contents list and the opening text.

```
no_law_search({query: "straffeloven"})
  -> 2005-05-20-28, Lov om straff (straffeloven), in force 2015-10-01
no_law_get({id: "2005-05-20-28", section: "§ 275"})
  -> "Den som dreper en annen, straffes med fengsel fra 8 inntil 21 år."
     amendments: Tilføyd ved lov 19 juni 2009 nr. 74.
```

## Auth

Keyless.

## Data sources

- Lovdata, the consolidated current Norwegian acts (gjeldende lover), published as open data. Every answer carries `data_as_of` (the date Lovdata last rebuilt the release), `answered_from` (always `stored_copy`), the licence and the attribution line.

Lovdata's public site does not serve act text to a non-browser client, but it publishes every current act as open data in one small release, so a stored copy of that release answers every call and a weekly job refreshes it. There is no live fallback: before the first load, or if the copy cannot be read, both tools return `found: false, reason: "stored_copy_unavailable"` rather than an empty result.

### Licence: NLOD 2.0

The release is published under the Norwegian Licence for Open Government Data 2.0 (https://data.norge.no/nlod/en/2.0). The terms that decide this pack, quoted from the licence text:

> The licensee, subject to the limitations that follow from this licence, may use the information for any purpose and in all contexts, by: copying the information and distributing the information to others, modifying the information and/or combining the information with other information, and copying and distributing such changed or combined information. This is a non-exclusive, free, perpetual and worldwide licence.

> The licensee shall attribute the licensor as specified by the licensor and include a reference to this licence. To the extent practically possible, the licensee shall provide a link to both this licence and the source of the information. [...] If the information has been changed, the licensee must clearly indicate that changes have been made by the licensee.

Lovdata states the same on its open-data page (translated from the Norwegian): the consolidated acts and central regulations "may be used freely for all purposes without charge", licensed under NLOD 2.0, with no user account or authentication required. Redistribution is permitted, so a stored copy is allowed. Every response therefore carries `licence` and `attribution` ("Contains data under the Norwegian licence for Open Government data (NLOD) distributed by Lovdata", the licence link, the source, and the note that the markup was converted to plain text).

## Caveats worth passing on

- **Lovdata's consolidated text is not an official version.** Norway has no official consolidated versions; the original announcements in Norsk Lovtidend are authoritative. Every act and section answer says so in `disclaimer`.
- **Norwegian only.** The text is in Norwegian (Bokmål). The Constitution is also stored in Nynorsk as `1814-05-17-nn`. Some acts carry annexed foreign-language instruments (the human rights convention, for example) in the original language.
- **Acts only.** Central regulations (forskrifter), local regulations and the Norsk Lovtidend announcements are not covered. The central regulations are a separate release about four times the size of the acts (about 5,150 documents), left out of this build; an `unknown_id` answer says a regulation is one reason an id may be missing.
- **Search is by title, short title, abbreviation and number, not by act text.** "personvern" finds nothing because no act title contains it; "personopplysninger" finds the Personal Data Act. Search for the subject word as it appears in a title.
- **In-force dates are per act, not per section.** `in_force_from` can be a list (an act that entered into force in stages) and `last_change_in_force` is the most recent amendment taking effect. A section shows its own amendment history, with the dates the changes entered into force, in `amendments`; a repealed section carries `repealed_date`.
- **Sections are whole sections.** The publication is not split below the section, so "§ 5-3 første ledd" returns all of § 5-3 and says so. Several sections can share a number (annexes repeat numbering); the first is returned and the rest are listed in `other_matches`.
- **Text is cut at `max_chars`** (default 20,000, max 100,000), and the answer says how long the full text is.
- **A miss is explicit.** An unknown act id returns `found: false, reason: "unknown_id"`; an unknown section returns the act with `section: null, empty_reason: "section_not_found"` and the sections that do exist.

## Related

- `denmark-law` for Danish statutes
- `eur-lex` for EU legislation that Norway implements through the EEA agreement

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "norway-law": {
      "url": "https://gateway.pipeworx.io/norway-law/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/norway-law/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2015+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/no_law_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"straffeloven","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/no_law_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-norway-law` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/norway-law/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "norway-law": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-norway-law"]
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
ask_pipeworx({ question: "your question about Norway Law data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
