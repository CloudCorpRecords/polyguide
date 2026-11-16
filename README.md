# polyguide

Finding a polygraph examination office and reading its published prices, without submitting anything. polyguide is a local forwarding client for the hosted LieDetectorTest MCP service: it answers `tools/list` on your own machine from the captured tool definitions and forwards every read to the service, returning the answers exactly as they come back.

Everything here is a published read, plus one status read keyed by a reference you already hold. You can search for offices across the United States, Canada and the United Kingdom, read the all-inclusive price tiers for a location, pull up general information about how the service, booking and payment work, and check the status of an existing booking by its reference. No tool in this package starts a booking or sends anyone's details anywhere.

## The tools

- **`find_offices`** — Offices near a place. Search by city, state or region, or country, and get each matching office with its address, all-inclusive prices and a booking link. `query` is required; `limit` (1–50, default 15) is optional.
- **`get_pricing`** — Price tiers for a location: single person, couple or two-person total, and group per-examinee, plus how payment works. Nothing is paid online; the client pays the examiner directly once the examiner accepts the case. `query` is required.
- **`get_service_info`** — General information: what is offered, how booking works, how payment works, the service area, and what happens after booking. Takes no arguments.
- **`get_booking_status`** — The status and location for an existing booking or appointment request, looked up by its `booking_reference` (for example `LDT-2026072062783`). A status of "request" means the request has been sent to the examiner, who will contact the person directly.

One upstream tool is deliberately not included: the booking-start tool, which would send an appointment request for a named person, with that person's email address, directly to the examiner. It is not registered, not served over MCP, and has no CLI subcommand.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/polyguide mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "polyguide": {
      "command": "npx",
      "args": ["-y", "@reneturcios/polyguide", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/polyguide
docker run --rm ghcr.io/cloudcorprecords/polyguide get_service_info '{}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
polyguide find_offices '{"query":"Miami"}'
polyguide find_offices '{"query":"Florida","limit":5}'
polyguide get_pricing '{"query":"Texas"}'
polyguide get_service_info '{}'
polyguide get_booking_status '{"booking_reference":"LDT-2026072062783"}'
polyguide --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `LIEDETECTORTEST_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://liedetectortest.com/wp-json/ldt-mcp/v1/mcp`. (The override variable is a reconstruction added by this rebuild.)

The LieDetectorTest registry record declares no headers on its remote, so polyguide sends no authentication header of any kind — the reads are keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `polyguide error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured subset, asserts the omitted booking-start tool is absent at registry, MCP and CLI level, exercises the session handshake and its reuse, asserts that no `Authorization` header is sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
