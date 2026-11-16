# BaziChart MCP — Reference

Server: `https://mcp.bazichart.ai/mcp` · Server name `bazichart_chart_mcp` · Transport: Streamable HTTP, stateless · Authentication: none

## Protocol

| Request | Result |
|---|---|
| `POST /mcp` with a JSON-RPC body | `application/json` response. Supported methods: `initialize`, `tools/list`, `tools/call` |
| `GET /mcp` | `405` — the server does not open event streams |
| `GET /healthz` | `200` with `ok` and `engine_pin` (`verified` when the running engine matches its pinned identity) |

Request bodies are limited to 16 KB. Only `tools/call` counts toward the rate limit.

## Input rules common to both tools

- **Dates are Gregorian.** Years 1900–2100.
- **Times are local clock time at the birthplace**, as written on the birth record. Do not convert to Beijing time or UTC.
- **`timezone` is the birthplace's IANA zone.** Hong Kong, Macao and Taipei have their own zones.
- **`longitude` is required** — degrees, east positive, west negative, −180 to 180. A city-level value is sufficient; one degree of longitude is four minutes of solar time.
- **`precision` is fixed to `exact`.** If the birth hour is unknown, do not call the tool.
- **Chart terms are returned in Simplified Chinese** (stems, branches, ten gods, Na Yin, star and palace names, relationship names), whatever `language` is set to. `language` selects the language of `source.url` only.
- **Local times that do not exist or occur twice** (daylight-saving gaps and repeated hours) are rejected rather than resolved by assumption.

## `cast_bazi`

| Parameter | Type | Required | Notes |
|---|---|---|---|
| `year` | integer 1900–2100 | yes | |
| `month` | integer 1–12 | yes | |
| `day` | integer 1–31 | yes | |
| `hour` | integer 0–23 | yes | |
| `minute` | integer 0–59 | no | default `0` |
| `gender` | string | yes | `male`/`female`, `M`/`F`, `男`/`女`; case-insensitive |
| `longitude` | number | yes | |
| `timezone` | string | yes | IANA name |
| `day_boundary` | `zi_hour_23` \| `midnight_00` | no | default `zi_hour_23` |
| `language` | `en` \| `zh-CN` \| `zh-TW` | no | default `en`; unknown values fall back to `en` |
| `precision` | `exact` | no | Fixed value. Leave unset; any other value returns `EXACT_PRECISION_REQUIRED` |
| `shichen_index` | integer 0–11 or null | no | Leave unset |

### Response

| Field | Content |
|---|---|
| `ok`, `status` | `true` / `"ok"` on success |
| `chart.four_pillars` | `year`, `month`, `day`, `hour`, each with `heavenly_stem` and `earthly_branch` |
| `chart.ten_gods` | Ten god of each heavenly stem and of each hidden stem, relative to the day master |
| `chart.hidden_stems` | Hidden stems of the four branches |
| `chart.nayin` | Na Yin of the four pillars |
| `chart.dizhi_relations` | An object whose keys are the Chinese relationship names (`六合`, `三合`, `拱合`, `六冲`, `三刑`, `两刑`, `自刑`, `六害`, `暗合`): six combinations, three harmonies, arching combinations, six clashes, punishments, six harms, hidden combinations |
| `chart.dayun` | Ten luck cycles: stem, branch, elements, age range, exact start age, transition date-time, direction, and the solar term used as reference |
| `chart.solar_time_info` | Time correction: `beijing_time` (the local clock time as entered, for any time zone), `longitude_correction`, `equation_of_time`, `dst_applied`, `dst_offset_minutes`, `total_offset_minutes`, `true_solar_time`, `corrected_datetime`, `original_shichen`, `corrected_shichen`, `shichen_changed`, `standard_meridian`, `instant_utc` |
| `chart.effective_datetime` | The corrected date-time the pillars were derived from |
| `input` | The normalised input as understood by the server |
| `conventions.day_boundary` | The day boundary applied |
| `source` | `name`, `url`, `attribution` |

Responses may carry additional diagnostic fields (for example `chart.correction_trace`, `chart.time_basis_version`). Treat fields not listed here as informational; they can change without notice.

**Day boundary.** With `zi_hour_23`, a birth at 23:00–23:59 true solar time takes the next day's pillar. With `midnight_00` it keeps the same day's pillar. The test is made on true solar time, not clock time.

## `cast_ziwei`

| Parameter | Type | Required | Notes |
|---|---|---|---|
| `year`, `month`, `day` | integers | yes | as above |
| `hour` | integer 0–23 | yes | |
| `minute` | integer 0–59 | no | default `0` |
| `gender` | string | yes | as above |
| `longitude` | number | yes | |
| `timezone` | string | no | default `Asia/Shanghai` |
| `year_boundary` | `lunar_new_year` \| `lichun` | no | default `lunar_new_year` |
| `language` | `en` \| `zh-CN` \| `zh-TW` | no | default `en` |
| `precision` | `exact` | no | Fixed value. Leave unset; any other value returns `EXACT_PRECISION_REQUIRED` |
| `shichen_index` | integer 0–11 or null | no | Leave unset |

### Response

| Field | Content |
|---|---|
| `chart.input` | Gregorian and lunar birth data as used, including leap-month and late-zi-hour flags |
| `chart.ming_gong`, `chart.shen_gong` | Life Palace (branch, stem, palace name) and Body Palace (branch, palace name) |
| `chart.wu_xing_ju` | Five-element bureau |
| `chart.year_ganzhi` | Year stem-branch under the selected `year_boundary` |
| `chart.ming_zhu`, `chart.shen_zhu` | Life ruler and body ruler stars |
| `chart.palaces` | Twelve palaces as an object keyed `"1"`–`"12"`. Each has `index`, branch, stem, name, `major_stars`, `major_star_brightness`, `auxiliary_stars`, `malefic_stars`, `sihua`, `is_shen_gong` |
| `chart.sihua` | Four Transformations of the birth-year stem |
| `chart.dalim` | Direction, start age, and the ten-year decades with their palaces and transformations |
| `chart.diegong` | Overlapping palaces — always reported as held (`HELD_SOURCE_GAP`) |
| `chart.validation` | Internal consistency check of the chart |
| `time_basis` | `birthplace_timezone`, `local_clock_time`, `dst_applied`, `true_solar_time` (`null` for `Asia/Shanghai` birthplaces), `correction_minutes`, and the rule applied |
| `conventions.year_boundary` | The year boundary applied |
| `source` | `name`, `url`, `attribution` |

**Time basis.** For `Asia/Shanghai` birthplaces the correction is longitude only. For every other zone the chart uses true solar time, including the equation of time and local daylight-saving rules.

**Year boundary.** `lunar_new_year` changes the year stem-branch on the first day of the first lunar month. `lichun` changes it at the Start of Spring solar term. The choice affects births between the two dates.

## Errors

Tool-level errors arrive as a normal `200` response. The MCP result has `isError: true`; its payload has `ok: false`, `status: "error"`, an `error` object (`code` where one exists, `message`, and sometimes `type` and `reason`) and a `hint` addressed to the assistant. `error.message` may be in Chinese; rely on `code` and `reason`.

| `error.code` | Meaning | What the assistant should do |
|---|---|---|
| `BIRTHPLACE_LONGITUDE_REQUIRED` | No longitude was supplied | Ask the user for the birthplace. Do not guess |
| `OVERSEAS_LONGITUDE_REQUIRED` | Zi Wei outside `Asia/Shanghai` without longitude | Same |
| `EXACT_PRECISION_REQUIRED` | `precision` other than `exact` | Ask for the exact birth time |
| `DAY_BOUNDARY_INVALID`, `YEAR_BOUNDARY_INVALID` | Unknown convention value | Use a documented value |
| `ZIWEI_VALIDATION_ERROR` | Zi Wei input rejected, e.g. an unsupported field (`error.reason` gives the cause) | Correct the input |
| `NONEXISTENT_LOCAL_TIME`, `AMBIGUOUS_LOCAL_TIME` | The clock time falls in a daylight-saving gap or repeated hour | Confirm the time with the user |
| *(no code; `error.type` and `error.reason`)* | BaZi input rejected by the engine contract, e.g. an impossible date (`invalid_calendar_date`), a year outside 1900–2100 (`integer_out_of_range`), an unknown time zone (`invalid_timezone`), a missing hour (`exact_hour_required`) or an unsupported field (`unexpected_field`) | Correct the input |

Transport-level errors use HTTP status codes.

| Status | Code | Meaning |
|---|---|---|
| 400 | `INVALID_NUMBER`, `INVALID_CONTENT_LENGTH` | Malformed request |
| 403 | `FORBIDDEN_HOST_OR_ORIGIN` | Request not addressed to the public host name |
| 405 | `METHOD_NOT_ALLOWED` | Use `POST /mcp` |
| 408 | `REQUEST_READ_TIMEOUT` | Body not received in time |
| 413 | `REQUEST_TOO_LARGE` | Body over 16 KB |
| 429 | `RATE_LIMITED` | Over 30 calls a minute or 1,000 a day from one source; see `Retry-After` |
| 503 | `SERVER_BUSY`, `ENGINE_UNAVAILABLE` | Retry shortly |

## Guidance for assistants

1. Collect gender, Gregorian birth date, local birth time and birthplace before calling. If any is missing, ask.
2. Derive `longitude` and `timezone` from the birthplace the user gave. Never substitute a default city.
3. Pass `language` to match the conversation.
4. Present the chart as returned. State that true solar time was applied when `shichen_changed` is `true`.
5. Cite `source.url` with the attribution line and include: "All readings are for entertainment only and are not professional advice."
6. The tools do not interpret. Make clear which statements are chart data and which are your own reading.
