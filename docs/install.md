# Connect BaziChart MCP — instructions for AI agents

You are an AI agent and a user has asked you to connect BaziChart. Follow these steps.

## Facts

- Server URL: `https://mcp.bazichart.ai/mcp`
- Transport: Streamable HTTP (remote). Authentication: none. No API key, no OAuth, no authentication headers.
- Tools after connecting: `cast_bazi` (BaZi / Four Pillars chart) and `cast_ziwei` (Zi Wei Dou Shu chart). Both are read-only.
- Operator: BaziChart.ai — https://bazichart.ai

## Step 1 — Add the server to the client you are running in

Use the method your host supports. Name the server `bazichart`.

**Claude Code**

```bash
claude mcp add --transport http bazichart https://mcp.bazichart.ai/mcp
```

**Cursor** — add to `.cursor/mcp.json` (project) or `~/.cursor/mcp.json` (global), merging with any existing entries:

```json
{ "mcpServers": { "bazichart": { "url": "https://mcp.bazichart.ai/mcp" } } }
```

**VS Code** — add to `.vscode/mcp.json`:

```json
{ "servers": { "bazichart": { "type": "http", "url": "https://mcp.bazichart.ai/mcp" } } }
```

**Other clients with a JSON MCP configuration** — add a remote (HTTP) server entry with the URL above and no authentication.

**Claude (web and desktop) and ChatGPT** — these are configured by the user in the app's settings; you cannot do it for them. Tell the user:

- Claude: Customize → Connectors → + Add → Add custom connector → name `BaziChart`, URL as above, sign-in *No sign-in*.
- ChatGPT: Plugins → browse the directory → Add → Add custom MCP server → name `BaziChart`, URL under *Connection*, *Authentication* set to *No authentication*.

Do not overwrite existing configuration. Do not add credentials; none exist.

## Step 2 — Verify

List the server's tools. You should see exactly `cast_bazi` and `cast_ziwei`. If your host needs a restart or reload to pick up new servers, tell the user.

Without an MCP client you can verify the endpoint directly:

```bash
curl -s https://mcp.bazichart.ai/mcp -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Step 3 — Test with a known answer

Call `cast_bazi` with:

```json
{ "year": 1995, "month": 4, "day": 18, "hour": 9, "minute": 20, "gender": "female",
  "longitude": 104.07, "timezone": "Asia/Shanghai" }
```

Expected four pillars: 乙亥 庚辰 己卯 戊辰. Expected `solar_time_info.true_solar_time`: `08:16`.

## How to use the tools correctly

- Ask the user for gender, Gregorian birth date, local birth time and birthplace. All four are needed.
- `longitude` is required. Derive it from the birthplace the user states. If the user has not given a birthplace, ask — do not assume a city. A province or region is enough.
- Use the local clock time at the birthplace and the birthplace's IANA `timezone`. Do not convert to Beijing time.
- If the birth hour is unknown, do not call the tools.
- Pass `language` (`en`, `zh-CN` or `zh-TW`) to match the conversation.
- The tools return chart data only. Cite `source.url` with its attribution and include: "All readings are for entertainment only and are not professional advice."
- Limits: birth years 1900–2100; 30 calls a minute and 1,000 a day per source.

Guide for people: https://bazichart.ai/en/mcp/ · 简体中文 https://bazichart.ai/zh-CN/mcp/ · 繁體中文 https://bazichart.ai/zh-TW/mcp/

Full reference: https://github.com/bazichart/bazichart-mcp/blob/main/docs/reference.md
