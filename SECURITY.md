# Security policy

The Lekta MCP server is a hosted service at `https://lekta.dev/mcp`. This repository carries documentation only, so there are no dependencies or builds to patch here.

## Reporting a vulnerability

Write to **hello@lekta.dev** with a description and, if possible, a reproduction. Reports go directly to the people who build the engine — there is no ticket queue. Please do not open public issues for security findings before we have had a chance to respond.

## Key handling

- API keys (`lekta_…`) are shown once at generation; only a hash is stored server-side.
- Rotate a leaked key immediately at [lekta.dev/en/panel/api](https://lekta.dev/en/panel/api) — the old key stops working at once.
- The server never asks for your key anywhere except the `Authorization` header of `https://lekta.dev/mcp` and `https://lekta.dev/api/v1/*`.
