# BUILD REPORT — rhymefind

Clean-room rebuild of `com.double-rhyme/rhymes` v1.0.0
(remote-only MCP server at `https://api.double-rhyme.com/mcp`), built
2026-10-28 as a forwarding client. Package: `@reneturcios/rhymefind`
0.1.0, registry name `io.github.CloudCorpRecords/rhymefind`.

## Surface built

Complete surface: **2 of 2 captured tools built**, no omissions.

| Tool | Required fields as served |
|------|---------------------------|
| `find_rhymes` | `word` |
| `find_rhyming_lines` | `word` |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema`, with the `inputSchema.$schema` marker
(`http://json-schema.org/draft-07/schema#`) exactly as captured.
The tool literals were generated programmatically from
`harness/fixtures/doublerhyme-tools-original.json` by the script at
`/tmp/b43-rhymefind-scratch/gen.py` (parse + re-serialise, placeholder
substitution into `src/forward.ts`) — never retyped — and a
post-generation check (the registration-integrity test suite, plus the
parity harness) confirmed the served literals deep-equal the fixture's
2 tool objects, in order. Tools are registered via the MCP SDK
low-level Server API (`ListToolsRequestSchema` /
`CallToolRequestSchema` handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on the tools' names, titles,
descriptions, required fields (`word` for both `find_rhymes` and
`find_rhyming_lines`) and property shapes.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://api.double-rhyme.com/mcp`.
- Env override `DOUBLERHYME_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent, under any environment. Tests assert that no
  `Authorization` header (and no `X-API-Key` header) is sent with an
  empty environment, with a `DOUBLERHYME_API_KEY`-style variable
  present, and with `AUTHORIZATION` / `API_KEY` / `DOUBLERHYME_TOKEN`
  variables present.
- Error rendering `rhymefind error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP JSON-RPC:
  initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; plain-JSON and SSE replies are both parsed. The
  forwarding core takes an injectable fetch; tests stub the
  upstream — no live network calls in tests.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`.
- Parses as JSON; `name` == `@reneturcios/rhymefind`.
- Package entries: **128**.
- sha256: `993acd0e419d4b8ecdecbce013247f6ee36fa126b9bf6cdf4cc4b6bd635dccb9`
  (re-verified unchanged after the subsequent offline `npm install`).

## server.json

- Description exactly: "Find rhymes and rhyming lines for any word,
  from the Double Rhyme dictionary." — **77 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/rhymefind:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/rhymefind"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/rhymefind`
  and its smoke test asserts the server name `rhymefind`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **60 tests, 60 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 2
   tools, plus session handling/reuse, env override, the
   no-Authorization-header assertions under every environment,
   verbatim argument forwarding and result pass-through including
   upstream isError, SSE parsing, error rendering, MCP-layer
   validation, and CLI behaviour).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/doublerhyme-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/rhymefind/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/rhymefind/parity-out` → **verdict PASS, parity 100.0**
   (2/2 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
