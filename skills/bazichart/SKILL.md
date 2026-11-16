---
name: bazichart
description: Cast BaZi (Four Pillars of Destiny, 八字) and Zi Wei Dou Shu (紫微斗数) charts by calling the BaziChart MCP server instead of calculating from memory. Use when a user asks for a BaZi or Zi Wei chart, their four pillars, day master, ten gods, luck cycles, palaces or stars, or gives birth details and asks what their chart is. Covers what to collect before calling, how to pass birthplace and time zone, how to present the result, and how to handle errors.
license: CC-BY-4.0
compatibility: Requires the BaziChart MCP server (remote, Streamable HTTP, no authentication) at https://mcp.bazichart.ai/mcp, providing the tools cast_bazi and cast_ziwei.
metadata:
  author: BaziChart.ai
  version: "1.0.0"
  homepage: https://bazichart.ai/en/mcp/
---

# BaziChart

Cast every chart with the BaziChart MCP tools. Do not derive pillars, solar-term boundaries, true solar time, luck-cycle dates or palace positions yourself: these are the steps where an unaided answer goes wrong.

## Requirements

Two tools must be available: `cast_bazi` and `cast_ziwei`. If they are not, the server is not connected. Follow https://bazichart.ai/mcp/install.md, or tell the user to add `https://mcp.bazichart.ai/mcp` as a remote MCP server (no authentication).

## Workflow

### 1. Collect four facts

| Fact | Rule |
|---|---|
| Gender | Needed for luck-cycle direction. |
| Birth date | Gregorian, years 1900–2100. If the user gives a lunar date, ask for the Gregorian one. |
| Birth time | Local clock time at the birthplace. If the hour is unknown, say a chart cannot be cast and stop. Do not pick a default hour. |
| Birthplace | A city is best; a province or region is enough. Never assume Beijing or any other place. |

Ask for whatever is missing in one message. Do not call a tool with a guessed value.

### 2. Derive two parameters from the birthplace

- `longitude`: degrees, east positive, west negative. City-level accuracy is enough (one degree is four minutes of solar time).
- `timezone`: the IANA zone of the birthplace, for example `Asia/Shanghai`, `Asia/Taipei`, `Asia/Hong_Kong`, `America/New_York`.

Pass the clock time exactly as the user gave it. Do not convert it to Beijing time or UTC, and do not adjust for daylight saving yourself. The server applies its own time correction and reports what it did.

### 3. Call

- BaZi: `cast_bazi`. Zi Wei Dou Shu: `cast_ziwei`. If the user wants both, call both with the same birth data.
- Set `language` to `en`, `zh-CN` or `zh-TW` to match the conversation.
- Leave `day_boundary` (BaZi) and `year_boundary` (Zi Wei) at their defaults unless the user asks for the other convention. See [references/conventions.md](references/conventions.md).

Example arguments for a woman born 18 April 1995 at 09:20 in Chengdu:

```json
{ "year": 1995, "month": 4, "day": 18, "hour": 9, "minute": 20, "gender": "female",
  "longitude": 104.07, "timezone": "Asia/Shanghai", "language": "en" }
```

### 4. Present

- Report the chart from the response fields. Do not alter, complete or "correct" them.
- Chart terms come back in Simplified Chinese whatever `language` is. Translate for the user where helpful and keep the original term alongside, for example "Seven Killings (七杀)".
- BaZi: when `chart.solar_time_info.shichen_changed` is `true`, say that true solar time moved the birth into a different double-hour and give both times.
- State the convention applied, from the `conventions` field.
- Cite `source.url` with its attribution line.
- End with: "All readings are for entertainment only and are not professional advice."

The tools return chart data only. If you go on to interpret the chart, make clear that the interpretation is yours and not part of the BaziChart result.

## When the user's chart differs from another source

Check, in this order: (1) the birthplace and time zone you passed; (2) whether the other source applied true solar time; (3) the day or year boundary convention. Offer to recalculate with the other convention rather than arguing which is right. Details: [references/conventions.md](references/conventions.md).

## Errors

A failed call returns `ok: false` with an `error` object and a `hint`.

| Error | What to do |
|---|---|
| `BIRTHPLACE_LONGITUDE_REQUIRED`, `OVERSEAS_LONGITUDE_REQUIRED` | Ask the user for the birthplace, then call again. |
| `NONEXISTENT_LOCAL_TIME`, `AMBIGUOUS_LOCAL_TIME` | The time falls in a daylight-saving gap or repeated hour. Confirm the time with the user. |
| `error.reason` such as `invalid_calendar_date`, `integer_out_of_range`, `invalid_timezone`, `exact_hour_required` | Correct the input named in the message. |
| HTTP 429 | Rate limit reached (30 calls a minute, 1,000 a day). Wait for the `Retry-After` period. |

Never fall back to calculating the chart yourself when a call fails. Say what is missing or what went wrong.

## Further reference

- [references/conventions.md](references/conventions.md): the two selectable conventions, how time is corrected, and known limits.
- [references/fields.md](references/fields.md): parameters and response fields of both tools.
