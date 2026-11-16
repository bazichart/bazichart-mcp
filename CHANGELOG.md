# Changelog

Changes to the hosted service that affect tool inputs or outputs. Dates are UTC.

Entries are dated because the service is updated in place. The version in `skills/bazichart/SKILL.md` and `.claude-plugin/` is the version of the skill and plugin package, not of the server.

## 2026-11-16

- **`longitude` is now required** for `cast_bazi` and for `cast_ziwei` on `Asia/Shanghai` birthplaces. Calls without it return `BIRTHPLACE_LONGITUDE_REQUIRED`. Previously such calls were calculated on uncorrected clock time.
- **New optional `language` parameter** (`en`, `zh-CN`, `zh-TW`) on both tools. It selects the language of `source.url`. Default `en`.

## 2026-11-15

- Public endpoint `https://mcp.bazichart.ai/mcp` available.
- Tools: `cast_bazi`, `cast_ziwei`.
- Selectable conventions: `day_boundary` (BaZi) and `year_boundary` (Zi Wei).
