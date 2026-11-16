# rhymefind

A rhyme dictionary that lives upstream, reached through one small bridge. rhymefind is a local forwarding client for the hosted Double Rhyme MCP service: it answers `tools/list` on your own machine from the captured tool definitions and forwards every lookup to Double Rhyme, returning the results exactly as the service sends them back.

The surface is a read-only dictionary lookup across 50+ languages: rhymes for a word or short phrase, and real lines — lyrics, proverbs, sayings — that end with a rhyme on a given word. Nothing is cached, stored or computed locally — the package is a pipe, not a dictionary.

## The tools

- **`find_rhymes`** — Find rhymes for a word or short phrase. `word` is required. The `type` parameter picks the rhyme kind: `"double"` (multisyllabic rhymes, best for rap and poetry, the default), `"classic"` (perfect end rhymes grouped by matching syllables) or `"vowel"` (assonance — same vowel sounds). `language` selects among 50+ language codes (default `"en"`), `vocabulary` chooses `"general"` or `"rap"` sources, and `limit` caps the results per group (1–50, default 30). Strongest results come first.
- **`find_rhyming_lines`** — Find real lines (lyrics, proverbs, sayings) that end with a rhyme on the given word — useful as inspiration for the next line of a verse. `word` is required; `language`, `vocabulary` (`"general"` proverbs and general texts, or `"rap"` lyrics) and `limit` (1–50, default 15) work as above.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/rhymefind mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "rhymefind": {
      "command": "npx",
      "args": ["-y", "@reneturcios/rhymefind", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/rhymefind
docker run --rm ghcr.io/cloudcorprecords/rhymefind find_rhymes '{"word":"heart"}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
rhymefind find_rhymes '{"word":"heart","type":"double"}'
rhymefind find_rhymes '{"word":"verloren","language":"de"}'
rhymefind find_rhyming_lines '{"word":"night","limit":5}'
rhymefind --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `DOUBLERHYME_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://api.double-rhyme.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The Double Rhyme registry record declares no headers on its remote, so rhymefind sends no authentication header of any kind — lookups are keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `rhymefind error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface, exercises the session handshake and its reuse, asserts that no `Authorization` header is sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
