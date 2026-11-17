# BaziChart MCP

**BaZi (Four Pillars) and Zi Wei Dou Shu charts for AI assistants — calculated, not guessed.**

[English](README.md) · [简体中文](README.zh-CN.md) · [繁體中文](README.zh-TW.md)

BaziChart MCP is a hosted [Model Context Protocol](https://modelcontextprotocol.io) server operated by [BaziChart.ai](https://bazichart.ai). It gives an AI assistant a deterministic chart engine to call, so that solar-term boundaries, true solar time, daylight-saving years and luck-cycle dates are computed rather than recalled.

```
https://mcp.bazichart.ai/mcp
```

Add this URL to any MCP client that supports remote servers. No installation, API key or account is needed.

You give your assistant a birth date, time and place. It derives the longitude and time zone, calls the server and receives the chart as structured data.

## At a glance

| Item | Detail |
|---|---|
| Transport | Streamable HTTP (stateless, JSON responses) |
| Authentication | None |
| Tools | `cast_bazi`, `cast_ziwei` — both read-only |
| Input | Gregorian birth date, local clock time, gender, birthplace longitude and IANA time zone |
| Output | Structured chart data, the time-correction record, the conventions used, and a source link |
| Birth years | 1900–2100 |
| Rate limit | 30 chart calls per minute and 1,000 per day, per IP address |
| Scope | Chart calculation only. No readings, no predictions |

## Quick start

### Claude (claude.ai and desktop)

Customize → Connectors → **+ Add** → **Add custom connector**. Name it `BaziChart`, paste the URL, leave sign-in as *No sign-in*, then **Add**. Enable it from the **+** menu in a chat.

### ChatGPT

Plugins → browse the directory → **Add** → **Add custom MCP server**. Name it `BaziChart`, paste the URL under *Connection*, and set *Authentication* to **No authentication**. Create and install it, then type `@BaziChart` in a new chat. After a server update, open the plugin's Settings → *Manage app* → **Refresh tools**.

### Claude Code

In a terminal:

```bash
claude mcp add --transport http bazichart https://mcp.bazichart.ai/mcp
```

Or install the plugin, which connects the server and adds the [BaziChart skill](#agent-skill) in one step. Type this at the prompt inside a Claude Code session, not in the terminal:

```
/plugin install bazichart --marketplace bazichart/bazichart-mcp
```

On versions that do not accept `--marketplace`, run the two steps separately:

```
/plugin marketplace add bazichart/bazichart-mcp
/plugin install bazichart@bazichart
```

### Cursor

Add to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "bazichart": { "url": "https://mcp.bazichart.ai/mcp" }
  }
}
```

### VS Code

Add to `.vscode/mcp.json`:

```json
{
  "servers": {
    "bazichart": { "type": "http", "url": "https://mcp.bazichart.ai/mcp" }
  }
}
```

### Let your agent do it

Point an agent that can read web pages at [`docs/install.md`](docs/install.md) and ask it to connect BaziChart.

### Check the endpoint

```bash
curl -s https://mcp.bazichart.ai/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Try it

Once connected, ask in plain language:

> Cast my BaZi chart. I'm female, born 18 April 1995 at 9:20 a.m. in Chengdu.

> Cast a Zi Wei Dou Shu chart for a man born 3 October 1988 at 11:45 p.m. in Taipei.

> I was born in New York on 7 July 1990 at 6:10 a.m. What are my four pillars and my current luck cycle?

> Cast a BaZi chart for a man born 12 May 1995 at 11:30 p.m. in Shanghai, then recalculate it with the day changing at midnight instead of 23:00.

If the birthplace or birth time is missing, the assistant asks for it.

## How it calculates

**True solar time by default.** BaZi charts are corrected for birthplace longitude, the equation of time and daylight saving, including China's 1986–1991 daylight-saving years. Each response includes the correction record: every adjustment in minutes, the resulting true solar time, and whether the double-hour changed.

**Birthplace is required.** Without a longitude the server returns an error (`BIRTHPLACE_LONGITUDE_REQUIRED`, or `OVERSEAS_LONGITUDE_REQUIRED` from `cast_ziwei` for time zones other than `Asia/Shanghai`) and tells the assistant to ask the user. The correction can change the chart: a 09:20 birth in Chengdu is 08:16 true solar time, which moves the hour pillar from 己巳 to 戊辰.

**Two conventions are selectable.** Schools differ on two boundaries. Each is a parameter with a documented default, and every response states which was applied.

| Question | Default | Alternative |
|---|---|---|
| BaZi: a birth at 23:00–23:59 true solar time belongs to which day? | Next day (`zi_hour_23`) | Same day (`midnight_00`) |
| Zi Wei: when does the year stem-branch change? | Lunar New Year (`lunar_new_year`) | Start of Spring (`lichun`) |

**Local time, local time zone.** Pass the clock time and IANA time zone of the birthplace. Hong Kong, Macao and Taiwan have their own zones. Times are not converted to Beijing time.

**Chart data only.** The tools return the chart structure and do not interpret it. For a given engine build, the same input returns the same chart.

**Fixed engine build.** The engine build is pinned. Its identity is checked at startup and reported at `/healthz`.

## Tools

### `cast_bazi`

Calculates a BaZi chart from a known local birth time and birthplace.

| Parameter | Required | Description |
|---|---|---|
| `year`, `month`, `day` | yes | Gregorian birth date. Years 1900–2100 |
| `hour` | yes | Local clock hour, 0–23. Do not call if the birth hour is unknown |
| `minute` | no | Local clock minute. Default `0` |
| `gender` | yes | `male` / `female`, `M` / `F`, or `男` / `女` |
| `longitude` | yes | Birthplace longitude in degrees, east positive |
| `timezone` | yes | Birthplace IANA time zone, e.g. `Asia/Shanghai`, `America/New_York` |
| `day_boundary` | no | `zi_hour_23` (default) or `midnight_00` |
| `language` | no | `en` (default), `zh-CN` or `zh-TW`. Selects the language of the source link only |

Returns the four pillars, ten gods, hidden stems, Na Yin, earthly-branch relationships (combinations, clashes, punishments, harms), ten luck cycles with exact transition times, and the time-correction record.

### `cast_ziwei`

Calculates a Zi Wei Dou Shu natal chart.

| Parameter | Required | Description |
|---|---|---|
| `year`, `month`, `day` | yes | Gregorian birth date. Years 1900–2100 |
| `hour` | yes | Local clock hour, 0–23 |
| `minute` | no | Local clock minute. Default `0` |
| `gender` | yes | As above |
| `longitude` | yes | Birthplace longitude in degrees, east positive |
| `timezone` | no | Birthplace IANA time zone. Default `Asia/Shanghai` |
| `year_boundary` | no | `lunar_new_year` (default) or `lichun` |
| `language` | no | As above |

Returns the Life Palace and Body Palace, the twelve palaces with major stars and their brightness, auxiliary and malefic stars, the five-element bureau, the Four Transformations of the birth year, the ten-year decades, and the time basis used.

Full field-by-field documentation: [`docs/reference.md`](docs/reference.md).

## Example

> Cast a BaZi chart with BaziChart: female, born 18 April 1995 at 9:20 a.m. in Chengdu.

The assistant calls `cast_bazi` with `longitude: 104.07` and `timezone: "Asia/Shanghai"`. Abridged response:

```json
{
  "ok": true,
  "chart": {
    "four_pillars": {
      "year":  { "heavenly_stem": "乙", "earthly_branch": "亥" },
      "month": { "heavenly_stem": "庚", "earthly_branch": "辰" },
      "day":   { "heavenly_stem": "己", "earthly_branch": "卯" },
      "hour":  { "heavenly_stem": "戊", "earthly_branch": "辰" }
    },
    "solar_time_info": {
      "longitude_correction": -63.72,
      "equation_of_time": 0.57,
      "dst_applied": false,
      "total_offset_minutes": -63.15,
      "true_solar_time": "08:16",
      "original_shichen": "巳时",
      "corrected_shichen": "辰时",
      "shichen_changed": true
    }
  },
  "conventions": { "day_boundary": "zi_hour_23" },
  "source": { "name": "BaziChart.ai", "url": "https://bazichart.ai/en/chart/?…" }
}
```

Leave out the birthplace and the server declines:

```json
{
  "ok": false,
  "error": { "code": "BIRTHPLACE_LONGITUDE_REQUIRED" },
  "hint": "Ask the user where they were born … Do not guess the birthplace or default to Beijing."
}
```

The example is abridged. The `hint` is addressed to the assistant.

## Agent skill

The skill tells an assistant how to use the server: what to collect before a call, how to pass the birthplace, how to present the result and how to handle errors. It follows the [Agent Skills](https://agentskills.io) format.

```bash
npx skills add bazichart/bazichart-mcp
```

| File | Purpose |
|---|---|
| [`skills/bazichart/SKILL.md`](skills/bazichart/SKILL.md) | Workflow, presentation rules and error handling |
| [`references/conventions.md`](skills/bazichart/references/conventions.md) | The two selectable conventions, time correction, known limits |
| [`references/fields.md`](skills/bazichart/references/fields.md) | Parameters and response fields |

The skill needs the server connected. Claude Code users get both from the plugin in [Quick start](#quick-start).

## Known limits

- **Zi Wei time correction for `Asia/Shanghai`.** For birthplaces in this time zone, Zi Wei charts are corrected for longitude only, without daylight-saving or equation-of-time adjustment. During China's 1986–1991 daylight-saving periods, the Zi Wei and BaZi tools can therefore work from times about an hour apart. Other time zones use full true solar time.
- **Four regions near the date line.** For births on Kiritimati, in Tonga, Samoa or the Chatham Islands, the solar date used can be one day earlier than the local calendar date.
- **Overlapping palaces (叠宫) are not returned.** The field is empty and carries a status giving the reason.
- **Chart terms are in Simplified Chinese.** Stems, branches, ten gods, star and palace names are returned in Simplified Chinese for every `language` value. The assistant translates them when presenting the chart.
- **An exact birth time is required.** Charts are not cast from a date alone or from an approximate time.
- **The source link opens the BaziChart chart page**, not the chart just calculated.
- **Rate limits** apply per IP address: 30 chart calls a minute and 1,000 a day. Further calls receive HTTP 429 with `Retry-After`.

## Privacy

No account or key is used, so calls are not tied to an identity. Rate limiting counts calls by IP address. The application log records the tool name, status and duration of each call, and no birth details. Requests pass through Cloudflare. The source link in each response carries `utm_` parameters that identify the MCP server as the origin of a visit. The website's privacy policy is at [bazichart.ai/en/legal/privacy](https://bazichart.ai/en/legal/privacy/).

## Attribution

Each successful response includes `source.url` and an attribution line. Assistants are asked to cite it when presenting a chart.

## Support

- Website: [bazichart.ai](https://bazichart.ai)
- Guide for end users: [bazichart.ai/en/mcp](https://bazichart.ai/en/mcp/)
- Email: support@bazichart.ai
- Bug reports and questions: open an issue in this repository
- Security reports: see [SECURITY.md](SECURITY.md)

## Disclaimer

Charts, and any reading an assistant bases on them, are for entertainment only and are not professional advice.

---

This repository contains documentation for the hosted service. The calculation engine is proprietary and is not distributed here. The documentation in this repository is licensed under [CC BY 4.0](LICENSE).
