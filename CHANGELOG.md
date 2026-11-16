# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted LieDetectorTest MCP server. The four captured subset tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `find_offices`, `get_pricing`, `get_service_info`, `get_booking_status` — a labelled subset, 4 of the 5 captured tools. The fifth captured tool, `start_booking`, is deliberately omitted because it sends an appointment request for a named person (name, email, location, service_type) directly to the examiner; it is not registered, not served, and has no CLI route.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `polyguide error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `LIEDETECTORTEST_MCP_URL`. The registry record declares no headers on the remote, so no auth header is ever sent.
