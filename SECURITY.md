# Security

BaziChart MCP is a hosted service. This repository contains documentation only.

## Reporting a vulnerability

Email **support@bazichart.ai** with the subject line `Security`. Please include what you observed, the request that triggered it, and the time (UTC). Do not open a public issue for a suspected vulnerability.

## Scope

- The MCP endpoint `https://mcp.bazichart.ai/mcp`
- The website `https://bazichart.ai`

Please do not run load tests or automated scanners against the service. The published rate limits (30 chart calls a minute, 1,000 a day per source) apply to everyone.
