# BUILD REPORT — polyguide

Clean-room rebuild of `com.liedetectortest/lie-detector-test` v1.4.0
(remote-only MCP server at
`https://liedetectortest.com/wp-json/ldt-mcp/v1/mcp`), built
2026-10-28 as a forwarding client. Package: `@reneturcios/polyguide`
0.1.0, registry name `io.github.CloudCorpRecords/polyguide`.

## Surface built

Labelled subset: **4 of 5 captured tools built**, one omission (below).

| Tool | Required fields as served |
|------|---------------------------|
| `find_offices` | `query` |
| `get_pricing` | `query` |
| `get_service_info` | *(none — no `required` key)* |
| `get_booking_status` | `booking_reference` |

**OMITTED TOOL — `start_booking`:** the upstream's 5th captured
tool. Its schema requires `name`, `email`, `location` and
`service_type` and its description states it sends an appointment
request for a named person directly to the examiner — a PII
submission / lead-routing tool. It is deliberately omitted: it is
not in the fixture, not in the registered tool literals, not served
in tools/list, and no CLI command or route reaches it. Tests assert
its absence at registry level (not in `TOOL_DEFS`/`TOOL_NAMES`,
`isKnownTool` false, not present in the serialised literals), MCP
level (not in the served tools/list surface, and `handleCallTool`
rejects it with MethodNotFound before any network hop), and CLI
level (invoking it prints usage, exits 2, and its name does not
appear in the usage text).

Each served tool carries exactly the keys `name`, `description`,
`inputSchema` — the fixture carries no `title` key, so the served
tools carry none either, and no inputSchema carries a `$schema`
marker. The tool literals were generated programmatically from
`harness/fixtures/liedetectortest-tools-original.json` by the
scripts at `/tmp/laneb3-polyguide-scratch/gen.py` and
`/tmp/laneb3-polyguide-scratch/gen_forward.py` (parse +
re-serialise, placeholder substitution into `src/forward.ts` from
the template in the same scratch dir) — never retyped — and a
post-generation check (the registration-integrity test suite, plus
the parity harness) confirmed the served literals deep-equal the
fixture's 4 tool objects, in order. Tools are registered via the
MCP SDK low-level Server API (`ListToolsRequestSchema` /
`CallToolRequestSchema` handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on the tools' names,
descriptions, required fields (`find_offices` and `get_pricing`
require `query`; `get_service_info` carries no `required` key;
`get_booking_status` requires `booking_reference`) and property
shapes.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned:
  `https://liedetectortest.com/wp-json/ldt-mcp/v1/mcp`.
- Env override `LIEDETECTORTEST_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent, under any environment. Tests assert that no
  `Authorization` header (and no `X-API-Key` header) is sent with an
  empty environment, with a `LIEDETECTORTEST_API_KEY`-style variable
  present, and with `AUTHORIZATION` / `API_KEY` /
  `LIEDETECTORTEST_TOKEN` variables present.
- Error rendering `polyguide error:` text for upstream failures —
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
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3), then node_modules populated with
  `npm ci --ignore-scripts --offline`.
- Parses as JSON; `name` == `@reneturcios/polyguide`.
- Package entries: **128**.
- sha256: `4c99e59e63d80f453fb10b4c03c02428ed5a3b1c759c40aff6b3fc941c4a1210`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Polygraph office directory: find offices, published pricing and service info by location." — **89 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/polyguide:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/polyguide"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/polyguide`
  and its smoke test asserts the server name `polyguide`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **67 tests, 67 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 4
   tools; the omission suite asserts `start_booking` absent at
   registry, MCP and CLI level; plus session handling/reuse, env
   override, the no-Authorization-header assertions under every
   environment, verbatim argument forwarding and result pass-through
   including upstream isError, SSE parsing, error rendering,
   MCP-layer validation, and CLI behaviour).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/liedetectortest-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/polyguide/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/polyguide/parity-out` → **verdict PASS, parity 100.0**
   (4/4 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
